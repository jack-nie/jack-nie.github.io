---
layout: post
title:  "postgresql's common table expressions"
date:   "2016-11-14"
keywords: ["postgresql"]
description: "postgresql"
category: "postgresql"
tags: ["postgresql"]
---
{% include JB/setup %}

WITH语句指定临时命名的结果集，这些结果集称为公用表表达式 (CTE)。 该表达式源自简单查询，并且在单条 SELECT、INSERT、UPDATE 或 DELETE 语句的执行范围内定义。WITH语句通常被用来作为一次查询的临时表而存在，通过使用WITH关键字能够创建辅助的查询语句，对于复杂的查询语句，能够极大的简化逻辑，使得SQL语句更加便于阅读。

### CTE's在查询语句中总是被物化的

CTE只要被引用一次就会被物化，以后再引用的话就不会重复进行额外的计算。如果一个CTE节点没有被任何地方引用,那么他将不会执行,下面的语句将会正常的输出而不会报错:

    ```
    WITH not_executed AS (SELECT 1/0),
        executed AS (SELECT 1)
    SELECT * FROM executed;

    ```

但是这条查询会给出一个`ERROR: division by zero`的错误。

    ```
    WITH a AS (SELECT 1/0),
    b AS (SELECT * FROM a),
    executed AS (SELECT 1)
    SELECT * FROM executed;

    ```


目前看来，使用了`order by`的CTE总是输出排序后的记录,但是不能完全依赖这种特性，可能再未来的某个时候，这项特性会被改掉。而且，如果在外部查询也使用了`order by`，那么postgresql可能不能正确的排序。

    ```
    WITH a AS (SELECT x, y FROM big_table ORDER BY x)
    SELECT *
    FROM a ;
    ```

### CTE没有自动的谓词下推(prediction pushdown)

优化关系SQL查询的一项基本技术是，将外层查询块的`WHERE`子句中的谓词移入所包含的较低层查询块（例如视图），从而能够提早进行数据过滤以及有可能更好地利用索引。只要新的查询和旧的查询在逻辑上是等价的，那么最终会返回所有相关的结果，相比较旧的查询，优化后的查询更快。但是在postgresql中，并没有采用这种优化方法，也没有区分只读还是可写的CTE，它再指定执行计划时采取了一种比较保守的做法。由于采用了相对保守的方式，优化器不会将`WHERE`语句提升到CTE子句中，即使这种做法看起来是安全的。

我们可以等待postgresql团队优化CTE，目前我们能够做的只能改变我们进行sql查询的方式。

#### 1. 在查询语句中，硬编码谓词下推后得到的sql

原始的查询语句：

    ```
    CREATE OR REPLACE FUNCTION suggest_movies(IN query text, IN result_limit integer DEFAULT 5)
      RETURNS TABLE(movie_id integer, title text) AS
    $BODY$
    WITH suggestions AS (

      SELECT
        actors.name AS entity_term,
        movies.movie_id AS suggestion_id,
        movies.title AS suggestion_title,
        1 AS rank
      FROM actors
      INNER JOIN movies_actors ON (actors.actor_id = movies_actors.actor_id)
      INNER JOIN movies ON (movies.movie_id = movies_actors.movie_id)

      UNION ALL

      SELECT
        searches.title AS entity_term,
        suggestions.movie_id AS suggestion_id,
        suggestions.title AS suggestion_title,
        RANK() OVER (PARTITION BY searches.movie_id ORDER BY cube_distance(searches.genre, suggestions.genre)) AS rank
      FROM movies AS searches
      INNER JOIN movies AS suggestions ON
        (searches.movie_id <> suggestions.movie_id) AND
        (cube_enlarge(searches.genre, 2, 18) @> suggestions.genre)
    )
    SELECT suggestion_id, suggestion_title
    FROM suggestions
    WHERE entity_term = query
    ORDER BY rank, suggestion_id
    LIMIT result_limit;
    $BODY$
    LANGUAGE sql;
    ```


