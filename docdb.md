### Thinking in Redis<br />───The way to document database

「No kidding! There is no such thing as [SQL injection](https://www.w3schools.com/sql/sql_injection.asp) whatsoever if you do not use SQL at all...」


### Prologue 
Rectangle is defined by four sides and four right angles, which makes it easy to understand and intuitive to use. By combining multiple rectagles vertically and/or horizontally, this effectively turns into a *table*, each small rectangle becomes a *cell* of the whole. Rectangle is a rigid and restricted structure in nature because of it's height and width, operations are either first by horizontal *row* or by vertical *column* and then down into individual cell. While rectangle, per se, is a geometric concept, table is a structure, a *matrix* in mathematics. Other structures such as *List*, *Set* and *Tree* bears their own elastic topology, operations are much different... 


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

*Constant access time* refers to the property or characteristic of a data structure or algorithm where the time taken to access or retrieve an element or perform an operation remains constant, regardless of the size or scale of the data, ie. time to retrieve one record within 10 or 10,0000 records is the same. 


### II. More involved example
Subject: Local government is planning to provide lodgings for low-input families. Based on a pre-defined formula, rental fee is calculated according to economic situation as well as physical condition of individuals. A periodical random inspection is performed to verify and re-calculation is done whenever necessary. 
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

