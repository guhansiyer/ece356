# Transactions

A transaction is a *unit* of program execution that accesses and possibly updates various data items. For example, a transcation to tranfer $50 from account A to B:

```c
1. read(A)
2. A := A - 50
3. write(A)
4. read(B)
5. B := B + 50
6. write(B)
```

Two important issues must be dealt with in transaction processing: various failures (hardware, system crash), and concurrent execution of multiple transcations.

## ACID Properties

```c
1. read(A)
2. A := A - 50
3. write(A)
4. read(B)
5. B := B + 50
6. write(B)
```

**Atomicity**: transactions are indivisible and uninterruptible. If $50 is debited from A, then it is *deposited* in B or *credited back* to B.

**Consistency**: transcations must act as transitions from one valid state to another. The total balance of A + B is preserved.

**Isolation**: transcations should be unaware of each other. The resulting state of A and B should be identical if written sequentially or in parallel.

**Durability**: transcations are permanently written to non-volatile storage. If $50 is deposited in B, it will not disappear from B.

### Transaction State

* Active: the initial state; the transaction remains here while executing.
* Partially committed: after the final statement has been executed, before updates are reflected in the database.
* Failed: after discovering that normal execution can no longer proceed, and the transaction must be rolled back.
* Aborted: after the transaction has been rolled back and the database is restored to its state prior to the start of the transaction. After an abort, we can restart or kill the the transaction.
* Committed: After successful completion, updates reflected in the database.

## Transactions in SQL

```sql
/* declare start of transaction */
start transcation;
/*find sum of ECE salaries */
select @A:=sum(salary)
from instructor
where dept_name='ECE';
/* set ECE budget to sum of salaries */
update department
set budget=@A
where dept_name='ECE';
/* try to commit the transaction, making the update durable */
commit;
```

## Concurrent Executions

Concurrency occurs when multiple transactions run simulataneously. It enables greater performance through increased processor and disk utilization, leading to better transaction **throughput**, and through reduced average response time for transactions, since short transactions will not need to wait behind long ones.

Concurrency complicates internal database design. A **concurrency control mechanism** is needed to preserve ACID properties, for example, when two transcations attempt to access the same row in a table.

## Schedules

We need a rigorous framework for reasoning about concurrent transaction execution.

A **schedule** is a sequence of instructions that specify the chronological order in which instructions of concurrent transcations are executed.

A **serial schedule** is one in which a transaction has been started runs to completion before another transaction may start.

A transaction that successfully completes its execution ends with a **commit instruction**.

A transaction that fails to successfully complete its execution ends with an **abort instruction**.

The commit and abort instructions are sometimes omitted for brevity when the outcome of the transaction is clear from the context.

A schedule preserves the order of instructions within each transaction. Our simplified model captures reads and writes on existing data items, but not the creation or deletion of items.

### Schedule Examples

Let `T1` transfer $50 from A to B, and `T2` transfer 10% of A's balance to B:

```text
T1: read(A); A := A - 50; write(A);
    read(B); B := B + 50; write(B); commit
T2: read(A); temp := A * 0.1; A := A - temp; write(A);
    read(B); B := B + temp; write(B); commit
```

Use `r1(A)` for `T1` reading A, `w1(A)` for writing A, and `c1` for committing. Calculations use each transaction's local copies and are omitted below. Suppose initially `A = 1000` and `B = 2000`.

**Schedule 1**: serial, with `T1` before `T2`.

```text
r1(A) w1(A) r1(B) w1(B) c1 r2(A) w2(A) r2(B) w2(B) c2
```

`T1` transfers $50, then `T2` transfers $95. Final balances: `A = 855`, `B = 2145`.

**Schedule 2**: serial, with `T2` before `T1`.

```text
r2(A) w2(A) r2(B) w2(B) c2 r1(A) w1(A) r1(B) w1(B) c1
```

`T2` transfers $100, then `T1` transfers $50. Final balances: `A = 850`, `B = 2150`. Serial schedules can produce different results while both preserving consistency.

**Schedule 3**: interleaved, but equivalent to Schedule 1.

```text
r1(A) w1(A) r2(A) w2(A) r1(B) w1(B) c1 r2(B) w2(B) c2
```

`T2` sees `T1`'s updated A and B. Final balances are again `A = 855`, `B = 2145`. Schedules 1, 2 and 3 all preserve `A + B`.

**Schedule 4**: interleaved, with lost updates.

```text
r1(A) r2(A) w2(A) r2(B) w1(A) r1(B) w1(B) c1 w2(B) c2
```

