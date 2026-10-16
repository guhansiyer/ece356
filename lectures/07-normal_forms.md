# Normal Forms

Normal forms define levels of "goodness" for relation schemas. They help identify redundancy and guide lossless-join decompositions that avoid update, insertion, and deletion anomalies.

**First normal form (1NF)** requires atomic attribute domains. Higher normal forms impose additional restrictions using functional dependencies (2NF, 3NF, BCNF), multivalued dependencies (4NF), or join dependencies (5NF and beyond).

## Third Normal Form: 3NF

A relation schema $R$ is in **third normal form** with respect to a set $F$ of functional dependencies if, for every dependency $\alpha \rightarrow \beta$ in $F^+$ involving only attributes of $R$, at least one of the following holds:

* The dependency is trivial: $\beta \subseteq \alpha$.
* $\alpha$ is a superkey for $R$.
* Each attribute in $\beta - \alpha$ belongs to some candidate key for $R$.

An attribute belonging to any candidate key is called a **prime attribute**. Different attributes on the right side may belong to different candidate keys.

### Testing for 3NF

1. Compute attribute closures and identify all candidate keys.
2. Examine the non-trivial dependencies in $F^+$ restricted to $R$.
3. Check that each determinant is a superkey, or that every newly determined attribute is prime.

If $F$ refers only to attributes of $R$, it is sufficient to check the dependencies in $F$ rather than explicitly computing $F^+$.

> Example: a schema in 3NF

```text
R(A, B, C, D)
F = { A -> B, BC -> AD }
```

The candidate keys are `AC` and `BC`: both determine all attributes, and neither can be reduced. Thus, `A`, `B`, and `C` are prime attributes.

* `A -> B` satisfies 3NF because `B` belongs to candidate key `BC`.
* `BC -> AD` satisfies 3NF because `BC` is a superkey.

Therefore, $R$ is in 3NF even though `A` is not a superkey.

### Instructor and Department Example

```text
inst_dept(ID, name, salary, dept_name, building, budget)

ID -> name, salary, dept_name
dept_name -> building, budget
```

The only candidate key is `ID`. The dependency `dept_name -> building, budget` violates 3NF: `dept_name` is not a superkey, and neither `building` nor `budget` is prime.

Decomposing into the following schemas removes this violation:

* `instructor(ID, name, salary, dept_name)`;
* `department(dept_name, building, budget)`.

Every non-trivial dependency within either schema has a superkey on the left side, so both are in 3NF.

## Boyce-Codd Normal Form: BCNF

A relation schema $R$ is in **Boyce-Codd normal form** with respect to $F$ if, for every dependency $\alpha \rightarrow \beta$ in $F^+$ involving only attributes of $R$, at least one of the following holds:

* The dependency is trivial: $\beta \subseteq \alpha$.
* $\alpha$ is a superkey for $R$.

BCNF removes the prime-attribute exception allowed by 3NF. **Every schema in BCNF is also in 3NF**, but the converse does not always hold. BCNF is sometimes called "3.5NF".

### Testing for BCNF

Compute closures and look for a non-trivial dependency whose left side is not a superkey. A single such dependency proves that the schema is not in BCNF.

As with 3NF, checking only $F$ is sufficient when all dependencies in $F$ refer only to attributes of $R$. This shortcut cannot be applied blindly to smaller schemas after decomposition.

For `R(A, B, C, D)` with `F = { A -> B, BC -> AD }`, $A^+ = AB$. Therefore, `A -> B` violates BCNF because `A` is not a superkey, even though the schema satisfies 3NF.

The decomposed `instructor` and `department` schemas above satisfy BCNF as well as 3NF.

## Limits of BCNF

BCNF handles redundancy expressed by functional dependencies, but other kinds of redundancy can remain.

```text
parent_info(parent_ID, child_name, parent_phone)
F = { child_name, parent_phone -> parent_ID }
```

This schema is in BCNF with respect to $F$ because `(child_name, parent_phone)` is a candidate key. However, if a parent's children and phone numbers are independent, each child is repeated once for every phone number.

For a parent with two children, adding one phone number requires two new tuples. This suggests decomposing into:

* `parent_child(parent_ID, child_name)`;
* `parent_phone(parent_ID, parent_phone)`.

This redundancy is addressed by **multivalued dependencies** and higher normal forms such as 4NF. BCNF and 3NF are usually sufficient for simple schemas.
