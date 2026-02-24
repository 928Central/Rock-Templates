## Repository Intent
The goal of this Rock Templates repo is for the 928 Central admins to store versionable copies of scripts that we develop for our Rock RMS environment.

Filenames should end with a logical extension; `.lava`, `.sql`, or `.html` for example, and organized into logical directories so we can find and maintain them easily.

In all cases, it's best if you use something resembling the title block below in the styling guide, where possible, using that language's comment features.

For SQL Scripts that change or remove database values, especially for deletion, you should prefix the name of the file with INSERT, UPDATE, or DELETE to serve as a warning, as well as adding 'caution tape' by commenting out the relevant DELETE line(s), for example.

## SQL Styling Guide
```
-- =====================================================================================================
-- Author:      Michael Garrison
-- Create Date: 2/24/2026
-- Description: Displays page load times for a specified page. Also displays who loaded the page if 
--              known.
--              Note: Time is in seconds.
--
-- Change History:
--    3/22/2020 Ted Decker: Updated SQL to allow passing in Page Id vs having to know ComponentId.
-- =====================================================================================================

DECLARE @PageId int = 1247      -- Id of the page you want timings from. 
DECLARE @MaxRows int = 1000     -- The maximum number of rows to return.
DECLARE @DaysBack int = 1       -- The number of days back to look.

-- ----------------------------------------------------------------------------------------------------

SET @DaysBack = @DaysBack * -1
DECLARE @StartDate datetime = (SELECT DATEADD (day , @DaysBack , GETDATE() ) )

-- ----------------------------------------------------------------------------------------------------

SELECT
    TOP (@MaxRows) 
    [InteractionTimeToServe]
    , [InteractionDateTime] 
    , ISNULL( p.[NickName], '') AS [First Name]
    , ISNULL( p.[LastName], '') AS [Last Name]
FROM
    [Interaction] i
    INNER JOIN [InteractionComponent] ic ON ic.[Id] = i.[InteractionComponentId]
    LEFT OUTER JOIN [PersonAlias] pa ON pa.[Id] = i.[PersonAliasId]
    LEFT OUTER JOIN [Person] p ON p.[Id] = pa.[PersonId]
WHERE
    ic.[EntityId] = @PageId 
    AND [InteractionTimeToServe] IS NOT NULL
    AND [InteractionDateTime] > @StartDate
ORDER BY [InteractionTimeToServe] DESC 
```

## Styling Elements
1. Be sure to add the intro comment section
    1. Add a block Change History section for the next person
    2. If the template is going to be used on a given page, include the Rock page in the comments, e.g. -- https://rock.rocksolidchurchdemo.com/page/123
2. Declare variables for the person be able to adjust the query logic. The goal is that the individual should not need to tweak the rest of the template. They should just adjust the variables. Think of these as Pinocchio's strings (before becoming a boy)
3. The next section of variables is to adjust the variables to filter values. You may or may not need these. For example it's easier for the individual to say 100 days back then to have to calculate the date 100 days ago.
4. SQL should match the syntax of the [Rock Developer Codex](https://community.rockrms.com/developer/sql-style-guide).
5. Lava should match the [Lava Style Guide](https://community.rockrms.com/lava/style)

## Tips and Tricks

### Finding Scripts
To find a script you're interested in either select the directory that best matches what you're looking for, or use Github's search feature by typing `t` and a keyword.

### Finding Inserted Data
When inserting or updating data, it's not a bad idea to include a foreign key relevant to the query, so that added records can be rapidly found.

---
_(This readme is based on a similar [readme by Spark Development Network](https://github.com/SparkDevNetwork/Rock-SQL-Library/blob/master/README.md) - thank you for all of your work Spark!)_
