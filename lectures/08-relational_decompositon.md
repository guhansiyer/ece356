# Relational Decomposition

Normalization removes redundancy by splitting a relation schema into smaller schemas. We want the resulting schemas to satisfy a normal form while preserving the original information and, where possible, the functional dependencies.

## Lossless-Join Decomposition

A decomposition of $R$ into $R_1$ and $R_2$ is **lossless-join** if, for every legal instance $r(R)$:

$$
r = \Pi_{R_1}(r) \bowtie \Pi_{R_2}(r)
$$

Joining the projections reconstructs exactly the original relation. A lossy decomposition fails this condition for some legal instance, potentially introducing spurious tuples.

## Dependency Preservation

Functional dependencies constrain legal data, but checking them can be expensive. Ideally, all constraints can be enforced by checking one table at a time, without computing joins.

For a decomposition into $R_1, \ldots, R_n$, let $F_i$ contain all dependencies in $F^+$ involving only attributes of $R_i$. The decomposition is **dependency preserving** if:

$$
(F_1 \cup F_2 \cup \cdots \cup F_n)^+ = F^+
$$

The local dependencies together must imply every original dependency. An original FD need not appear verbatim in one table if it follows from the local dependencies.

Losslessness and dependency preservation are separate properties: a lossless decomposition may still require joins to check some dependencies.

## BCNF Decomposition

Suppose a non-trivial dependency $\alpha \rightarrow \beta$ violates BCNF on $R$: $\alpha$ is not a superkey. Replace $R$ with:

* $R_1 = \alpha \cup \beta$;
* $R_2 = R - (\beta - \alpha)$.

The shared attributes $\alpha$ determine $R_1$, making this split lossless. Repeat on any resulting schema that still violates BCNF.

```text
result := {R}
while some Ri in result violates BCNF:
    choose a non-trivial FD alpha -> beta on Ri
        where alpha is not a superkey
    replace Ri with alpha union beta and Ri - (beta - alpha)
```

**BCNF decomposition always produces a lossless-join decomposition into BCNF schemas, but does not always preserve dependencies.** Different choices of violating dependencies can produce different results.

### Simple Example

```text
R(A, B, C)
F = { A -> B, B -> C }
```

`A` is the candidate key. Since `B -> C` violates BCNF, split into `R1(B, C)` and `R2(A, B)`.

Both schemas are in BCNF, and this particular decomposition preserves dependencies: each original FD can be checked in one table.

### Implied Dependencies After Decomposition

```text
R(A, B, C, D, E)
F = { A -> B, BC -> D }
```

1. `A -> B` violates BCNF, so split into `R1(A, B)` and `R2(A, C, D, E)`.
2. Compute closures using the full set $F$. Since $(AC)^+ = ABCD$, `AC -> D` holds on `R2`.
3. `AC` does not determine `E`, so it is not a superkey for `R2`. Split `R2` into `R2.1(A, C, D)` and `R2.2(A, C, E)`.

The final schemas are `AB`, `ACD`, and `ACE`, all in BCNF. The decomposition is lossless but does not preserve `BC -> D`.

**Do not test a decomposed schema using only the original FDs that fit inside it.** Implied dependencies may pass through attributes outside that schema. Compute closures under $F$, then consider the attributes within the schema being tested.

## 3NF Decomposition

The 3NF synthesis procedure guarantees both losslessness and dependency preservation:

1. Compute a canonical cover $F_c$ of $F$.
2. For each dependency $\alpha \rightarrow \beta$ in $F_c$, create a schema containing $\alpha \cup \beta$.
3. If no created schema contains a candidate key for the original $R$, add a schema containing any candidate key of $R$.
4. Optionally remove schemas contained in another schema.

Use a **canonical cover**, not $F^+$, to avoid unnecessary work and redundant schemas.

### Finding and Checking a Candidate Key

To check whether a created schema $R_i$ contains a candidate key for $R$, compute $R_i^+$ under $F_c$. If the closure contains all attributes of $R$, no additional key relation is needed.

To find one candidate key:

* Start with a superkey, using all attributes of $R$ if necessary.
* Remove attributes one at a time whenever the remaining set still determines $R$.
* Stop when no attribute can be removed.

Attributes that never appear on the right side of any FD must occur in every candidate key.

### Example: Canonical Cover

```text
R(A, B, C, D)
F = { AB -> CD, B -> C, AC -> B, B -> D }
```

The candidate keys are `AB` and `AC`. The dependency `B -> D` violates 3NF because `B` is not a superkey and `D` is not prime.

A canonical cover is `Fc = { B -> CD, AC -> B }`. Create:

* `R1(B, C, D)`;
* `R2(A, B, C)`.

`R2` contains a candidate key for $R$, and neither schema is redundant. The result is lossless, dependency preserving, and in 3NF.

### Example: Adding a Key Relation

```text
R(A, B, C, D)
F = { A -> B, B -> C }
```

`A` and `D` must occur in every candidate key, and $(AD)^+ = ABCD$. Thus, `AD` is the only candidate key.

The canonical cover is already $F$. Initially create `R1(A, B)` and `R2(B, C)`. Neither contains a candidate key for $R$, so add `R3(A, D)`.

The final decomposition is `AB`, `BC`, and `AD`. The first two relations preserve the dependencies; the key relation ensures losslessness and retains `D`.

## Choosing a Decomposition

| Procedure | Resulting normal form | Lossless-join | Dependency preserving |
| --- | --- | --- | --- |
| BCNF decomposition | BCNF | Always | Not always |
| 3NF synthesis | 3NF | Always | Always |

BCNF places stricter limits on redundancy. 3NF allows some redundancy so that all functional dependencies can be enforced locally.

## Relationship to SQL

SQL can enforce functional dependencies using primary keys, uniqueness constraints, and triggers. Sometimes key constraints logically imply every FD; otherwise, triggers may be needed.

Assertions express more general constraints, but are expensive to test and are not supported by mainstream databases in the slides' discussion.
