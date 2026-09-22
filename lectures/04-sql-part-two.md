# SQL Part Deux

This lecture develops SQL data-definition language (DDL) and additional data-manipulation language (DML) features, using the university schema as a running example.

## Running Example

The main relations are:

* `instructor(ID, name, dept_name, salary)`;
* `teaches(ID, course_id, sec_id, semester, year)`;
* `section(course_id, sec_id, semester, year, building, room_no, time_slot_id)`;
* `course(course_id, title, dept_name, credits)`;
* `department(dept_name, building, budget)`.

## Data Definition Language

The SQL **data-definition language (DDL)** specifies information about relations, including:

* the schema for each relation;
* the domain of values associated with each attribute;
* integrity constraints;
* the indexes maintained for each relation;
* the physical storage structure of each relation, such as the choice between InnoDB and MyISAM in MySQL.

## Domain Types in SQL

Common SQL data types include:

* `char(n)`: fixed-length character string of length `n`;
* `varchar(n)`: variable-length character string with maximum length `n`;
* `int`: integer type;
* `smallint`: integer type with a smaller range than `int`;
* `numeric(p, d)`: fixed-point number with `p` significant digits and `d` digits after the decimal point;
* `real`, `double precision`: floating-point types with machine-dependent precision;
* `float(n)`: floating-point number with precision of at least `n` digits;
* `date`: calendar date in `YYYY-MM-DD` format;
* `time`: time of day in `hh:mm:ss` format.

## Creating Tables

An SQL relation is defined with `create table`:

```sql
create table relation_name (
    attribute_1 domain_1,
    ...,
    attribute_n domain_n,
    integrity_constraint_1,
    ...,
    integrity_constraint_k
)
```

For example:

```sql
create table instructor (
    ID char(5),
    name varchar(20),
    dept_name varchar(20),
    salary numeric(8,2)
)
```

## Dropping and Altering Tables

`drop table` deletes a table and its contents:

```sql
drop table student
```

To avoid an error when the table does not exist, use:

```sql
drop table if exists student
```

`delete from` removes all tuples while retaining the table itself:

```sql
delete from student
```

`alter table` can add an attribute or, where supported, remove one:

```sql
alter table relation_name add attribute_name domain
alter table relation_name drop attribute_name
```

When an attribute is added, existing tuples receive `null` for that attribute unless a default is specified.

## Integrity Constraints

SQL supports several common integrity constraints:

* `not null` disallows null values;
* `primary key (A_1, ..., A_n)` identifies a unique tuple;
* `unique (A_1, ..., A_n)` requires the listed attributes to form a superkey;
* `foreign key (A_1, ..., A_n) references relation(B_1, ..., B_n)` defines a foreign key;
* `default V` supplies `V` when no value is provided;
* `check (predicate)` requires the predicate to hold.

For example:

```sql
create table instructor (
    ID char(5),
    name varchar(20) not null,
    dept_name varchar(20),
    salary numeric(8,2) default 0,
    primary key (ID),
    foreign key (dept_name) references department(dept_name)
)
```

### Primary Keys and Unique Constraints

Both primary-key and unique constraints identify superkeys. A table can have at most one primary key but can have multiple unique constraints.

A primary-key attribute cannot be null; declaring a primary key implies `not null`. The treatment of null values in a unique attribute is system-dependent.

```sql
create table customer (
    customer_id int,
    social_insurance_num numeric(9,0),
    first_name varchar(20),
    last_name varchar(20),
    primary key (customer_id),
    unique (social_insurance_num)
)
```

### Foreign Keys

A foreign key in a child table refers to a key in a parent table. The referenced attributes must identify rows in the parent table.

```sql
create table department (
    dept_name varchar(20) primary key,
    building varchar(20) not null,
    budget int
)

create table instructor (
    ID char(5) primary key,
    name varchar(20) not null,
    dept_name varchar(20),
    salary numeric(8,2),
    foreign key (dept_name) references department(dept_name)
)
```

In MySQL, the referenced key must be a superkey of the referenced table, or a prefix of a multi-attribute primary key.

### Referential Actions

