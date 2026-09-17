
# Formal Relational Query Languages: Relational Algebra

## Relational Algebra

Relational algebra is a procedural language, as opposed to declarative languages.

There are six basic operators:

* Select: $\sigma$
* Project: $\prod$
* Union: $\cup$
* Set difference: $-$
* Cartesian Product: $\times$
* Rename: $\rho$

These operators take one or two relations as inputs and output a single relation.

## Select Operation

> Syntax: $\sigma_{p}(r)$

$p$ is called the **selection predicate**.

A more formal definition is $\sigma_{p}(r) = \{t | t \in r \land p(t)\}$, where $p$ is a formula in propositional logic consisting of **terms** connected by $\land, \lor, \lnot$.

Each **term** is one of `<attribute> op <attribute> OR <constant>`, where `op` is one of $=, \neq, >, \geq, <, \leq$

An example with the select operation:

* Relation $r$:
  
| $A$ | $B$ | $C$ | $D$ |
|---|---|---|---|
| $\alpha$ | $\alpha$ | 1 | 7 |
| $\alpha$ | $\beta$ | 5 | 7 |
| $\beta$ | $\beta$ | 12 | 3 |
| $\beta$ | $\beta$ | 23 | 10 |

* $\sigma_{(A=B) \land (D > 5)}(r)$:

| $A$ | $B$ | $C$ | $D$ |
|---|---|---|---|
| $\alpha$ | $\alpha$ | 1 | 7 |
| $\beta$ | $\beta$ | 23 | 10 |

## Project Operation

> Syntax: $\prod_{A_1, A_2, ...,A_k}(r)$, where $A_1,A_2$ are attribute names and $r$ is a relation name.

The result is defined as the relation of $k$ obtained by erasing the columns that are not listed.

Since relations are sets, duplicate rows are "removed" from the result.

An example with the project operation:

* Relations $r$:
  
| $A$ | $B$ | $C$ |
|---|---|---|
| $\alpha$ | 10 | 1 |
| $\alpha$ | 20 | 1 |
| $\beta$ | 30 | 1 |
| $\beta$ | 40 | 2 |

* $\prod_{A,C}(r)$:

| $A$ | $C$ |
|---|---|
| $\alpha$ | 1 |
| $\alpha$ | 1 |
| $\beta$ | 1 |
| $\beta$ | 2 |

is equal to

| $A$ | $C$ |
|---|---|
| $\alpha$ | 1 |
| $\beta$ | 1 |
| $\beta$ | 2 |

## Union Operation

> Syntax: $r \cup s$

A more formal definition is $r \cup s = \{t | t \in r \lor t \in s\}$.

For $r \cup s$ to be valid:

1. $r, s$ must have the same $n$-arity (number of attributes).
2. The attribute domains must be compatible
   1. Example: The 2nd column of $r$ deals with the same type of values as the 2nd column of $s$.

An example with the union operation:

* Relation $r$:
  
| $A$ | $B$ |
|---|---|
| $\alpha$ | 1 |
| $\alpha$ | 2 |
| $\beta$ | 1 |

* Relation $s$:

| $A$ | $B$ |
|---|---|
| $\alpha$ | 2 |
| $\beta$ | 3 |

* $r \cup s$:

| $A$ | $B$ |
|---|---|
| $\alpha$ | 1 |
| $\alpha$ | 2 |
| $\beta$ | 1 |
| $\beta$ | 3 |

## Set Difference Operation

> Syntax: $r - s$

A more formal definition is $r - s = \{t | t \in r \land t \in s\}$.

For $r \cup s$ to be valid:

Set differences must be taken between compatible relations, just like with unions.

An example with the set difference operation:

* Relation $r$:
  
| $A$ | $B$ |
|---|---|
| $\alpha$ | 1 |
| $\alpha$ | 2 |
| $\beta$ | 1 |

* Relation $s$:

| $A$ | $B$ |
|---|---|
| $\alpha$ | 2 |
| $\beta$ | 3 |