Both transactions read the initial A and B. `T1` overwrites `T2`'s update to A, and `T2` overwrites `T1`'s update to B. Final balances: `A = 950`, `B = 2100`; the total has increased from $3000 to $3050, violating consistency.

## Serializability

We are assuming that each transaction, when executed in isolation, preserves database consistency. This also implies that any serial schedule containing possibly many transactions also preserves database consistency.

Informally speaking, a schedule is **serializable** if it is equivalent to a serial schedule. As a result, serializable schedules also preserve database consistency even though the transactions may not be serially executed.

Different definitions of schedule equivalence give rise to the formal notions of **conflict serializability** and **view serializability**.

## Conflicting Instructions

> Instructions `I` and `J` of `R` and `T` respectively **conflict** if and only if there exists some item `Q` accessed by both `I` and `J` and at least one of the instructions wrote `Q`.

Intuitively, a conflict between `I` and `J` forces a logical ordering between them. If `I` and `J` are consecutive in a schedule and don't conflict, their results would remain the same even if they had been interchanged in a schedule.

For operations in different transactions on the same item, read-read does not conflict; read-write, write-read and write-write do. Operations on different items do not conflict.

## Conflict Serializability

> Let $S$ and $S'$ be schedules for some set $R$ of transactions. If schedule $S$ can be transformed into schedule $S'$ by a finite series of swaps of non-conflicting instructions, then we say that $S$ and $S'$ are **conflict equivalent**.

We say that a schedule $S$ is **conflict serializable** if it is conflict equivalent to a serial schedule.

Only adjacent instructions from different transactions may be swapped, preserving the instruction order within each transaction.

### Conflict Serializability Examples

```text
S:  r1(A) w2(B) w1(A)
S': w2(B) r1(A) w1(A)
```

Swap `r1(A)` and `w2(B)`: they access different items. `S'` is serial (`T2`, then `T1`), so `S` is conflict serializable.

For Schedule 3, move `r2(A) w2(A)` past `r1(B) w1(B) c1`. The data operations access different items, so the swaps produce Schedule 1. Thus, Schedule 3 is conflict serializable.

An example that is **not conflict serializable**:

```text
r3(Q) w4(Q) w3(Q)
```

`r3(Q)` before `w4(Q)` requires `T3` before `T4`, while `w4(Q)` before `w3(Q)` requires `T4` before `T3`. Neither serial order preserves both conflicts.

## View Serializability

Schedules $S$ and $S'$ over the same transactions are **view equivalent** if, for each data item `Q`:

1. Each read that returns the initial value of `Q` in $S$ also returns the initial value in $S'$.
2. Each read that returns a written value in $S$ reads from the corresponding write of the same transaction in $S'$.
3. The same transaction performs the final write of `Q` in both schedules.

A schedule is **view serializable** if it is view equivalent to a serial schedule.

> Every conflict serializable schedule is also view serializable. The converse does not always hold.

### View Serializability Example

```text
S:  r27(Q) w28(Q) w27(Q) w29(Q)
S': r27(Q) w27(Q) w28(Q) w29(Q)
```

`S'` is serial in the order `T27`, `T28`, `T29`. Both schedules have `T27` read the initial Q and `T29` perform the final write; there are no reads of another transaction's writes. Thus, they are view equivalent.

However, `S` is not conflict serializable: `r27(Q)` before `w28(Q)` requires `T27` before `T28`, while `w28(Q)` before `w27(Q)` requires the reverse. The writes by `T28` and `T29` are **blind writes**: writes without first reading the item.

## Testing for Conflict Serializability

Consider a given schedule for a set of transactions $\{T_1, T_2, ..., T_n \}$. A **precedence graph** is a directed graph where the vertices are the transactions and edges represent conflicting operations. Draw an edge `Ti -> Tj` if an operation of `Ti` occurs before a conflicting operation of `Tj`: write-read, read-write or write-write on the same item `X`.

> Theorem: A schedule is conflict serializable if and only if its precedence graph is acyclic.

Cycle detection can be done in $O(n+e)$ time for $n$ vertices and $e$ edges. If the precedence graph is acyclic, the order of transactions in an equivalent serial schedule can be obtained by a topological sorting of the graph.

An edge `Ti -> Tj` requires `Ti` before `Tj` in every conflict-equivalent serial schedule. Schedule 3 has only `T1 -> T2`; the example `r3(Q) w4(Q) w3(Q)` has the cycle `T3 -> T4 -> T3`.

For a graph with edges `Ti -> Tj`, `Ti -> Tk`, `Tj -> Tm`, and `Tk -> Tm`, two valid serial orders are:

