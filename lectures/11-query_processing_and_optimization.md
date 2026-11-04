# Query Processing and Optimization

One of the fundamental components of a DBMS is the **query processor** which includes a **query evaluation engine**.

The processor translates every SQL query submitted by an application to a concrete **evaluation plan**. The evaluation engine then executes that plan by interacting with the **storage manager** layer.

The storage manager is responsible for fetching rows from tables, inserting and updating rows, enforcing integrity constraints, performing concurrency control and managing transactions.

There are three basic steps in query processing:

1. Parsing and translation
2. Optimization
3. Evaluation

## Optimization and Evaluation

One relational algebra expression may have many equivalent expression:

$$
(\text{instructor} \bowtie \text{teaches}) \bowtie \text{course}
\\ \text{is equivalent to }
\\ \text{instructor} \bowtie (\text{teaches} \bowtie \text{course})
$$

Furthermore, each operation can be evaluated using several different algorithms. As a result, there are many ways to evaluate a given relational algebra expression.

An evaluation plan is an annotated expression specifying the detailed evaluation strategy for a given query.

Optimizing a query entails reasoning about:

* equivalent relational algebra expressions for the query.
* possible evaluation plans for each candidate RA expression.
* cost of each candidate evaluation plan.

Assuming that we can settle on a meaningful definition of "cost", optimizing a query entails finding the equivalent RA expression and evaluation plan that minimizes cost.

In practice, cost is estimated using statistical information recorded in the system catalogue, such as:

* the number of tuples in a table or the number of entries in an index.
* the size of each tuple or index entry, height of a $B^+$-tree index.
* cardinality of an attribute.
* selectivity of an attribute.

## Engineer's Dilemma

On one hand, we know that adding indexes can speed up some queries drastically. 

On the other, we know that indexing slows down insertions/updates, repeatedly looking up an index during a join isn't always faster than scanning the inner relation, the DBMS may create the most important indexes automatically, and it is difficult to outsmart a good query optimizer.

Therefore, designing a good physical schema may require the following:

* With a basic understanding of query evaluations, making an educated guess as to what indexes might benefit from the most important queries.
* Understanding how an index is used by the query optimizer by inspecting the evaluation plan for a given query.
* Determining the impact of adding an index by measuring performance differences empirically.

We will focus on the first two points.

## Making the Educated Guess

```sql
select ID 
from instructor 
where dept_name = "Finance" and salary < 80000
```

An index on either `dept_name` or `salary` could be useful. Separate indexes on `dept_name` and `salary` could be better. A single index on `(dept_name, salary)` could be even better.

```sql
select *
from instructor natural join department
```

Depending on the join order, an index on either the primary key `department.dept_name` or the foreign key `instructor.dept_name` could be useful.

```sql
select dept_name, budget
from department
order by budget
```

An index on `budget` could be used to find all the departments and their budgets without additional sorting.

```sql
select dept_name, avg(salary)
from instructor
group by dept_name
```

An index on `dept_name` would make it possible to efficiently identify the groups (departments) and compute the members of the group (instructors in a given department).

## Index Extensions

**Index extensions** are when the DBMS automatically appends the primary key to each secondary index entry.

They have important advantages, in that they make it possible to physically relocate table rows without having to update all secondary indexes, and they enable efficient evaluation of queries that refer to both the primary key and non-key attributes.

They also have some disadvantages in that retrieving a department record given the building requires an additional primary index lookup, the secondary index becomes larger and less likely to remain buffered in main memory.

## Covering Indexes

A **covered query** is a query that can be evaluated only with indexes and without accessing the tables. A **covering index** is an index that is used to evaluate the covered query.

```sql
select ID 
from instructor 
where dept_name = "Finance" and salary < 80000
```

Suppose that a secondary $B^+$-tree index is defined on `(dept_name, salary)`. Due to extension, the entries of the index are triples of the form `(dept_name, salary, ID)`. Such an index is a covering index for the query.

## Index Size

The smaller the index, the more likely it is to be buffered in main memory and the lower the cost of accessing it.

Several techniques can be used to reduce index size:

1. **Shorten the primary key**.
2. **Index only a prefix of a column**.
