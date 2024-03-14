### Redis Stack aggregation pipeline <br />─── "Don't worry about being old, worry about thinking old"

![alt Jasper's sacrifices](img/Jaspers-sacrifices.jpg)

### Prologue 
> Time was when travel-stained pilgrims rode in clattering parties through the city’s welcome shades; time is when wayfarers, leading a gipsy life between haymaking time and harvest, ... [more](https://www.gutenberg.org/cache/epub/564/pg564-images.html#chap19)


### I. Variation is invariant 
Ever since I made acquaintance with [relational databases](https://lc.fie.umich.mx/~rodrigo/BD/An%20Introduction%20to%20Database%20Systems%208e%20By%20C%20J%20Date.pdf), things were kept in a well organized hierarchy, ie. database, schema, table, record, field... line by line and step by step along the way from top to down. Tables were *normalized*, indexes were extensively built to enforce *constraints* and to further speed up file access. The power of table joining and aggregation it evinced was so unparalleled as to make all competitors tumbled down instantaneously. Various [RDBMS](https://www.oracle.com/database/what-is-a-relational-database/) artefact quickly populated and successfully dominated the computing world thenceforth. 

![alt phpMyAdmin](img/phpMyAdmin.JPG)

Three decades have passed since then... When I first scratched [MongoDB](https://www.mongodb.com/), similar hierarchy vaguely emerges from my soul, ie. database, collection, document, property... etc. Instead of grid of rigid rows, a more elastic JSON format is employed to enable property of array and embedded objects. [Aggregation](https://github.com/Albert0i/MongoDB-aggregation-pipeline/blob/main/README.md) can be devised ingeniously to pair with relational database, comprehensive and advanced are provided. 

![alt MongoDB Compass](img/MongoDB_Compass.JPG)

Of late, I come access [Redis Stack](https://redis.io/docs/about/about-stack/), which is a new species from NoSQL family, makes me petrified and almost Loss of consciousness: **Everything iss kept on top-level**. The reason is obvious, by keeping them on the top level, a constant access time through primary keys is attained. This simplicity and speed have made it a popular choice for applications that require fast and efficient data access. 

![alt RedisInsight](img/RedisInsight.JPG)

In addition, Redis provides a rich set of data structures besides *key-value pairs*, such as *lists*, *sets*, *sorted sets*, *hashes*. These data structures offer efficient and flexible storage and retrieval of data. Use cases are caching, session management, real-time analytics, and job queues. 

---

> Redis has been designed to offer an alternative for problems where relational databases (RDBMSs) are not a good fit because there is something wrong if we use an RDBMS for all kinds of work. However, in comparison to other data storage options that became popular when the NoSQL wave shook the world of databases (Memcached, the key-value data store released in 2003, or MongoDB, the document store released in 2009, and many more), Redis has its roots in computer science and makes a rich variety of data structures available. This is one of the distinguishing features of Redis and the likely reason that fostered its adoption by software engineers and developers – presenting data structures such as hashes, lists, sets, bitmaps, and so on that are familiar to software engineers so they could transfer the programming logic to data modeling without any lengthy and computationally expensive data transformation. Viewed in this light, we could say that Redis is about persisting the data structures of a programming language.

IHMO, MongoDB is rival to relational database, while Redis Stack competes for time and efficiency on it's own. 


### II.  [RedisJSON](https://github.com/RedisJSON/RedisJSON)
> RedisJSON is a Redis module that implements ECMA-404 The JSON Data Interchange Standard as a native data type. It allows storing, updating and fetching JSON values from Redis keys (documents).

#### 1. [JSON.SET](https://redis.io/commands/json.set/)
```
JSON.SET key path value [NX | XX]
```
Set the JSON value at path in key. 

```
JSON.SET inventory:12345 $ '{ 
        "name": "Noise-cancelling Bluetooth headphones",
        "category": "headphone", 
        "description": "Wireless Bluetooth headphones with noise-cancelling technology",
        "wireless": true,
        "connection": "Bluetooth",
        "price": 78.99,
        "stock": 25,
        "free-shipping": true,
        "colors": [
          "black",
          "silver",
          "pink"
        ]
    }' 
```

```
JSON.SET "bicycle:0" "." "{\"pickup_zone\": \"POLYGON((-74.0610 40.7578, -73.9510 40.7578, -73.9510 40.6678, -74.0610 40.6678, -74.0610 40.7578))\", \"store_location\": \"-74.0060,40.7128\", \"brand\": \"Velorim\", \"model\": \"Jigger\", \"price\": 270, \"description\": \"Small and powerful, the Jigger is the best ride for the smallest of tikes! This is the tiniest kids\\u2019 pedal bike on the market available without a coaster brake, the Jigger is the vehicle of choice for the rare tenacious little rider raring to go.\", \"condition\": \"new\"}"
```

#### 2. [JSON.GET](https://redis.io/commands/json.get/)
```
JSON.GET key [INDENT indent] [NEWLINE newline] [SPACE space] [path
  [path ...]]
```
Return the value at path in JSON serialized form. 

```
JSON.GET inventory:12345

JSON.GET "bicycle:0"
```


### III. [RediSearch](https://github.com/RediSearch/RediSearch)
> However, if in addition to the ability to use core data structures to store the data, we ensure that fast searches can be performed (besides primary key lookup), it is possible to think beyond the basic caching use case and start looking at Redis as a full-fledged database, capable of high-speed searches.

> Redis Stack complements Redis with the ability to create secondary indexes on Hashes or JSON documents, the two document types supported by Redis Stack. The search examples seen so far can be resolved with the indexing features.

> RediSearch is a Redis module that provides querying, secondary indexing, and full-text search for Redis. To use RediSearch, you first declare indexes on your Redis data. You can then use the RediSearch query language to query that data.

#### 1. [FT.CREATE](https://redis.io/commands/ft.create/)
```
FT.CREATE index 
  [ON HASH | JSON] 
  [PREFIX count prefix [prefix ...]] 
  [FILTER {filter}]
  [LANGUAGE default_lang] 
  [LANGUAGE_FIELD lang_attribute] 
  [SCORE default_score] 
  [SCORE_FIELD score_attribute] 
  [PAYLOAD_FIELD payload_attribute] 
  [MAXTEXTFIELDS] 
  [TEMPORARY seconds] 
  [NOOFFSETS] 
  [NOHL] 
  [NOFIELDS] 
  [NOFREQS] 
  [STOPWORDS count [stopword ...]] 
  [SKIPINITIALSCAN]
  SCHEMA field_name [AS alias] TEXT | TAG | NUMERIC | GEO | VECTOR | GEOSHAPE [ SORTABLE [UNF]] 
  [NOINDEX] [ field_name [AS alias] TEXT | TAG | NUMERIC | GEO | VECTOR | GEOSHAPE [ SORTABLE [UNF]] [NOINDEX] ...]
```
Create an index with the given specification. For usage, see [Examples](https://redis.io/commands/ft.create/#examples).

```
FT.CREATE inventory:index
  ON JSON 
  PREFIX 1 inventory:
  SCHEMA $.name AS name TEXT SORTABLE 
         $.category AS category TAG SORTABLE 
         $.description AS description TEXT
         $.wireless AS wireless TAG SORTABLE 
         $.connection AS connection TAG SORTABLE
         $.price AS price NUMERIC SORTABLE
         $.stock AS stock NUMERIC SORTABLE
         $.free-shipping AS free_shipping TAG SORTABLE
         $.colors AS colors TAG SEPARATOR "," SORTABLE
         $.regex_pat AS regex_pat TAG SORTABLE
```

```
FT.CREATE bicycle:index ON 
    JSON PREFIX 1 bicycle: SCORE 1.0 SCHEMA 
    $.pickup_zone AS pickup_zone GEOSHAPE 
    $.store_location AS store_location GEO 
    $.brand AS brand TAG 
    $.model AS model TAG 
    $.description AS description TEXT WEIGHT 1.0 
    $.price AS price NUMERIC 
    $.condition AS condition TAG SEPARATOR , 

```

#### 2. [FT.SEARCH](https://redis.io/commands/ft.search/)
```
FT.SEARCH index query 
  [NOCONTENT] 
  [VERBATIM] [NOSTOPWORDS] 
  [WITHSCORES] 
  [WITHPAYLOADS] 
  [WITHSORTKEYS] 
  [FILTER numeric_field min max [ FILTER numeric_field min max ...]] 
  [GEOFILTER geo_field lon lat radius m | km | mi | ft [ GEOFILTER geo_field lon lat radius m | km | mi | ft ...]] 
  [INKEYS count key [key ...]] [ INFIELDS count field [field ...]] 
  [RETURN count identifier [AS property] [ identifier [AS property] ...]] 
  [SUMMARIZE [ FIELDS count field [field ...]] [FRAGS num] [LEN fragsize] [SEPARATOR separator]] 
  [HIGHLIGHT [ FIELDS count field [field ...]] [ TAGS open close]] 
  [SLOP slop] 
  [TIMEOUT timeout] 
  [INORDER] 
  [LANGUAGE language] 
  [EXPANDER expander] 
  [SCORER scorer] 
  [EXPLAINSCORE] 
  [PAYLOAD payload] 
  [SORTBY sortby [ ASC | DESC] [WITHCOUNT]] 
  [LIMIT offset num] 
  [PARAMS nargs name value [ name value ...]] 
  [DIALECT dialect]
```
Search the index with a textual query, returning either documents or just ids

```
FT.INFO inventory:index
FT.SEARCH inventory:index * 
FT.SEARCH inventory:index * return 3 name category price
FT.SEARCH inventory:index '@name:(keyboard)'
FT.SEARCH inventory:index '-@name:(keyboard wireless)'
FT.SEARCH inventory:index '@category:{headphone}'
FT.SEARCH inventory:index '@description:(combo)'
FT.SEARCH inventory:index '-@wireless:{true}'
FT.SEARCH inventory:index '@connection:{usb}'
FT.SEARCH inventory:index '@price:[-inf 50]'
FT.SEARCH inventory:index '@stock:[52 75]'
FT.SEARCH inventory:index '@stock:[(52 (75]'
FT.SEARCH inventory:index '@free_shipping:{true}'
FT.SEARCH inventory:index '@colors:{pink|silver}'

FT.SEARCH inventory:index "wire"
FT.SEARCH inventory:index "(keyboard) (wireless)"
FT.SEARCH inventory:index "(keyboard)|(wireless)"
FT.SEARCH inventory:index "@name:(keyboard) @price:[0 30]"
FT.SEARCH inventory:index  "@name:(keyboard) @price:[0 30]" return 3 _key name price
```


### IV. [Aggregation](https://redis.io/docs/interact/search-and-query/advanced-concepts/aggregations/)

![alt aggregation pipeline](img/Pipeline%20-%20DO%20NOT%20CHANGE.png)

[Here](https://redis.io/docs/interact/search-and-query/query/) is a short SQL comparison using the [bicycle dataset](https://redis.io/docs/interact/search-and-query/query/data/bicycles.txt):

| Type | SQL | Redis Stack |
| ----------- | ----------- | --- |
| Selection | SELECT * FROM bicycles WHERE price >= 1000 | FT.SEARCH idx:bicycle "@price:[1000 +inf]"  | 
| Simple projection | SELECT id, price FROM bicycles | FT.SEARCH idx:bicycle "*" RETURN 2 __key, price |
| Calculated projection | SELECT id, price-price*0.1 AS discounted FROM bicycles | FT.AGGREGATE idx:bicycle "*" LOAD 2 __key price APPLY "@price-@price*0.1" AS discounted |
| Aggregation | SELECT condition, AVG(price) AS avg_price FROM bicycles GROUP BY condition | FT.AGGREGATE idx:bicycle "*" GROUPBY 1 @condition REDUCE AVG 1 @price AS avg_price |

```
SELECT condition, count(*) AS num, sum(price) AS money
FROM bicycles 
GROUP BY condition
ORDER BY 2
OFFSET 0 LIMIT 10 
```

```
FT.AGGREGATE bicycle:index * 
    LOAD 3 __key brand model     
    APPLY 'format("%s-%s", @brand, @model)' AS name     
    GROUPBY 1 @condition     
    REDUCE TOLIST 1 @__key AS keys 
    REDUCE TOLIST 1 @name AS names 
    REDUCE COUNT 0 AS num 
    REDUCE SUM 1 @price AS money
    SORTBY 2 @num DESC 
    LIMIT 0 10 
```

### V. Summary 
In summary, the MongoDB aggregation pipeline is a comprehensive and feature-rich framework for versatile data transformations and aggregations across collections, while Redis Stack's `FT.AGGREGATE` focuses on basic aggregations within the context of full-text search, leveraging Redisearch's capabilities. The choice between the two depends on the specific requirements of your application and whether full-text search is a primary consideration.

How does MongoDB aggregation pipeline differs from those in Redis Stack FT.AGGREGATE
(Generated by ChatGPT) 


### VI. Reference
1. [Redis Stack Tutorial](https://youtu.be/McPR39mkp7w)
2. [Redis Stack for Application Modernization](https://www.amazon.com/Redis-Stack-Application-Modernization-applications/dp/1837638187)
3. [Caching Slow Database Queries with Node.js and Redis](https://youtu.be/krSgKN-5DHs?list=PL83Wfqi-zYZHtpd4Glbj-NBIz7RB0Jw5u)
4. [Querying, Indexing, and Full-text Search in Redis](https://youtu.be/infTV4ifNZY?list=PL83Wfqi-zYZHtpd4Glbj-NBIz7RB0Jw5u)
5. [The Mystery of Edwin Drood](https://www.gutenberg.org/cache/epub/564/pg564-images.html)


### Epilogue
> Abstract thought has formed the basis of innovation and creativity throughout human existence, and can therefore be seen as one of humanity’s most characteristic defining features. [more](https://bc805.medium.com/abstract-thought-how-is-it-significant-and-how-does-it-define-the-basis-for-modern-humanity-a98a5b92fb9f)


### EOF (2024/03/15)