```text
Ti, Tj, Tk, Tm
Ti, Tk, Tj, Tm
```

## Testing for View Serializability

The precedence graph test cannot directly decide view serializability. An acyclic graph implies view serializability, but a cyclic graph may or may not be view serializable.

Checking view serializability is **NP-complete**; extending the graph test can require exponential work. Practical algorithms may check sufficient conditions instead, but can reject schedules that are actually view serializable.

## Recoverable Schedules

Serializability addresses concurrent execution; we must also handle transaction failures.

A schedule is **recoverable** if, whenever `Tj` reads a value written by `Ti` and later commits, `Ti` also commits successfully **before `Tj` commits**.

### Non-Recoverable Example

```text
r8(A) w8(A) r9(A) c9 r8(B) a8
```

Here `a8` denotes an abort. `T9` reads `T8`'s uncommitted A and commits before `T8` aborts. `T9` has already committed a result based on a value that must be undone, so the schedule is not recoverable.

A recoverable completion instead waits for `T8` to commit:

```text
r8(A) w8(A) r9(A) r8(B) c8 c9
```

If `T8` aborts, `T9` must also abort rather than commit. Serializability alone does not guarantee recoverability.

## Cascading Rollbacks

A **cascading rollback** occurs when one transaction's failure forces other transactions to roll back because they read its uncommitted results.

```text
r10(A) r10(B) w10(A) r11(A) w11(A) r12(A) a10
```

`T11` reads from `T10`, and `T12` reads from `T11`. If `T10` aborts, both `T11` and `T12` must also roll back. None has committed, so this remains recoverable, but significant CPU and disk work may be wasted.

## Cascadeless Schedules

A schedule is **cascadeless** if, whenever `Tj` reads a value written by `Ti`, `Ti` commits successfully **before that read**.

```text
r8(A) w8(A) r8(B) c8 r9(A) c9
```

`T9` only reads A after `T8` commits, so an abort by `T8` cannot force `T9` to undo a dependent read.

> Every cascadeless schedule is recoverable, but a recoverable schedule may still require cascading rollbacks.

Recoverability constrains **commit order**; cascadelessness constrains **when reads may occur**. Serial schedules are both recoverable and cascadeless, provided aborted transactions' effects are rolled back before the next transaction starts.

## Concurrency Control

A database should ensure that schedules are:

* conflict or view serializable;
* recoverable, and preferably cascadeless.

Running one transaction at a time guarantees serial schedules but severely limits concurrency. Testing serializability after execution is too late; **concurrency control protocols** must guarantee it by design. Serializability tests help explain why a protocol is correct, even if the protocol does not run those tests internally.

## Weaker Isolation

Some applications tolerate weaker isolation to improve performance, such as computing an approximate total account balance or approximate statistics for query optimization. Choosing an isolation level trades stronger correctness guarantees for greater concurrency.

### Transaction Isolation Anomalies

**Phantom read**: repeating a query returns a different set of matching rows because another transaction changes the data. For example, `T1` queries instructors with `salary < 80000`, `T2` inserts a matching instructor and commits, then `T1` repeats the query and sees an extra row. Our simple read/write model does not capture these insertions and deletions.

**Non-repeatable read**: repeated reads of the same record return different values because another transaction updates it.

```text
r1(A) r2(A) w2(A) c2 r1(A) c1
```

`T1` sees the old A, then the updated A. Conflicts require both `T1 -> T2` and `T2 -> T1`, so the schedule is not conflict serializable.

**Dirty read**: a transaction reads a value written by another transaction that has not yet committed.

```text
w1(A) r2(A) c2 a1
```

This is not cascadeless. It is also not recoverable because `T2` commits before `T1`, which then aborts. A dirty read can occur in a recoverable schedule if the reader waits to commit until the writer commits.

## Transaction Isolation Levels in SQL-92

| Isolation level | Allows phantom reads? | Allows non-repeatable reads? | Allows dirty reads? |
| --- | --- | --- | --- |
| Serializable | No | No | No |
| Repeatable read | Yes | No | No |
| Read committed | Yes | Yes | No |
| Read uncommitted | Yes | Yes | Yes |

These are the SQL-92 guarantees; implementations may provide stronger guarantees. The slides note that MySQL's InnoDB storage engine uses repeatable read by default.

SQL's serializable level requires behavior equivalent to serial execution; it does not prescribe conflict or view serializability as the implementation test. The textbook model also omits insertions and deletions, so checking its item-level conflicts alone does not address phantoms.
