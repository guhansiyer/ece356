# Storage and File Structures

Database storage structures support efficient, reliable storage and retrieval. The main concerns are the properties of storage media, the cost of block I/O, redundancy across disks, and the organization of records within files.

## Physical Storage Media

Storage media differ in:

* **Latency**: time required for an I/O operation.
* **Throughput**: amount of data transferred per second.
* Cost, capacity, and storage density.
* **Volatility**: whether contents survive a loss of power.
* Reliability, including mean time between failures (MTBF) and write endurance.

Performance depends on whether operations are reads or writes, random or sequential, and on how full the device is.

### Storage Hierarchy

The hierarchy moves from fast, small storage toward slower, larger storage:

```text
CPU registers/cache -> main memory -> flash -> magnetic disk -> optical disk/tape
```

* **Registers and CPU cache** use volatile SRAM and are managed by the processor.
* **Main memory** uses volatile DRAM and is managed by the OS/application. A sufficiently small database can fit entirely in memory.
* **Flash memory** is non-volatile and is used in SSDs and USB drives.
* **Magnetic disks** are non-volatile, have mechanical components, and offer lower cost per unit of storage than flash in the slides' comparison.
* **Optical disks** provide removable, non-volatile storage for software, data sets, and backups.

### Hard Disk Drives: HDDs

HDDs store data on rotating platters. Read/write heads move between tracks, which are divided into sectors, typically 512 bytes or 4 KB.

Access latency includes:

* **Seek time**: moving the head to the correct track.
* **Rotational latency**: waiting for the sector to rotate under the head.
* The time required to transfer the data.

Random I/O repeatedly pays positioning costs, so it is much slower than sequential I/O. On-board circuitry controls head movement, schedules requests, and buffers data.

### Solid-State Drives: SSDs

SSDs expose logical sector reads and writes, but internally store data in flash pages grouped into erase blocks.

* Reads and writes operate on pages.
* Overwriting requires erasing an entire block of pages first.
* Erasing is much slower than reading or writing a page.
* Flash cells wear out after repeated write/erase cycles.

The **flash translation layer (FTL)** maps logical addresses to physical locations. It remaps writes, performs wear leveling, and uses spare capacity to manage erase costs and worn-out cells.

The slides illustrate the scale of the performance gap: SSD accesses take microseconds, while HDD accesses take milliseconds. Exact capacities, prices, and performance figures depend on the device.

## Blocks and Storage Access

A **block** is the unit of storage allocation and transfer from the operating system's perspective, commonly 4 KB. A file is a sequence of blocks, also called pages in database storage.

* Smaller blocks require more transfers for the same amount of data.
* Larger blocks can waste more space when only partly filled.
* An OS block, an SSD page, and an SSD erase block are distinct units and need not have the same size.

Database systems minimize block transfers by keeping frequently used blocks in memory.

### Buffer Management

A **buffer** stores a copy of a disk block in main memory. The **buffer pool** is the collection of these buffers.

The **buffer pool manager**:

* allocates buffer space;
* loads blocks from disk;
* evicts blocks when space is needed;
* writes **dirty blocks**, which have been modified, back to disk.

This resembles virtual memory management, but uses policies tailored to database workloads.

## RAID

**Redundant Arrays of Inexpensive/Independent Disks (RAID)** combine multiple physical disks into one logical storage device. Parallelism can improve capacity and performance, while redundancy allows data to survive some disk failures.

Adding disks increases the chance that some disk fails. With independent disks of equal MTBF, the mean interval between disk failures in an $N$-disk array is approximately the individual MTBF divided by $N$. This is different from the mean time to data loss.

### Redundancy and Parallelism

* **Mirroring** stores two copies of each block. Writes update both copies; reads can use either. Data survives one disk failing in a mirrored pair.
* **Parity** stores extra information from which missing data can be reconstructed.
* **Striping** distributes blocks across disks so requests can run in parallel.

With $N$ disks, block $i$ can be assigned to disk $(i \bmod N) + 1$. Striping improves throughput for many independent requests and reduces response time for large requests.

Reliability depends on both failure frequency and repair time: a second failure during recovery can destroy the redundancy needed to recover data.

### RAID 10