将查询过滤的条件移到CTE语句内部：

    ```
    CREATE OR REPLACE FUNCTION suggest_movies(IN query text, IN result_limit integer DEFAULT 5)
      RETURNS TABLE(movie_id integer, title text) AS
    $BODY$
    WITH suggestions AS (

      SELECT
        actors.name AS entity_term,
        movies.movie_id AS suggestion_id,
        movies.title AS suggestion_title,
        1 AS rank
      FROM actors
      INNER JOIN movies_actors ON (actors.actor_id = movies_actors.actor_id)
      INNER JOIN movies ON (movies.movie_id = movies_actors.movie_id)
      WHERE actors.name = query

      UNION ALL

      SELECT
        searches.title AS entity_term,
        suggestions.movie_id AS suggestion_id,
        suggestions.title AS suggestion_title,
        RANK() OVER (PARTITION BY searches.movie_id ORDER BY cube_distance(searches.genre, suggestions.genre)) AS rank
      FROM movies AS searches
      INNER JOIN movies AS suggestions ON
        (searches.movie_id <> suggestions.movie_id) AND
        (cube_enlarge(searches.genre, 2, 18) @> suggestions.genre)
      WHERE searches.title = query
    )
    SELECT suggestion_id, suggestion_title
    FROM suggestions
    ORDER BY rank, suggestion_id
    LIMIT result_limit;
    $BODY$
    LANGUAGE sql;
    ```

第一个查询语句会通过两次查询将数据库中相关的表的数据全部查询出来然后组合成一张表，然后再进行过滤。可以看到通过这种方式，数据库返回了很多不必要的数据，更糟糕的是组合的新表由于无法利用索引导致查询的效率更低下。优化后的语句由于将过滤的条件移动到了CTE中，所以会提早的进行过滤处理，从而提高查询的效率。

#### 2. 使用子查询

postgresql能够很好的针对子查询语句进行谓词下推的优化，通过这一特性，我们能够编写出效率一样而且更加的易于维护的查询语句。
首先准备需要的表结构和数据：

    ```
    CREATE TABLE a (c INT);

    CREATE TABLE b (c INT);

    CREATE INDEX a_c ON a(c);

    CREATE INDEX b_c ON b(c);

    INSERT INTO a SELECT 1 FROM generate_series(1, 1000000);

    INSERT INTO b SELECT 2 FROM a;

    INSERT INTO a SELECT 3;
    ```

使用CTE进行查询：

    ```
    EXPLAIN ANALYZE
    WITH cte AS (
      SELECT c FROM a
      UNION ALL
      SELECT c FROM b
    )
    SELECT c FROM cte WHERE c = 3;
    ```

查询计划如下：

    ```
      QUERY PLAN
    ---------------------------------------------------------------------------------------------------------------------------
     CTE Scan on cte (cost=28850.00..73850.00 rows=10000 width=4) (actual time=607.968..1138.062 rows=1 loops=1)
      Filter: (c = 3)
      Rows Removed by Filter: 2000000
      CTE cte
      -> Append (cost=0.00..28850.00 rows=2000000 width=4) (actual time=0.013..579.749 rows=2000001 loops=1)
      -> Seq Scan on a (cost=0.00..14425.00 rows=1000000 width=4) (actual time=0.012..142.085 rows=1000001 loops=1)
      -> Seq Scan on b (cost=0.00..14425.00 rows=1000000 width=4) (actual time=0.007..122.516 rows=1000000 loops=1)
     Planning time: 0.284 ms
     Execution time: 1144.234 ms
    (9 rows)
    ```

使用子查询：

    ```
    EXPLAIN ANALYZE
    SELECT c
    FROM (
      SELECT c FROM a
      UNION ALL
      SELECT c FROM b
    ) AS subquery
    WHERE c = 3;
    ```

查询计划如下：

    ```
      QUERY PLAN
    ------------------------------------------------------------------------------------------------------------------
     Append (cost=0.42..8.88 rows=2 width=4) (actual time=0.045..0.055 rows=1 loops=1)
      -> Index Only Scan using a_c on a (cost=0.42..4.44 rows=1 width=4) (actual time=0.044..0.045 rows=1 loops=1)
      Index Cond: (c = 3)
      Heap Fetches: 1
      -> Index Only Scan using b_c on b (cost=0.42..4.44 rows=1 width=4) (actual time=0.008..0.008 rows=0 loops=1)
      Index Cond: (c = 3)
      Heap Fetches: 0
     Planning time: 0.163 ms
     Execution time: 0.097 ms
    (9 rows)

    ```

###  参考文献

- [WITH common_table_expression](https://msdn.microsoft.com/zh-cn/library/ms175972.aspx)
- [谓词下推](http://doudouclever.blog.163.com/blog/static/175112310201261355984/)
- [PostgreSQL’s CTEs are optimisation fences](http://blog.2ndquadrant.com/postgresql-ctes-are-optimization-fences/)
- [Why are the plans different if the queries are logically alike?](http://dba.stackexchange.com/questions/54388/why-are-the-plans-different-if-the-queries-are-logically-alike)
