# "Hot boiled beans and butter; walk in and find your supper!"

### I. Introduction
1. FT.CREATE:
The FT.CREATE command is used to create a new search index in Redisearch. It allows you to define the schema for the index, including the fields and their types. Here's the basic syntax of FT.CREATE:

```
FT.CREATE index_name [SCHEMA field_name field_type [field_name field_type ...]]
```

- `index_name` is the name of the index you want to create.
- `SCHEMA` specifies the schema definition for the index.
- `field_name` is the name of a field in the index.
- `field_type` is the type of the field. Redisearch supports various types such as TEXT, NUMERIC, TAG, and more.

For example, to create an index named "products" with two fields, "name" of type TEXT and "price" of type NUMERIC, you can use the following command:

```
FT.CREATE products SCHEMA name TEXT price NUMERIC
```

2. FT.SEARCH:
The FT.SEARCH command is used to search for documents in a Redisearch index. It allows you to perform full-text searches and apply various filters and sorting options. Here's the basic syntax of FT.SEARCH:

```
FT.SEARCH index_name search_query [option [value] ...]
```

- `index_name` is the name of the index to search.
- `search_query` is the query string used to match documents.
- `option` and `value` are optional arguments that can be used to modify the behavior of the search.

For example, to search for documents in the "products" index that contain the term "apple" in the "name" field, you can use the following command:

```
FT.SEARCH products "apple"
```

3. FT.AGGREGATE:
The FT.AGGREGATE command is used to perform aggregations on Redisearch indexes. It allows you to calculate various metrics and apply groupings and reductions to search results. Here's the basic syntax of FT.AGGREGATE:

```
FT.AGGREGATE index_name search_query aggregation_query
```

- `index_name` is the name of the index to aggregate.
- `search_query` is the query string used to match documents.
- `aggregation_query` defines the aggregations to be performed on the search results.

For example, to calculate the average price of products in the "products" index, you can use the following command:

```
FT.AGGREGATE products "*" APPLY AVG(@price) as avg_price
```

This query performs a search on all documents in the "products" index using the "*" wildcard. It then applies the AVG function on the "price" field and aliases the result as "avg_price".

These are the basic explanations of the FT.CREATE, FT.SEARCH, and FT.AGGREGATE commands in Redisearch. They provide powerful functionality for creating and searching full-text indexes and performing aggregations on the indexed data.


### II. [FT_CREATE](https://redis.io/commands/ft.create/)
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


### III. [FT.SEARC](https://redis.io/commands/ft.search/)
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


### IV. [FT.AGGREGATE]()
```
FT.AGGREGATE
  {index_name:string}
  {query_string:string}
  [VERBATIM]
  [LOAD {nargs:integer} {property:string} ...]
  [GROUPBY
    {nargs:integer} {property:string} ...
    REDUCE
      {FUNC:string}
      {nargs:integer} {arg:string} ...
      [AS {name:string}]
    ...
  ] ...
  [SORTBY
    {nargs:integer} {string} ...
    [MAX {num:integer}] ...
  ] ...
  [APPLY
    {EXPR:string}
    AS {name:string}
  ] ...
  [FILTER {EXPR:string}] ...
  [LIMIT {offset:integer} {num:integer} ] ...
  [PARAMS {nargs} {name} {value} ... ]
```


### EOF (2024/03/07)