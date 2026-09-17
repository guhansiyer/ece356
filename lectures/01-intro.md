# Relational Model

Relational databases "think" in terms of relations which can be used to represent things and the relationships between them (eg: students, courses, enrollments).

Conceptually, relations are tables with rows and columns. The rows may represent entities or relationships, and the columns represent attributes.

A relation is a *mathematical object* and a table is its *physical embodiment*.

## Attribute Types

The set of allowed values for each attribute is called the **domain** of the attribute. Attribute values are (usually) required to be atomic, or indivisible.

* A single attribute cannot represent a set or a list of values.
  * Ex: A database designer may insist that the first and last name be separated into distinct attributes.

The special value **null** is a member of every domain, and is used to represent missing or unknown data.

## Relation Schema and Instance

Let $A_1, A_2, ..., A_n$ denote attributes.

Let $D_1, D_2, ..., D_n$ denote domains.

$R(A_1, A_2, ..., A_n)$ denotes a relation schema over these attributes.

* Ex: instructor(ID, name, dept_name, salary).

A relation $r$ conforming to schema $R$, denoted as $r(R)$, is a subset of $D_1 \times D_2 \times ... \times D_n$.

* The table is a subset of the set of all possible values in the attribute domain(s).

Thus, a relation is a set of $n$-tuples ($a_1, a_2, ..., a_n$) where each $a_i \in D_i$.

An element $t$ or $r$ is a tuple, and corresponds to a row in a table.

A relation instance refers to the concrete values of a relation (e.g.: set of Waterloo instructors as of 10 am on September 7th, 2018).

## Databases

A database typically comprises many relations. In the design process, information about an enterprise is broken up:

* instructor
* student
* advisor

Sometimes, database designers make questionable decisions:

```
univ(instructor_ID, name, dept_name, salary, student_ID, ...)
```

* Repitition of information (ex: two students have the same instructor)
* Need for null values (ex: represent a student with no advisor)

## Keys

Let $R$ be a relation schema and let $K \subseteq R$ (K is a subset of R's attributes).

**Superkey** and **Candidate Key** are defined as follows:

* $K$ is a **superkey** of $R$ if values for $K$ are sufficient to identify a unique tuple of each possible relation instance for schema $R$.
  * Ex: {ID} and {ID, name} are superkeys of instructor.
* Superkey $K$ is a **candidate key** if $K$ is minimal.
  * Ex: {ID} is a candidate key for instructor since it can uniquely identify each instructor.
* **One** of the candidate keys is selected to be the **primary key**.

**Foreign keys** are an attribute value in one relation that must appear in another relation.

* **Referencing relation** (child table) contains a foreign key.
* **Referenced relation** (parent table) contains a referenced key
* This is always the primary key according to the textbook, but allowed by some DBMS implementations to be any candidate key.