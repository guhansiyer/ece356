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

## Joins

### Theta Join

A theta join is obtained using an `inner join` with an `on` clause. The condition in the `on` clause specifies which tuples should be matched.

For example, to find the course ID, semester, year, and title of each course offered in the Physics department:

```sql
select section.course_id, semester, year, title
from section inner join course
on section.course_id = course.course_id
	 and course.dept_name = 'Physics'
```

This is equivalent to taking the Cartesian product, selecting matching tuples, and then projecting the requested attributes.

### Natural Join

A natural join matches tuples with the same values for all attributes shared by the two relations. It retains only one copy of each common attribute.

```sql
select *
from instructor natural join teaches
```

An inner join with a `using` clause can express the same join condition while retaining one copy of the named attribute:

```sql
select name, course_id
from instructor inner join teaches using (ID)
```

Natural joins should be used carefully. Unrelated attributes with the same name are also equated. For example, joining `instructor`, `teaches`, and `course` naturally would incorrectly require `instructor.dept_name = course.dept_name`.

The following query explicitly joins `course` using `course_id`:

```sql
select name, title
from instructor natural join teaches
		 inner join course using (course_id)
```

## Renaming

SQL allows relations and attributes to be renamed using the `as` clause:

```sql
old_name as new_name
```

An attribute can be renamed in the result of a query:

```sql
select ID, name, salary / 12 as monthly_salary
from instructor
```

Relation aliases are useful when the same relation appears more than once. The following query finds instructors whose salary is higher than some instructor in Physics:

```sql
select distinct T.name
from instructor as T, instructor as S
where T.salary > S.salary
	and S.dept_name = 'Physics'
```

The keyword `as` is optional for relation aliases in many systems, so `instructor as T` can also be written as `instructor T`.

## Set Operations

The set operations `union`, `intersect`, and `except` combine the results of compatible queries. They automatically eliminate duplicates.

Find courses offered in Fall 2009 or Spring 2010:

```sql
(select course_id
 from section
 where semester = 'Fall' and year = 2009)
union
(select course_id
 from section
 where semester = 'Spring' and year = 2010)
```

Find courses offered in both semesters:

```sql
(select course_id
 from section
 where semester = 'Fall' and year = 2009)
intersect
(select course_id
 from section
 where semester = 'Spring' and year = 2010)
```

Find courses offered in Fall 2009 but not Spring 2010:

```sql
(select course_id
 from section
 where semester = 'Fall' and year = 2009)
except
(select course_id
 from section
 where semester = 'Spring' and year = 2010)
```

To retain duplicates, use the multiset versions `union all`, `intersect all`, and `except all`.

If a tuple occurs $m$ times in $r$ and $n$ times in $s$, then it occurs:

* $m+n$ times in `r union all s`;
* $\min(m,n)$ times in `r intersect all s`;
* $\max(0,m-n)$ times in `r except all s`.

## Aggregate Functions

Aggregate functions operate on a multiset of values and return a single value:

* `avg`: average value;
* `min`: minimum value;
* `max`: maximum value;
* `sum`: sum of values;
* `count`: number of values.

Use `distinct` inside an aggregate function to eliminate duplicates, such as `count(distinct ID)`.

For example:

```sql
select avg(salary)
from instructor
where dept_name = 'Physics'
```

```sql
select count(distinct ID)
from teaches
where semester = 'Spring' and year = 2010
```

```sql
select count(*)
from course
```

## Aggregation with Grouping

The `group by` clause divides tuples into groups with the same value for the grouped attributes. For example, to find the average salary in each department:

```sql
select dept_name, avg(salary)
from instructor
group by dept_name
```

Departments with no instructors do not appear in the result. Any attribute in the `select` clause that is not inside an aggregate function must appear in the `group by` list.

The following query is invalid because `name` is neither aggregated nor included in the `group by` list:

```sql
select dept_name, name, avg(salary)
from instructor
group by dept_name
```

### Having Clause

The `having` clause filters groups after they have been formed. In contrast, the `where` clause filters individual tuples before grouping.

Find departments whose average salary is greater than 42000:

```sql
select dept_name, avg(salary)
from instructor
group by dept_name
having avg(salary) > 42000
```

## Ordering Results

The `order by` clause controls the display order of result tuples. Ascending order is the default; use `desc` for descending order and `asc` for ascending order.

```sql
select distinct name
from instructor
order by name
```

Multiple attributes can be used for ordering, each with its own direction:

```sql
select distinct dept_name, name
from instructor
order by dept_name asc, name desc
```

Here, `distinct` applies to combinations of `dept_name` and `name`, not to each attribute separately.

## Nested Subqueries

A subquery is a `select`-`from`-`where` expression nested within another query. Subqueries can be used to test maximum or minimum values, set membership, whether a relation is empty, and other set operations.

### Set Membership

The `in` construct returns true when the value on its left is an element of the relation generated by the subquery on its right. `not in` tests that the value is not a member of the result.

Find courses offered in both Fall 2009 and Spring 2010:

```sql
select distinct course_id
from section
where semester = 'Fall' and year = 2009
	and course_id in (
			select course_id
			from section
			where semester = 'Spring' and year = 2010
	)
```

Find courses offered in Fall 2009 but not Spring 2010 by replacing `in` with `not in`.

### Empty Relations

The `exists` construct returns true when its subquery is nonempty. Conversely, `not exists` returns true when its subquery is empty:

$$
\operatorname{exists}(r) \iff r \ne \varnothing
$$

$$
\operatorname{not\ exists}(r) \iff r = \varnothing
$$

For example, the following query finds courses taught in both Fall 2009 and Spring 2010:

```sql
select S.course_id
from section as S
where S.semester = 'Fall' and S.year = 2009
	and exists (
			select *
			from section as T
			where T.semester = 'Spring' and T.year = 2010
				and S.course_id = T.course_id
	)
```

This is a correlated subquery because the inner query uses a value from the outer query (`S.course_id`).

### Universal Conditions

Nested `not exists` subqueries can express an "all" condition. To find students who have taken every course offered by Biology, search for students for whom there does not exist a Biology course that they have not taken:

```sql
select S.ID, S.name
from student as S
where not exists (
		select C.course_id
		from course as C
		where C.dept_name = 'Biology'
			and not exists (
					select T.course_id
					from takes as T
					where T.course_id = C.course_id
						and T.ID = S.ID
			)
)
```

This pattern is similar to relational division.

### Scalar Subqueries

A scalar subquery yields a single value. It can be used where a single value is expected, but an error occurs if the subquery returns more than one result tuple.

For example, the following query counts the instructors in each department:

```sql
select department.dept_name,
			 (select count(*)
				from instructor
				where department.dept_name = instructor.dept_name)
			 as num_instructors
from department
```

A scalar subquery can also compare an instructor's salary with the budget of their department:

```sql
select name
from instructor
where salary * 10 > (
		select budget
		from department
		where department.dept_name = instructor.dept_name
)
```
