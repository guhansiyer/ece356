# Functional Dependencies

Redundancy in a database leadsa to troublesome anomalies:

* **Update anomalies**: a repeated value may be changed in one place but not another.
* **Insertion anomalies**: in order to insert a value, it becomes necessary to insert some unrelated value.
* **Deletion anomalies**: deleting one type of information leads to the loss of another type of information.

We can remove redundancy by decomposing attributes and relations but we pay a price:

* Additional processing to compute joins.
* Additional space for tables and indexes.
* Must enforce referential integrity constraints.

On one hand, compute and storage (on average) becomes less expensive over time, and joins can be quite fast. On the other hand, dealing with anomalies can require human intervention, which is slow and costly.

## Repetition

> Repetition within one tuple

```sql
section(course_id, sec_id, semester, year, building, room_number)

('ECE356', '001', 'W13', 2013, 'E2', 'E2-1303') /* 13/2013, E2
```

> Repetition across tuples

```sql
inst_dept(ID, name, salary, dept_name, building, budget)

('22222', 'Einstein', '95000', 'Physics', 'Watson', 70000)
('33456', 'Gold', '87000', 'Physics', 'Watson', 70000) /* Physics, Watson, 70000
```

In these examples, there are two types of repetition:

* The value domains of some attributes are not **atomic**, one value may encode multiple pieces of information:
  * 'E2-1303' repeats information in 'E2'.
* Attribute values in different tuples are related by **functional dependencies**, one subset of attributes functionally determines the values of another subset.
  * `dept_name -> building, budget`

We can remedy this by breaking up value domains to create atomic domains, and/or by decomposing relations to avoid specific types of functional dependencies.

## Relational Decomposition

Consider the schema `inst_dept(ID, name, salary, dept_name, building, budget)`. 

How do we know that `(dept_name, building, budget)` should be a separate relation? Which attributes should remain in `inst_dept` if we construct a new relation on `(dept_name, building, budget)`?

Our goal is to produce a **lossless-join decomposition**, a decomposition of relation schema $R$ into schemas $R_1$ and $R_2$ such that, for every instance $r(R), r = r_1 \bowtie r_2$. We must beware of **lossy decomposition** where $r$ cannot always be reconstructed by joining $r_1$ and $r_2$.

We will define a theory of functional dependencies, using them to decide whether a relation schema $R$ is in "good" form. If $R$ is not in "good" form, we will carefully decompose it into a set of schemas, such that each new schema is in "good" form and the decomposition is a lossless-join decomposition.

We will study precise definitions of levels of "goodness" called **normal forms**, which can be achieved with specific decompositions.

## First Normal Form: 1NF

A value domain is **atomic** if its elements are considered to be indivisible units (e.g.: identifiers like 'ECE356', which can be broken up into parts).

A relational schema $R$ is in **first normal form** if the domains of all attributes of $R$ are atomic.

Non-atomic domains are bad because they complicate storage, encourage redundancy, and lead to information being encoded in business logic (e.g.: an application parses 'ECE356' to get department name).

**From now on, we assume all relations are in 1NF unless otherwise stated.**

## A Theory of Functional Dependencies

Functional dependencies (FDs) are constraints on the set of legal relational instances, ones that conform to some conceptual data model, which itself is guided by our informal understanding of the world.

FDs state that the value for a certain set of attributes uniquely determines the value for another set of attributes. An FD is a generalization of the notion of a **key**.

Let $R$ be a relation schema where $\alpha \subseteq R, \beta \subseteq R$.

The functional dependency $\alpha \rightarrow \beta$ holds on an instance $r(R)$ if and only if whenever two tuples $t_1$ and $t_2$ of $r$ agree on the attributes $\alpha$, they also agree on $\beta$. In other terms:

$$
(t_1(\alpha) = t_2(\alpha)) \implies (t_1(\beta) = t_2(\beta))  
$$

The functional dependency $\alpha \rightarrow \beta$ holds on $R$ if and only if for any legal relation instance $r(R)$, the functional dependency holds on $r$.

$K$ is a superkey for relation schema $R$ if and only if $K \implies R$. 

