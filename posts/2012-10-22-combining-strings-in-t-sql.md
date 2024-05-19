---
title: Combining strings in T-SQL
author: Jorick van der Hoeven
date: '2012-10-22'
categories: [snippet, T-SQL]
---

I found out the other day how to solve a problem that had been causing me issues when presenting data. I often got requests to resent data in such a way that we display information from a table in a grouped form while still allowing someone to consult all of the values relating to this grouping.

The solution isn't very machine friendly and is end-user oriented. The code can be seen below:
<pre lang="tsql" line="1">SELECT
    ta.a1,
    ta.a2,
    ta.a3,
    ( SELECT
          tta.a4,
      FROM TABLE_A tta
      WHERE
          ta.a1=tta.a1
          and ta.a2=tta.a2
          and ta.a3=tta.a3
      ORDER BY tta.a4
      FOR XML PATH('')
    ) AS a4plus
FROM TABLE_A ta</pre>