---
title: Distinct values in all columns in T-SQL
author: Jorick van der Hoeven
date: '2017-01-31'
categories: [snippet, T-SQL]
---

# A nifty little tool to find all the distinct values in all columns

A few months ago I became quite frustrated because I had a huge backlog of work to do performing the Quality Control of the analyses of my team members. A large portion of performing this type of Quality Control was getting a sense of the data ingested by the analyses and determining if there were any data-specific elements of the code which were not appropriately commented or if there were strange data points in the output data. To do this I needed to quickly get a sense for the possible values in an unfamiliar table, doing the same thing over and over again gets tedious, so I wrote this little script to allow me to quickly get all the distinct values in a column for every column in a table. I've provided the code to do this below with explanatory comments.

``` sql
-----------------------------------------
-- DISTINCT COLUMN QUERY
-----------------------------------------
DECLARE --Variable to hold our table specification
        @tblSpec  NVARCHAR(MAX),
        --Variable listing all unwanted columns in the table
        @unwantedColumns NVARCHAR(MAX)


SET @tblSpec = ''

--##List any columns you don't want the distinct values from here
SET @unwantedColumns = ''

--Check if the temp table exists already, if so, boot it
IF OBJECT_ID('tempdb..#unwantedcols') IS NOT NULL
  DROP TABLE #unwantedcols

--Create a table with all the unwanted columns to ignore
CREATE TABLE #unwantedCols
(colname nvarchar(max))

--This is a table valued function which splits a string,
--there are plenty of examples online for how to write one of these
INSERT INTO #unwantedCols(colname)
SELECT splitdata from fnSplitString(@unwantedColumns,'.')

--Now is where the magic starts, we start to build our query.
--What we are building here are a series of temporary tables
--which split out each of the columns and counts the number
--distinct elements in that column using the row_number()
--function
SELECT @tblSpec = @tblSpec+' '+
'SELECT [' + COLUMN_NAME +']
      , [ID] = ROW_NUMBER() OVER ( ORDER BY ['+ COLUMN_NAME+'])
INTO [#'+COLUMN_NAME+']
FROM [' +TABLE_NAME+']
GROUP BY ['+COLUMN_NAME+']'
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME=@tablename
and COLUMN_NAME not in (SELECT colname from #unwantedCols)

--Next we create a backbone for the distinct column table
--which is a sequential list of numbers as long as the
--number of rows in the table.
SET @tblSpec += '
SELECT
id = ROW_NUMBER() OVER (order by newID())
INTO #seed
from ['+ @tablename+'] '

--We then build a massive select statement which lists
--all of the columns and left joins each of the
--temporary tables created above.

--To start we build the first part of the select with
--our seed backbone column the list of numbers as
--long as the number of rows in the table we are analysing.
SELECT top 1
@tblSpec = @tblSpec+' '+
'SELECT #seed.id '
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME=@tablename
and COLUMN_NAME not in (SELECT colname from #unwantedCols)
order by ORDINAL_POSITION asc

--We then add each of the column names we want to
--analyse to our select statement
SELECT
@tblSpec = @tblSpec+' '+
',[#' +COLUMN_NAME+'].['+COLUMN_NAME+'] '
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME=@tablename
and COLUMN_NAME not in (SELECT colname from #unwantedCols)
order by ORDINAL_POSITION asc

--We start on the FROM section of the select statement
--with our seed/backbone temp table
SET @tblSpec += ' FROM #seed '

--We then left join all of our distinct column tables
--to the seed/backbone
SELECT
@tblSpec = @tblSpec+' '+
' LEFT JOIN [#' +COLUMN_NAME+'] ON [#'+COLUMN_NAME+'].id=#seed.id '
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME=@tablename
and COLUMN_NAME not in (SELECT colname from #unwantedCols)
order by ORDINAL_POSITION asc

--Finally we make our massive select statement ordered
--by the seed/backbone to make sure that all of our
--distinct values are near the top
SET @tblSpec += ' ORDER by 1'

-- And finally, we execute the entire creation,
--temp tables, seed table, and massive select statement.
EXEC (@tblSpec)
```