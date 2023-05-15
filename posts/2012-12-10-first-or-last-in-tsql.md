---
title: First or last in T-SQL
author: Jorick van der Hoeven
date: '2012-12-10'
categories: [snippet, T-SQL]
---

I've been looking for a while now to find a way to return just one row when the fields I am using split up my table return several rows per unique field combination. I found this little gem today:
<pre class="brush: sql; gutter: false; first-line: 1; highlight: [11]">Select
   t1,
   t2,
   t3
From
(
   Select
      t1,
      t2,
      t3,
      row_number() over (Partition by t1 order by t1) rowrank
   from tTable
)
Where rowrank &lt;=1</pre>
This of course does not emulate first and last exactly as it doesn't really allow you to influence the order of the returned fields in the partitioning. It is however very useful when you just want to return one value.
