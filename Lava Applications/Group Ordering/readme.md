# Group Ordering

In Rock, the Group model has an [Order] property, but unless groups are a part of a Check-In Configuration, there is no UI to adjust their order.

Most of 928Central's groups are for storing RSVPs and attendance at the monthly events, so the groups are primarily month names. When these are alphabetized it looks messy.

This file documents a custom page with a Lava Application Content block, using a single Lava Application endpoint, that allows us to re-order child groups of a selected parent.

![A custom RockRMS page interface, with a Group Tree View list on the left with the Monthly Events > 2026 group selected. The right side has a panel with title "Child Groups", showing a table listing one child group per row, all named with a month and year. On the right side of the table, each row has an "Order" column with text inputs, and the order for the groups are set to ascending integers from 0 to 9. "March 2026" is highlighted indicating it's being edited with the number 2, and a green "Saved" label is shown to the left of the order input indicating that the value has been updated.](GroupOrder-Screenshot.png "Completed Group Ordering page")

## 1. Lava Application: /group-order
- **Endpoint:** Order Set (/set)
  - **Description:** This endpoint sets the [Order] property of a given GroupId.
  - **HTTP Method:** Put
  - **Enabled Lava Commands:** Rock Entity Modify
  - **Code Template:**

```liquid
{% assign fieldParts = QueryString.field | Split:'-' %}
{% assign fieldLength = fieldParts | Size %}

{% if fieldParts[0] == 'group' and fieldLength == 2 %}
    //- unfortunate that to>from JSON is needed. But I can't seem to get the property from Form otherwise unless I use dot notation which requires some ugly `raw` tags.
    {% assign newValue = Form | ToJSON | FromJSON | Property:QueryString.field | AsInteger %}
    
    {% if newValue >= 0 %}
    
        {% modifygroup id:'{{ fieldParts[1] }}' %}
            [[ property name:'Order' ]]{{ newValue }}[[ endproperty ]]
        {% endmodifygroup %}
        
        {% if ModifyResult.Success == true %}
            <span class="label label-success auto-dismiss">Saved</span>
        {% else %}
            <div class="alert alert-warning">
                <strong>{{ ModifyResult.ErrorMessage }}</strong><br>
                Validation Messages<br>
                <ul>
                {% for message in ModifyResult.ValidationErrors %}
                    <li>{{ message.ErrorMessage }}</li>
                {% endfor %}
                </ul>
            </div> 
        {% endif %}
        
    {% else %}
        <span class="alert alert-warning">Requires a number greater than or equal to 0</span>
    {% endif %}
    
{% endif %}
```

## 2. Page: Group Ordering
- **Page Route:** /admin/settings/group-ordering
- **Icon CSS Class:** ti ti-stack-front
- **Parent Page:** Admin Tools > Administration > Settings > General
- **Layout:** Left Sidebar

### Sidebar Zone:
- **Block:** Group Tree View
  - **Group Types Include:** Monthly Events

### Main Zone:
- **Block:** Lava Application Content
  - **Lava Template:**
```liquid
{% assign childGroups = 'Global' | PageParameter:'GroupId' | GroupById | Property:'Groups' | OrderBy:'Order, Name' %}
{% assign childGroupCount = childGroups | Size %}

{[ panel title:'Child Groups' type:'block' ]}
    {% if childGroupCount > 0 %}
        <div class="grid grid-panel">
            <table class="table table-grid table-striped table-hover">
                <thead>
                    <tr><th>Group</th><th style="width: 5rem;">Order</th></tr>
                </thead>
                <tbody>
                    {% for g in childGroups %}
                        <tr>
                            <td>{{ g.Name }}<span id="result-{{ g.Id }}" class="pull-right mr-3"></span></td>
                            <td class="pt-3 pb-0">
                                {[ textbox
                                    name:'group-{{ g.Id }}'
                                    type:'number'
                                    showlabel:'false'
                                    value:'{{ g.Order }}'
                                    width:'xs'
                                    additionalattributes:'hx-put="^/group-order/set?field=group-{{ g.Id }}" hx-target="#result-{{ g.Id }}"'
                                ]}
                            </td>
                        </tr>
                    {% endfor %}
                </tbody>
            </table>
        </div>
    {% else %}
        No child groups found. Please select a parent group on the left to re-order its child groups.
    {% endif %}
{[ endpanel ]}

{% stylesheet %}
    .auto-dismiss {
      animation: fadeAndRemove 0.5s ease 1.5s forwards;
    }
    
    @keyframes fadeAndRemove {
      to {
        opacity: 0;
        visibility: hidden;
      }
    }
{% endstylesheet %}
```