Referential actions specify what happens when an update or deletion affects a value referenced by a child table:

* `cascade` updates or deletes matching foreign-key values in the child table;
* `set null` sets matching foreign-key columns to `null`;
* `set default` sets matching foreign-key columns to their default value;
* `no action` rejects the operation and generates an error. MySQL also calls this `restrict`.

For example, this constraint cascades parent-key updates and sets the child foreign key to null on deletion:

```sql
create table instructor (
    ID char(5) primary key,
    name varchar(20) not null,
    dept_name varchar(20),
    salary numeric(8,2),
    foreign key (dept_name) references department(dept_name)
        on update cascade
        on delete set null
)
```

### MySQL Foreign-Key Requirements

MySQL enforces foreign keys only when the relevant requirements are met:

* the parent and child tables use a storage engine that supports foreign keys, such as InnoDB;
* corresponding attributes have similar data types;
* the foreign key and referenced key are indexed.

### Check Constraints

The `check` clause expresses a predicate that must hold for table rows. For example, institutional salary limits can be enforced as follows:

```sql
create table instructor (
    ID char(5) primary key,
    name varchar(20) not null,
    dept_name varchar(20),
    salary numeric(8,2),
    foreign key (dept_name) references department(dept_name),
    check (salary > 50000 and salary < 150000)
)
```

## Data Manipulation Language

### Inserting Rows

Insert a row by providing values in the table's attribute order:

```sql
insert into course
values ('ECE-356', 'Databases', 'ECE', 0.5)
```

Null can be inserted explicitly:

```sql
insert into course
values ('ECE-356', 'Databases', 'ECE', null)
```

It is usually clearer and safer to name the target columns explicitly:

```sql
insert into course (course_id, title, dept_name, credits)
values ('ECE-356', 'Databases', 'ECE', 0.5)
```

### Updating Rows

An `update` changes rows satisfying its `where` clause. For example, give instructors earning less than `$80,000` a 3% raise:

```sql
update instructor
set salary = salary * 1.03
where salary < 80000
```

Different rows can receive different updates using `case`:

```sql
update instructor
set salary = salary * (
    case
        when salary < 80000 then 1.03
        when salary >= 80000 then 1.02
    end
)
```

If no `when` condition matches, a `case` expression returns `null` unless an `else` clause is included.

### Deleting Rows

Without a `where` clause, `delete` removes every row:

```sql
delete from instructor
```

To delete instructors in the Math department:

```sql
delete from instructor
where dept_name = 'Math'
```

To delete instructors whose departments are in the EIT building:

```sql
delete from instructor
where dept_name in (
    select dept_name
    from department
    where building = 'EIT'
)
```

## String Pattern Matching

The `like` operator compares character strings using wildcards:

* `%` matches any substring, including an empty substring;
* `_` matches exactly one character.

Find instructor names containing `dar`:

```sql
select name
from instructor
where name like '%dar%'
```

To match a literal wildcard, specify an escape character. The following pattern matches the string `100 %`:

```sql
like '100 \%' escape '\\'
```

Pattern matching is case-sensitive according to the slides; actual behavior can depend on the database collation.

## Limiting Output

To find the three highest-paid instructors in ECE, sort salaries in descending order and limit the result:

```sql
select name
from instructor
where dept_name = 'ECE'
order by salary desc
limit 3
```

`order by` defaults to ascending order, so `desc` is necessary here. `limit` does not correctly express a top-`k` query when ties should all be included.

## Inner and Outer Joins

A join combines two relations based on a join condition. The **join condition** determines which tuples match, while the **join type** determines how unmatched tuples are handled.

Suppose `course` contains course information and `prereq` contains prerequisite information. Some courses may have no prerequisites, and some prerequisite records may have no matching course.

An inner join returns only matching rows:

```sql
select *
from course inner join prereq
    on course.course_id = prereq.course_id
```

The keyword `inner` can be omitted in many databases.

A left outer join retains every row from the left relation and fills unmatched right-side attributes with `null`:

```sql
select *
from course left outer join prereq
    on course.course_id = prereq.course_id
```

A right outer join retains every row from the right relation:

```sql
select *
from course natural right outer join prereq
```

