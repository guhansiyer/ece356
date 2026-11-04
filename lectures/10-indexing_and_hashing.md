# Indexing and Hashing

Queries often select a subset of records in a table:

* **Point Query**: find records with a specified attribute value.
* **Range Query**: find records with attribtue values in a given range.

In most cases the query only returns a small subset of the records in a table, we would like to locate these records efficiently.

Simple file structures have fundamental weaknesses:

* **Heap file**: insertion is fast but lookup requires a scan.
* **Sequential file**: supports binary search (log time) but only on one search key, all other queries require a scan, and the use of overflow blocks degrades performance.

When a file is stored on a disk, even "efficient" binary search can be slow because each record fetched may require a random I/O.

An **index** is a data structure that speeds up lookup of records by specific search keys.

Indicies boost performance in two ways:

1. They organize data in ways that benefit specific types of queries.
2. They are usually smaller than the data files they refer to.

## Designing the Physical Schema

Suppose that the logical schema is decided. First, we choose a structure for each relation (sequential, heap, index-organized).

Then, we add indices *judiciously*:

* Identify *costly* queries that would benefit from the index.
  * Queries with `where` clauses.
  * Queries with `joins`.
* Choose index type: usually $B^+$-tree or hash.

## Basic Concepts

> Search key: an attribute or set of attributes used to look up records.

> Index: a collection of records (index entries) of the form `search-key | pointer`

There are two kinds of indicies:

1. **Ordered indices**: search keys are stored in sorted order.
2. **Hash indicies**: search keys are near uniformly distributed across buckets with a hash function.

### Ordered Indicies

* **Primary Index**: the index whose search key coincides with the sequential order of the file that stores the table.
* **Secondary Index**: an index whose search key specifices an order different from the sequential order of the file that stores the table
* **Index-sequential File**: ordered sequential file and a primary index.

### Dense Index Files

A **dense index** is one with an index record for every value of the search key.

* Ex: index on *ID* attribute of `instructor` relation.

### Sparse Index Files

A **sparse index** contains records for only some values of the search key. This is possible only when the file is ordered by the same search key.

To locate a record with search key value $K$:

* find the record with the largest search key value $< K$.
* scan the file forward starting at the record to which the index record points, stop when search key value $> K$.

Compared to dense indices, sparse indicies use less space and incur less maintenance overhead for insertions/deletions. Additionally, they are generally slower for locating individual records.

### Primary vs. Secondary Indices

Indicies enable faster search. Updating indices incurs overhead; when a table is modified every index must be updated. 

A sequential scan using primary index is efficient as long as the file has few overflow blocks, but using a secondary index is expensive because each record access may fetch a new block from the disk, where each fetch requires 5-10 ms if it leads to random I/O.

## The Need for Advanced Data Structures

Sequential files can be used to store both indicies and data. They enable efficient search, but suffer from the degradation problem: searching becomes slower as file grows due to overflow blocks. Therefore, the file must be periodically reorganized.

The solution is to use advanced data structures to organize the index file and/or data file:

* $B^+$-tree: good all arround, supports efficient range queries.
* Hashes: good for point queries only.

## $B^+$-trees

$B^+$-tree indices are an alternative to index-sequential files, which suffer from performance degradation over time:

* automatically reorganizes itself with small, local changes, in the face of insertions/deletions.
* reorganization of entire structure at once is not required to maintain performance.

Minor disadvantages include computation and space overheads, which are usually outweighed by their big advantages.

A $B^+$-tree satisfies many of the useful properties of B-trees:

* all paths from root to leaf have equal length.
* large branching factor, hence shallow height.
* each node that is not a root is at least "half full".

A $B^+$-tree differs as follows:

* internal nodes only store **separator keys** and pointers to other nodes.
* all the data valeus are stored in leaf nodes, which are chained together with **sibling pointers**.
* the data values are pointers to records in a file, which can be represented as record numbers or as values of the primary key.

## Indexing Strings with $B^+$-trees

Keys are often variable length strings. It is possible to use a variable branching factor, and the decision to split ndoes is determined by space utilization, not the number of keys.

**Prefix compression** is a technique that reduces space overhead. Keys at internal nodes can be prefixes of full keys if they retain enough characters to distinguish entries in the sub-trees separated by the key value.

## Index-Organized Tables

In an index-organized tale, the index stores the rows of the table rather than pointers to the rows.

The advantage is that we avoid having to traverse a pointer from the index to the file when performing a row lookup by primary key.

The disadvantage is that a full traversal of the index structure is required to scan the entire table, which may incur more random I/Os than scanning a sequential file.

## Multiple-Key Accesses

Multiple indices might be useful for certain types of queries:

```sql
select ID 
from instructor 
where dept_name = "Finance" and salary = 80000
```

Strategies for processing the query using indices on single attributes:

1. Index on `dept_name` to find instructors with department name `Finance`, test `salary = 80000`.
2. Index on `salary` to find instructors with a salary of $80000, test `dept_name = "Finance"`.
3. Use `dept_name` to find pointers to all records pertaining to the `Finance` department, similarly index on `salary`. Take the intersection of all pointers obtained.

**Composite search keys** are search keys with more than one attribute. They are compared using **lexicographic ordering**, where $(a_1,a_2) < (b_1,b_2)$ if $a_1 < b_1$ or $a_1 = b_1$ and $a_2 < b_2$.

Going back to the previous example, if we have an index on the composite search key `(dept_name, salary)`, the index can efficiently handly the query with the where clause as is. It can also efficiently handle `where dept_name = "Finance" and salary < 80000`, but it cannot efficiently handle `where dept_name < "Finance" and salary = 80000`.