* $r - s$:

| $A$ | $B$ |
|---|---|
| $\alpha$ | 1 |
| $\beta$ | 1 |

## Cartesian Product Operation

> Syntax: $r \times s$

A more formal definition is $r \times s = \{t q| t \in r \land q \in s\}$, where $tq$ represents a tuple obtained by concatenating $t$ and $q$.

This definition assumes that attribtues of $r(R)$ and $s(S)$ are disjoint, or that $R \cap S = \emptyset$. If they are not disjoint, then renaming must be used

An example with the cartesian operation:

* Relation $r$:

| $A$ | $B$ |
|---|---|
| $\alpha$ | 1 |
| $\beta$ | 2 |

$ Relation $s$:

| $C$ | $D$ | $E$ |
|---|---|---|
| $\alpha$ | 10 | a |
| $\beta$ | 10 | a |
| $\beta$ | 20 | b |
| $\gamma$ | 10 | b |

* $r \times s$:

| $A$ | $B$ | $C$ | $D$ | $E$ |
|---|---|---|---|---|
| $\alpha$ | 1 | $\alpha$ | 10 | a |
| $\alpha$ | 1 | $\beta$ | 10 | a |
| $\alpha$ | 1 | $\beta$ | 20 | b |
| $\alpha$ | 1 | $\gamma$ | 10 | b |
| $\beta$ | 2 | $\alpha$ | 10 | a |
| $\beta$ | 2 | $\beta$ | 10 | a |
| $\beta$ | 2 | $\beta$ | 20 | b |
| $\beta$ | 2 | $\gamma$ | 10 | b |

## Rename Operation

> Syntax: $\rho_{X}(E)$

The rename operation allows us to name, and thus refer to, the results of relational-algebra expressions. It also allows us to refer to a relation by more than one name.

If a relation algebra expression $E$ has arity n, then $\rho_{x(A_1,A_2,...,A_n)}(E)$ returns the result of expression $E$ under the name $X$ and with the attributes renamed to $A_1,A_2,...,A_n$.

## Composition of Operations

We can build expressions with multiple operations.

> Example: $\sigma_{A=C}(r \times s)$

* $r \times s$

| $A$ | $B$ | $C$ | $D$ | $E$ |
|---|---|---|---|---|
| $\alpha$ | 1 | $\alpha$ | 10 | a |
| $\alpha$ | 1 | $\beta$ | 10 | a |
| $\alpha$ | 1 | $\beta$ | 20 | b |
| $\alpha$ | 1 | $\gamma$ | 10 | b |
| $\beta$ | 2 | $\alpha$ | 10 | a |
| $\beta$ | 2 | $\beta$ | 10 | a |
| $\beta$ | 2 | $\beta$ | 20 | b |
| $\beta$ | 2 | $\gamma$ | 10 | b |

* $\sigma_{A=C}(r \times s)$

| $A$ | $B$ | $C$ | $D$ | $E$ |
|---|---|---|---|---|
| $\alpha$ | 1 | $\alpha$ | 10 | a |
| $\beta$ | 2 | $\beta$ | 10 | a |
| $\beta$ | 2 | $\beta$ | 20 | b |

## Formal Definition of Relational Algebra

A *basic expression* in the relational algebra consists of either a relation in the database (e.g.: instructor) or a constant relation (e.g.: {(1, Einstein), (2, Crick)}).

A general *relational algebra expression* is either a basic expression or an expression constructed recursively using one of the following rules, where $E_1$ and $E_2$ denote existing relational algebra expressions:

* $E_1 \cup E_2$
* $E_1 - E_2$
* $E_1 \times E_2$
* $\sigma_{P}(E_1)$, where $P$ is a predicate on attributes in $E_1$
* $\prod{p}(E_1)$, where $p$ is a predicate on attributes in $E_1$
* $\rho_{x(A_1, A_2,..., A_n)}(E_1)$, where $x(A_1, A_2,..., A_n)$ isthe new name for $E_1$ and its attributes.