$K$ is a candidate key for $R$ if and only if $K \implies R$ and there is no $\alpha \subset K$ such that $\alpha \implies R$.

Functional dependencies allow us to express constraints that cannot be expressed using superkeys.

Consider the schema `inst_dept(_ID_, name, salary, dept_name, building, budget)`. We expect the following functional dependencies to hold:

* `dept_name -> building`
* `ID -> building`

But, we would not expect `dept_name -> salary` to hold.

## Use of Functional Dependencies

We use functional dependencies to:

* Test relation instances to see if they are legal.
  * If a relation instance $r(R)$ is legal under a set $F$ of FDs, then we say that $r$ satisfies $F$.
* Specify constraints on the set of legal relations.
  * If, for a relation schema $R$, every relation instance $r(R)$ satisfies a set $F$ of FDs, then we say that $F$ holds on $R$.

## Armstrong's Axioms

Given a set $F$ of FDs, there may be certain other FDs that are not in $F$ but are logically implied by those in $F$. For example, if $F = \{A \implies B, B \implies C\}$, we can infer that $A \implies C$. 

The set of all FDs logically implied by F is the **closure** of $F$, denoted by $F^+$. Genreally, $F^+ \supseteq F$.

We can find $F^+$ by repeatedly applying **Armstrong's Axioms**:

* If $\beta \subseteq \alpha$, $\alpha \implies \beta$ (reflexivity).
* If $\alpha \implies \beta$, $\gamma\alpha \implies \gamma\beta$ (augmentation).
* If $\alpha \implies \beta$ and $\beta \implies \gamma$, then $\alpha \implies \gamma$ (transitivity).

### Additional Inference Rules

These are inferences from Armstrong's Axioms.

* If $\alpha \implies \beta$ holds and $\alpha \implies \gamma$ holds, $\alpha \implies \beta \gamma$ (union).
* If $\alpha \implies \beta \gamma$ holds, $\alpha \implies \beta$ and $\alpha \implies \gamma$  (decomposition).
* If $\alpha \implies \beta$ and $\gamma \beta \implies \delta$, then $\alpha \gamma \implies \delta$ (pseudotransitivity).

## Additional Properties

### Extraneous Attributes

Given a set of attributes $\alpha$, the **closure** of $\alpha$ under $F$ is defined as the set of attributes functionally determined by $\alpha$ under $F$, denoted by $\alpha^+$

We can use this to:

* Test for a superkey.
  * Compute $\alpha^+$ and check if it containts all attribtues of a relation schema $R$.
* Test individual FDs.
  * Compute $\alpha^+$ and check that $\beta \supseteq \alpha^+$ to test an FD $\alpha \implies \beta$.
* Compute $F^+$ for a set of FDs.

Consider a set $F$ of functional dependencies and an FD $\alpha \implies \beta$ in $F$. 

An attribute $A$ is **extraneous** in $\alpha$ if $A \in \alpha$ and $F$ logically implies $(F - \{\alpha \implies \beta\}) \cup \{(\alpha - A) \implies \beta\}$

To test whether $A \in \alpha$ is extraneous in $\alpha$:

* Compute $(\alpha - A)^+$ with the dependencies in F.
* $A$ is extraneous in $\alpha$ if and only if $(\alpha - A)^+$  contains $\beta$.

An attribute $B$ is **extraneous** in $\beta$ if $B \in \beta$ and the set of functional dependencies $(F - \{\alpha \implies \beta\}) \cup \{\alpha \implies (\beta - B) \}$ logically implies $F$.

To test whether $B \in \beta$ is extraneous in $\beta$:

* Compute $\alpha^+$ with the dependencies in $F^{'} = (F - \{\alpha \implies \beta\}) \cup \{\alpha \implies (\beta - B) \}$
* $B$ is extraneous in $\beta$ if and only if $\alpha^+$  contains $B$.

### Canonical Cover

A canonical cover for a set $F$ of functional dependencies is a set $F_c$ of functional dependencies such that:

1. $F$ logically implies all dependencies in $F_c$,
2. $F_c$ logically implies all dependencies in F,
3. No FD in $F_c$ contains an extraneous attribute,
4. Each left side of an FD in $F_c$ is unique.
