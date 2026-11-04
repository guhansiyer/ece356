# Denormalized Schemas

## Data Fragmentation

Data fragmentation, or partitioning, is the division of a relation $r$ into fragments $r_1, r_2, ..., r_n$ that contain sufficient information to reconstruct the relation.

**Horizontal fragmentation** is where each tuple of $r$ is assigned to one or more fragments stored on different servers.

**Vertical fragmentation** is when the schema for $r$ is split into smaller schemas, where all schemas must contain a common candidate key to preserve lossless join. A special attribute (`tuple-id`) may be added to each schema to serve as a candidate key.

### Advantages of Fragmentation

Horizontal:

* allows parallel processing on relation fragments which leads to scalability.
* allows a relation to be split so that tuples are located where they are most frequently accessed.

Vertical:

* allows tuples to be split so that each part of the tuple is stored where it is most frequently accessed.
* `tuple-id` allows efficient joining of vertical fragments.
* allows parallel processing on a relation.

Vertical and horizontal fragmentation can be mixed.

## Practical Considerations

Suppose a relation has $k$ attributes and $n$ tuples. Horizontal fragmentation makes it possible to partition the table across up to $n$ fragments. Vertical fragmentation makes it possible to partition the table across up to $k$ fragments. Since $n >> k$ in practice, horizontal partitioning enables greater scalability.

The simplest strategy for horizontal fragmentation is to determine the fragment by applying a hash function to a **partitioning key**, which is usually the primary key or a prefix of the primary key.

This is called **hash partitioning**, it tends to spread the storage load uniformly across fragments or partitions, but does not always spread the computation load well because some tuples might be accessed more frequently than others.

## Mapping Fragments to Servers

A server can hold multiple fragments. A hash function maps each partitioning key to a fragment, and fragments are assigned to servers.

When a server is added, **consistent hashing** limits how much data must move. With four evenly loaded servers, adding the fourth server moves roughly one quarter of the fragments, rather than requiring all fragments to be redistributed.

## Modeling for Horizontal Fragmentation

For a distributed workload, two main goals are to:

* spread data evenly across partitions, often using hash partitioning;
* minimize the number of partitions each query must read, by choosing a key that matches common queries.

Minimizing writes and avoiding data duplication are not necessarily goals: denormalization may duplicate data to make reads more local and efficient.

### Choosing a Partitioning Key

Suppose `account` has primary key `(branch_name, account_number)`. For this query:

```sql
select account_number
from account
where branch_name = 'Hillside';
```

Using `branch_name` as the partitioning key keeps a branch's accounts together, so the query can read one partition. The same key supports grouping by branch:

```sql
select branch_name, count(*)
from account
group by branch_name;
```

If the entire primary key were used, accounts from one branch could be spread across multiple partitions, making both queries more expensive. A good partitioning key is selected for the workload; it need not be the entire primary key.

## Denormalization by Duplication

Partitioning related tables by their own keys can make a join a cross-partition operation. Consider:

```text
instructor(ID, name, dept_name, salary)
department(dept_name, building, budget)
```

To answer a lookup of an instructor's department building, the instructor may be partitioned by `ID`, while the department is partitioned by `dept_name`. The rows may be on different servers, so the join requires network communication.

### Example 1: Lookup Building by Instructor ID

One option is to duplicate `building` in `instructor`:

```text
instructor(ID, name, dept_name, building, salary)
department(dept_name, building, budget)
```

Then the lookup needs no join and can use the instructor's `ID` partition:

```sql
select building
from instructor
where ID = 123;
```

This redundancy violates 3NF and BCNF for `instructor`. If a department's building changes, the duplicated values must be updated consistently, ideally in an atomic transaction; coordinating such updates across servers can be difficult.

Alternatively, add a lookup relation:

```text
id_building(ID, building)
```

Partitioning it by `ID` makes the same query local, while the original relations remain normalized:

```sql
select building
from id_building
where ID = 123;
```

This duplicates both `ID` and `building`, and the added relation may only help this particular lookup.

## Example 2: Many-to-Many Instructor-Department Mapping

If instructors can belong to multiple departments, use:

```text
instructor(ID, name, salary)
department(dept_name, building, budget)
inst_dept(ID, dept_name)
```

Looking up buildings for an instructor requires joining `inst_dept` with `department`. To avoid that join, one design stores a list of buildings in each instructor row. This speeds up lookup by `ID`, but uses a non-atomic, multi-valued attribute, violating 1NF and making lookup by department harder.

Another option duplicates `building` in the mapping relation:

```text
inst_dept(ID, dept_name, building)
```

Now the query can read `inst_dept` directly:

```sql
select building
from inst_dept
where ID = 123;
```

The primary key is `(ID, dept_name)`. Partitioning by `ID` (a proper prefix of that key) keeps all departments for one instructor together. Partitioning by the full primary key could scatter them across partitions and would not provide the same one-partition lookup.

## Design Trade-offs

Denormalization can make frequent reads faster by reducing joins and the number of partitions accessed. In exchange, it duplicates data or uses non-atomic values, increasing update work and the risk of inconsistent copies. Choose the design based on the workload and the queries that need to be efficient.