A full outer join retains unmatched rows from both relations. The `using` clause joins on attributes with the given name and retains one copy of the join attribute:

```sql
select *
from course full outer join prereq using (course_id)
```

Full outer join is not supported by MySQL.

## Views

A **view** is a virtual relation defined by a query. Views can hide sensitive attributes from users. For example, users who need instructor names and departments but not salaries can be given a view containing only those columns.

### Defining Views

```sql
create view view_name as <query expression>
```

The query expression can be any legal SQL query, including a query involving joins or grouping. Once defined, the view name can be used like a relation.

Views are dynamic: changes to the underlying relations cause corresponding changes in the view's result.

### View Examples

Create a view of instructors without salary information:

```sql
create view faculty as
select ID, name, dept_name
from instructor
```

Query the view:

```sql
select name
from faculty
where dept_name = 'Biology'
```

Create a view containing total instructor salary by department:

```sql
create view departments_total_salary(dept_name, total_salary) as
select dept_name, sum(salary)
from instructor
group by dept_name
```

```sql
select *
from departments_total_salary
```

## Auto-Increment Attributes

In MySQL, `auto_increment` automatically generates integer primary-key values:

```sql
create table instructor_auto (
    ID int auto_increment primary key,
    name varchar(20),
    dept_name varchar(20),
    salary numeric(8,2)
)
```

The generated column can be omitted during insertion:

```sql
insert into instructor_auto (name, dept_name, salary)
values ('Watson', 'Biology', 90210)

select *
from instructor_auto
```

The first generated ID is normally `1`. Auto-increment is a database implementation feature, not an ER-model construct, so it does not eliminate the need for weak entity sets.

## Stored Procedures

A **stored procedure** is a database subroutine that centralizes logic that might otherwise be duplicated across applications.

MySQL uses a delimiter change when defining a procedure containing multiple statements:

```sql
delimiter @@

create procedure ProcTopSalary(in target_dept_name varchar(20))
begin
    select max(salary)
    from instructor
    where instructor.dept_name = target_dept_name;
end @@

delimiter ;

call ProcTopSalary('Physics');
```

Stored procedures are persistent database objects and can be removed with:

```sql
drop procedure ProcTopSalary
```

## Cursors

A database **cursor** is a control structure for traversing records, similar to an iterator in Java. Cursors are used inside stored procedures when row-by-row processing is needed.

The following procedure assigns Biology instructors to teach `BIO-101`, one instructor per year starting in 2016:

```sql
delimiter @@

create procedure CursorDemo()
begin
    declare current_year int default 2016;
    declare instructor_id varchar(5);
    declare instructor_dept varchar(20);
    declare done int default 0;

    declare instructor_cursor cursor for
        select ID, dept_name
        from instructor;

    declare continue handler for not found set done = 1;

    open instructor_cursor;

    instructor_loop: loop
        fetch instructor_cursor into instructor_id, instructor_dept;

        if done = 1 then
            leave instructor_loop;
        end if;

        if instructor_dept = 'Biology' then
            insert into teaches
            values (instructor_id, 'BIO-101', 1, 'Summer', current_year);

            set current_year = current_year + 1;
        end if;
    end loop instructor_loop;

    close instructor_cursor;
end @@

delimiter ;

call CursorDemo();
```

The `not found` handler sets `done` when the cursor has no more rows. The loop checks this flag before processing the fetched values.

## Triggers

A **trigger** is procedural code that executes automatically in response to an event on a table or view. MySQL supports `insert`, `update`, and `delete` trigger events, each of which can execute `before` or `after` the event.

For an update trigger, `old` refers to the value before the update and `new` refers to the proposed value. The following trigger rejects salary increases greater than 10%:

```sql
delimiter @@

create trigger SalaryTrigger
before update on instructor
for each row
begin
    if new.salary > old.salary * 1.10 then
        signal sqlstate '45000'
            set message_text = 'increase higher than 10%';
    end if;
end @@

delimiter ;
```

An update that attempts an excessive increase causes the trigger to signal an error:

```sql
update instructor
set salary = salary * 3.14
where salary < 80000
```