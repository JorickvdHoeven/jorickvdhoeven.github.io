---
title: Pivoting tables when you don't know the columns in T-SQL
author: Jorick van der Hoeven
date: '2014-06-11'
categories: [snippet, T-SQL]
---

Here's the situation, you've got a column in a table which you want to pivot on but you don't have any control over the distinct number of values that can be in that column. It could be 3 or it could be 20. However, you still need to be able to pivot the whole table to display the information to the user. How do you do this?

The answer here lies in the way you can use string variables as code in dynamic SQL, and there are two parts of the pivot code which have become variables in the situation outlined above:
```sql
SELECT
    UnPivotedColumn1,
    --Variable Block #1
    PivotVal1,
    ...
    PivotValX
    --End: VB #1
FROM (
    SELECT UnPivotedColumn1
                ,PivotSource --Contains values PivotVal1 - X
                ,ColToAggregate
    FROM TableYouWantToPivot
) tbl
PIVOT
(
SUM(ColToAggregate)

FOR PivotSource
IN (
   --Variable Block #2
   [PivotVal1], ... , [PivotValX]
   --End: VB #2
   )

) pvt

```

Therefore, to create this pivot when we have a variable number of values in our PivotSource column we need to populate two strings with the distinct values of the PivotSource Column -- Variable Block #1 and Variable Block #2.

To do this we will use the method I explained in an earlier <a title="How to include several values in one field in SQL" href="http://jorick.vanderhoeven.ch/2012/10/how-to-include-several-values-in-one-field-in-sql/">post</a>. For Variable Block #1 the code looks like:

``` sql
DECLARE @VB1 VARCHAR(MAX);

SET @VB1 = STUFF(
          SELECT ', ' + PivotSource
          FROM TableYouWantToPivot
          FOR XML PATH(''),
          1,2,'')

</pre>
The stuff command here removes the comma at the beginning so that we have a comma-separated list of values and no trailing comma.

The code for Variable block #2 is similar but formatted slightly differently.
<pre lang="sql">DECLARE @VB1 VARCHAR(MAX);

SET @VB1 = STUFF(
          SELECT ', [' + PivotSource + ']'
          FROM TableYouWantToPivot
          FOR XML PATH(''),
          1,2,'')

```

Finally we need to construct our Dynamic SQL statement that we need to execute:

``` sql
DECLARE @SQLPivot VARCHAR(MAX)

SET @SQLPivot = '
  SELECT
      UnPivotedColumn1,
      ' + @VB1 + '
  FROM (
      SELECT UnPivotedColumn1
                  ,PivotSource --Contains values PivotVal1 - X
                  ,ColToAggregate
      FROM TableYouWantToPivot
  ) tbl
  PIVOT
  (
  SUM(ColToAggregate)

  FOR PivotSource
  IN (
     ' + @VB2 + '
     )

) pvt '

EXEC @SQLPivot

```
And there you have it, a Pivot statement that can pivot a table if you don't know how many variables will be in the column you want to pivot on.

-J