RAID 10 combines mirrored disk pairs with block-level striping.

* Two copies of all data: usable capacity is half the raw capacity.
* Reads can run in parallel across both mirrors.
* Each logical write requires two physical writes.
* Recovery copies data from the surviving mirror.
* Well suited to workloads with frequent updates.

An eight-disk array can ideally provide up to eight times one disk's read performance and four times its write performance.

### RAID 5

RAID 5 uses block-level striping with distributed parity. With $N$ total disks, each stripe contains $N - 1$ data blocks and one parity block; the parity location rotates between disks.

* Usable capacity is $(N - 1)/N$ of raw capacity.
* The array can recover from one disk failure by reconstructing data from the remaining disks.
* Recovery requires reading the remaining disks, making it more expensive than copying a RAID 10 mirror.
* Small reads access one data block.
* Small writes normally require reading old data and parity, then writing new data and parity: **four physical I/Os per logical write**.
* Large writes covering complete stripes can calculate parity from the new data without reading the old blocks.

RAID 5 is space-efficient and works well for reads and large writes, but small writes pay a substantial parity-update penalty.

### Capacity and Performance Calculations

**Raw capacity** is total physical storage. **Usable capacity** is the amount available for application data after accounting for redundancy.

**IOPS** measures small I/O operations per second. **Raw IOPS** counts physical disk operations; logical operations may require several physical operations.

The slides use the idealized model:

$$
\text{RAID performance} = \frac{\text{performance of one disk} \times N}{\text{penalty factor}}
$$

Here, $N$ is the total number of disks. Use IOPS for small random operations and bytes per second for large sequential transfers.

| Operation | RAID 10 penalty | RAID 5 penalty |
| --- | --- | --- |
| Small random read | 1 | 1 |
| Small random write | 2 | 4 |
| Large sequential read | 1 | $N/(N - 1)$ |
| Large sequential write | 2 | $N/(N - 1)$ |

For five-disk RAID 5, this gives up to $5\times$ one disk's small-read IOPS, $5/4\times$ its small-write IOPS, and $4\times$ its sequential throughput. These estimates assume sufficient parallelism and exclude recovery overhead.

## File and Record Organization

A database can be viewed as files representing relations, with records representing tuples and fields representing attributes. Files are stored as sequences of blocks/pages.

### Fixed-Length Records

For record length $n$, record $i$ can begin at byte $n(i - 1)$. Access is straightforward, but records may cross block boundaries; a common remedy is to prohibit this.

Deleting a record leaves a choice:

* Shift later records to close the gap: expensive.
* Move the last record into the gap: destroys sort order.
* Leave the gap and link unused slots into a **free list** for reuse.

### Variable-Length Records

Variable lengths arise from fields such as `varchar`, multiple record types in one file, or repeating fields in older data models.

Attributes have a fixed order. Variable-length fields can be represented by fixed-size `(offset, length)` entries, with their contents stored after the fixed-length attributes. A **null bitmap** efficiently records which attributes are null.

### Slotted Pages

A **slotted page** has a header containing:

* the number of record entries;
* the end of free space;
* the location and size of each record.

Records can move within a page during compaction as long as their header entries are updated. External pointers should refer to the slot entry rather than directly to the record's byte location.

## Organization of Records in Files

* **Heap**: place a record anywhere there is space.
* **Sequential**: store records in search-key order, useful when processing the file sequentially.
* **Index-organized table**: store records in a dictionary structure such as a hash table or B-tree. In practice, tables are often B-trees ordered by primary key.

### Sequential File Updates

Pointer chains maintain the logical order of records:

* Deletion updates the chain to bypass the deleted record.
* Insertion locates the correct position and uses nearby space if available.
* If space is unavailable, place the record in an overflow block and update the chain.

Periodic reorganization restores physical sequential order as overflow records accumulate.

## Data Dictionary

The **data dictionary**, or **system catalog**, stores metadata: data about the database itself.

* Relation names, attribute names, types, and lengths.
* View definitions and integrity constraints.
* User and accounting information.
* Statistics, such as the number of tuples in each relation.
* Physical file organization and storage locations.
* Index information.

Metadata can be stored relationally on disk, with specialized in-memory structures for efficient access.
