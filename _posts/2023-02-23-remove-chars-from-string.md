---
title: "Snippet: Remove non-printing or other unwanted characters from a string"
categories:
  - Snippet
tags:
  - nonprinting
  - string
  - character
  - GLOB
  - Utilities.GenerateSequence
  - tally
  - CROSS JOIN
  - GROUP_CONCAT
  - SUBSTRING
---
This snippet will take a string with a bunch of goofy characters in it and replace them with only a-z A-Z or 0-9

What's happening in this code is:
1. We define a string and get the length of the string
1. We generate a sequence of numbers from 1 to the length of that string.
1. We cross-join that sequence/tally table to the table with the string in it
1. This will start at position 1 and extract that character and the `GLOB` in the `WHERE` clause will compare it to a-z, A-Z, and 0-9 and only allow characters through that match letters or numbers. Each character of the string will be in a new row
1. The `GROUP_CONCAT` will then put all of the characters back into a single string without those goofy characters.

## CODE
```sql
/* a string with goofy characters, and its length */
@string = SELECT "String|With;Charac:ters" AS string, LENGTH("String|With;Charac:ters") AS length;

/* generate a sequence of numbers as long as that string */
@sqn = Utilities.GenerateSequence(Limit:@string.length);

/* filter out any non-alphanumeric characters */
SELECT
    str.string AS OriginalString,
    GROUP_CONCAT(SUBSTRING(str.string,sqn.Sequence,1),"") AS FilteredString
FROM
    @sqn sqn
    CROSS JOIN @string str
WHERE
    SUBSTRING(str.string,sqn.Sequence,1) GLOB "[a-zA-Z0-9]";
```

## OUTPUT

|OriginalString           |FilteredString      |
|-------------------------|--------------------|
|String\|With;Charac:ters |StringWithCharacters|

