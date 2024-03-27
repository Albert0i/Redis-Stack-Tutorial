### Thinking in Redis<br />───The way to document database

「No kidding! There is no such thing as [SQL injection](https://www.w3schools.com/sql/sql_injection.asp) whatsoever if you do not use SQL at all...」


### Prologue 
Rectangle is defined by four sides and four right angles, which makes it easy to understand and intuitive to use. By combining multiple rectagles vertically and/or horizontally, this effectively turns into a *table*, each small rectangle becomes a *cell* of the whole. Rectangle is a rigid and restricted structure in nature because of it's height and width, operations are either first by horizontal *row* or by vertical *column* and then down into individual cell. While rectangle, per se, is a geometric concept, table is a structure, a *matrix* in mathematics. Other structures such as *List*, *Set* and *Tree* bears their own elastic topology, operations are of much difference... 

Regular shape data structure are easier to handle because they are easier to understand. Irregular shape data structure ... 

### I. A piece of cake 
[Database normalization](https://en.wikipedia.org/wiki/Database_normalization) is so common a practice to be carried out in table design of day-to-day life. Setting up a trivial code table is a drudgery, ie: 

| parcod | pardes |
| ------ | ------ |
| 11 | S.ANTONIO |
| 12 | S.L ZARO |
| 13 | S.LOURENCO |
| 14 | SE |
| 15 | FATIMA |
| 21 | TAIPA |
| 22 | COLOANE |

```
CREATE TABLE tbparcod
   (
    parcod CHAR(2) NOT NULL, 
    pardes CHAR(12), 

    CONSTRAINT tbparcod_pk PRIMARY KEY (parcod)
   );

INSERT INTO tbparcod(parcod, pardes) values('11', 'S.ANTONIO');
INSERT INTO tbparcod(parcod, pardes) values('12', 'S.L ZARO');
INSERT INTO tbparcod(parcod, pardes) values('13', 'S.LOURENCO');
INSERT INTO tbparcod(parcod, pardes) values('14', 'SE');
INSERT INTO tbparcod(parcod, pardes) values('15', 'FATIMA');
INSERT INTO tbparcod(parcod, pardes) values('21', 'TAIPA');
INSERT INTO tbparcod(parcod, pardes) values('22', 'COLOANE');
```

To retrive the whole table and individual record: 
```
SELECT * FROM tbparcod;

SELECT pardes FROM tbparcod WHERE parcod='15';
```

Using [hash](https://redis.io/commands/?group=hash) to store flat file is straightforward: 
```
HSET tbparcod 11 "S.ANTONIO" 12 "S.L ZARO" 13 "S.LOURENCO" 14 "SE" 15 "FATIMA" 21 "TAIPA" 22 "COLOANE"
```

To retrive the whole table and individual record is a breeze:
```
HGETALL tbparcod

HGET tbparcod 15
```

The **Time complexity** of [HGETALL](https://redis.io/commands/hgetall/) is O(N) where N is the size of the hash; [HGET](https://redis.io/commands/hget/) is O(1). It doesn't mean a lot when table is small in size, but if your table grows up to 10,000 records, the time involved does matter. 

> In traditional SQL databases, you have two types of general queries: queries scanning an entire table (or multiple tables) or queries using an index. Table scans are O(N) while fully indexed queries are O(lg N).

The term "lg" represents the logarithm function with a base of 2. It is often called the *binary logarithm*. We can rewrite it as:

O(log2 10,000) = log(10,000) / log(2) ≈ 13.29

So to speak, when n = 10,000, the time complexity O(lg n) is approximately 13.29, which means that the algorithm or operation requires up to 14 iterations or comparisons before the search is completed. That also means the index [B+ tree](https://en.wikipedia.org/wiki/B%2B_tree) is 14 levels deep, besides occupying disk space, *re-balancing* has to be performed whenever necessary. 

*Constant access time* refers to the property or characteristic of a data structure or algorithm where the time taken to access or retrieve an element or perform an operation remains constant, regardless of the size or scale of the data, ie. time to retrieve one record within 10 or 10,000 records is the same. 


### II. More involved example
Subject: Local government is planning to provide lodgings for low-input families. Based on a pre-defined formula, rental fee is calculated according to economic situation as well as physical condition of individuals. A periodical random inspection is performed to verify and re-calculation is done whenever necessary. One sample record is as follow: 
```
   {
      "rentNo": "240031",
      "location": {
         "building": "AS-03",
         "flat": "15F"
      },
      "rent": 35.5,
      "startDate": 19990601,
      "endDate": 20240531,
      "status": "active",
      "famsiz": 3, 
      "remark": "Father recovered from the car accident. Mother continues working the night shift at the supermarket. Son enjoys playing football and is doing well in school.",
      "members": [
         {"memseq": 1, "name": "Jose Madox", "post": "father", "gender": "male", "birthday": 19620320},
         {"memseq": 2, "name": "Maria Sousa", "post": "mother", "gender": "female", "birthday": 19680703},
         {"memseq": 3, "name": "Paulo Madox", "post": "son", "gender": "male", "birthday": 19911017}
      ],
      "updateBy": "BOB",
      "updateAt": 20220820
   }
```

A combination of [RedisJSON](https://github.com/RedisJSON/RedisJSON) and [RediSearch](https://github.com/RediSearch/RediSearch) is required to store, retrieve and aggregate JSON documents. 


#### 1. Indexing documents
```
FT.CREATE rentals:index ON 
    JSON PREFIX 1 rentals: SCHEMA 
    $.rentNo AS rentNo TAG SORTABLE
    $.location.building AS building TAG SORTABLE  
    $.location.flat AS flat TAG SORTABLE  
    $.rent AS rent NUMERIC SORTABLE
    $.startDate AS startDate NUMERIC SORTABLE
    $.endDate AS endDate NUMERIC SORTABLE
    $.status AS status TAG SORTABLE  
    $.famsiz AS famsiz NUMERIC SORTABLE  
    $.remark AS remark TEXT WEIGHT 3.0 SORTABLE
    $.members[*].memseq as memseq NUMERIC SORTABLE
    $.members[*].name as name TEXT WEIGHT 3.0 SORTABLE
    $.members[*].post as post TAG SORTABLE  
    $.members[*].gender as gender TAG SORTABLE 
    $.members[*].birthday as birthday NUMERIC SORTABLE
    $.updateBy AS updateBy TAG SORTABLE
    $.updateAt AS updateAt NUMERIC SORTABLE
```

#### 2. Search 
To list all families (not recommended):
```
    FT.search rentals:index * 
```

To search for all family keys (not recommended): 
```
    keys rentals:* 
```    

To search for first 100 family keys:
```
    scan 0 match rentals:* count 100
```    

To search for family 240005:
```
    FT.search rentals:index @rentNo:{240005}
```

To get indivisual family:
```
    JSON.GET rentals:240005:001
```

To make a full text search for "walking cane":
```
    ft.search rentals:index "walking cane"
```

To make a full text search on remark field for "walking cane":
```
    ft.search rentals:index '@remark:("walking cane")'
```

To search for families with rent 50 to 100 (inclusive):
```
    ft.search rentals:index '@rent:[50 100]'
```

To search for families with rent 50 to 100 (not inclusive):
```
    ft.search rentals:index '@rent:[(50 (100]'
```

To search for all active families:    
```
    ft.search rentals:index @status:{active}
```

To search for families which lived in building AS-01 to AS-04:
```
    ft.search rentals:index @building:{as\-01|as\-02|as\-03|as\-04}
```

To search for families which lived in in flat "5A":
```
    ft.search rentals:index @flat:{5a}
```

To make a full text search for "william":
```
    ft.search rentals:index william
```

To make a full text search on name field for "william":
```
    ft.search rentals:index @name:(william)
```

To search for female and birthed in 1974:
```
    ft.search rentals:index '@birthday:[19740101 19741231] @gender:{female}'
```

To search for families updated by operator Alice: 
```
    ft.search rentals:index @updateBy:{alice}
```


#### 3.  [Aggregate](https://redis.io/docs/interact/search-and-query/advanced-concepts/aggregations/)
Commonly used GROUPBY reducers: 
| Redis Reducer       | Description                                            |
|---------------------|--------------------------------------------------------|
| COUNT               | Counts the number of items in each group.               |
| SUM                 | Calculates the sum of values in each group.             |
| MIN                 | Finds the minimum value in each group.                   |
| MAX                 | Finds the maximum value in each group.                   |
| AVG                 | Calculates the average value in each group.              |
| FIRST_VALUE         | Returns the first value in each group.                   |
| LAST_VALUE          | Returns the last value in each group.                    |
| CONCAT              | Concatenates values in each group as a string.           |
| COLLECT             | Collects values in each group as a list.                 |
| DISTINCT_COUNT      | Counts the distinct values in each group.                |
| STDDEV              | Calculates the standard deviation of values in each group. |
| VARIANCE            | Calculates the variance of values in each group.         |
| APPROXIMATE_UNIQUE  | Estimates the number of unique values in each group.     |

Commonly used APPLY expressions: 
| Redis Apply Expression | Description                                            |
|------------------------|--------------------------------------------------------|
| STRLEN                 | Returns the length of a string expression.              |
| SUBSTR                 | Extracts a substring from a string expression.          |
| CONCAT                 | Concatenates multiple string expressions.              |
| TOUPPER                | Converts a string expression to uppercase.             |
| TOLOWER                | Converts a string expression to lowercase.             |
| TRIM                   | Removes leading and trailing spaces from a string expression. |
| REPLACE                | Replaces occurrences of a substring in a string expression. |
| STRREV                 | Reverses the characters in a string expression.         |
| STRALPHA               | Checks if a string expression contains only alphabetic characters. |
| STRNUMERIC             | Checks if a string expression contains only numeric characters. |
| STRALNUM               | Checks if a string expression contains only alphanumeric characters. |
| STRCMP                 | Compares two string expressions.                       |
| STRCASECMP             | Performs a case-insensitive comparison of two string expressions. |
| STRINDEX               | Returns the index of the first occurrence of a substring in a string expression. |
| STRRINDEX              | Returns the index of the last occurrence of a substring in a string expression. |
| STRCOUNT               | Counts the occurrences of a substring in a string expression. |
| STREMPTY               | Checks if a string expression is empty.                 |
| STRTRIM                | Trims specified characters from the beginning and end of a string expression. |
| STRSPLIT               | Splits a string expression into an array of substrings based on a delimiter. |
| STRJOIN                | Joins an array of strings into a single string using a delimiter. |

To sum up all active families: 
```
    ft.aggregate rentals:index @status:{active} 
        GROUPBY 0
        REDUCE COUNT 0 AS total 
```

To group all active families by building: 
```
    ft.aggregate rentals:index @status:{active} 
        GROUPBY 1 @building
        REDUCE COUNT 0 AS num
        SORTBY 1 @building
```

To group all families by operators: 
```
    ft.aggregate rentals:index * 
        GROUPBY 1 @updateBy
        REDUCE COUNT 0 AS num
        SORTBY 1 @updateBy
```

To group all families by name: 
```
    ft.aggregate rentals:index * 
        groupby 1 @name 
        reduce count 0 as num
        sortby 2 @num desc 
```

To find out how many young people are there: 
```
    ft.aggregate rentals:index @post:{son}} 
        groupby 0 
        reduce count 0 as num

    ft.aggregate rentals:index @post:{daughter}} 
        groupby 0 
        reduce count 0 as num
```

To count by family size: 
```
    ft.aggregate rentals:index * 
        groupby 1 @famsiz 
        reduce count 0 as num 
        sortby 1 @famsiz 
```


### III. Summary 


### IV. Reference
1. [Redis as a document database quick start guide](https://redis.io/docs/get-started/document-database/)

2. [Redis Stack for Application Modernization](https://www.amazon.com/Redis-Stack-Application-Modernization-applications/dp/1837638187)

3. [Redis Command | Search and Query](https://redis.io/commands/?group=search)

4. [Thinking In Redis (part one)](https://matt.sh/thinking-in-redis-part-one)

5. [The Mystery of Edwin Drood](https://www.gutenberg.org/cache/epub/564/pg564-images.html)

### Epilogue

### EOF (2024/03/27)

ephemeral
dementia
retrofit

Using either NUMBER or DECIMAL, either CHAR or VARCHAR, is a matter of storage issue. Their access method, in and out, remains the same. 
Instead of datatype, we consider data structure here, each data structure has pros and cons suits for different scenario, access methods greatly differ.  

> In traditional SQL databases, you have two types of general queries: queries scanning an entire table (or multiple tables) or queries using an index. Table scans are O(N) while fully indexed queries are O(log N).

> In SQL systems, you have complex query planners to convert your SQL into internal database retrieval, table scanning, index lookup, and result returning commands.



"Constant access time" refers to the property or characteristic of a data structure or algorithm where the time taken to access or retrieve an element or perform an operation remains constant, regardless of the size or scale of the data. It implies that the data structure or algorithm provides efficient and consistent performance for accessing elements, regardless of the data's complexity.

This property is highly desirable in many applications and scenarios, as it ensures predictable and efficient performance, especially when dealing with large volumes of data. Data structures like arrays and hash tables often provide constant access time for specific operations, such as retrieving an element by index or accessing a value by key.

