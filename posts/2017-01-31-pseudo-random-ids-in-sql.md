---
title: Creating anonymized pseudo-random IDs in T-SQL
author: Jorick van der Hoeven
date: '2017-01-31'
categories: [snippet, T-SQL]
---

## Creating anonymized pseudo-random IDs

I sometimes need to convert a series of unique sensitive IDs into a list of unique IDs which I can use to share with people who aren't allowed to know the IDs in the first list. The very easiest was to do that in SQL is to either use the HASHBYTES function in T-SQL and provide it with the SHA512 hashing algorithm or to use the t-SQL newID function to create new GUIDs. The difficulty with this is that it creates a very difficult to read ID so if I started with relatively simple to remember 5 character IDs I end up with a super user unfriendly 40 character hexadecimal number.

So how could one scramble a friendly&nbsp;6 character ID into an anonymous friendly 7 character ID? The easiest way to do this is to use some basic number theory and combine that with SQL's NEWID() function.

- We start off by getting a unique list of all the ID numbers we want to anonymize and scramble the order by using NEWID()
``` sql
Select ID,
       RandomRow = ROW_NUMBER() OVER (ORDER BY NEWID())
INTO RandOrder
FROM ListOfIDs
```

- We then Take that randomly ordered list and perform some math on the order using two carefully selected primes, one is a small prime the number of digits that we want the result to be in (A), and the other is a very large prime (B). I've chosen only a middlingly large prime here to make reading easier.
A = 7957597
B = 362736035870515331128527330659
We multiply these in the following way to create a unique list of numbers. NB, prime A must be larger than the total number of IDs you want to create or else this doesn't work.
``` sql
DECLARE @A INT = 7957597
DECLARE @B INT = 362736035870515331128527330659
Select ID,
       Number = (RandomRow * @B) % @A
INTO RandomNums
FROM RandOrder
```

- If we want to be fancy we can convert the integer into hex and add a prefix to denote that this is an anonymous ID. NB the size of your VARBINARY depends on the size you want your strings to be.
``` sql
Select ID,
       AnonymousID = 'RANDID' +
          RIGHT(
           master.dbo.fn_varbintohexstr(CONVERT(VARBINARY(8), Number))
          ,7)
FROM RandomNums
```