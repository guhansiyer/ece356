# Introduction to SQL

## Multiset Relations

SQL borrows concepts from both relational algebra and relational calculus.

Because removing duplicates is expensive, in practice, relational databases allow duplicate tuples in relations, including in query results.

In other words, relations are treated as **multisets**.

## Basic Query Structure

A simple SQL query has the form: 

```sql
select A_1, A_2, ..., A_n
from r_1, r_2, ..., r_n
where P
```

* $A_i$ represents an attribute.
* $r_i$ represents a relation.
* $P$ represents a predicate over the relations.

## The Select Clause

The `select` clause lists the requested attributes in the result of a query. This corresponds to projection in relational algebra.

In general, relation names are case-sensitive but attribute names are not.

The keyword `distinct` eliminates duplicates. The keyword `all` keeps duplicates (this is default in MySQL).

An asterisk in the select clause denotes all attributes: `select * from instructor`.

The select clause can contain arithmetic expressions involving the operators `+,-.*,/`, and operating on constants or attributes of tuples.

## The Where Clause

The `where` clause specifices conditions that the result must satisfy. This corresponds to selection in relational algebra.

Comparisons can be combined using the logical connectives **and, or, not**. They can also be applied to the results of arithmetic expressions.

## The From Clause

The `from` clause lists the relations involved in the query. This corresponds to the cartesian product in relational algebra.

