# Memcached

Databases are fast but they have limitations

SQL can process around 1M small queries per second on commodity hardware, which may be insufficient for small websites.

Scaling a databse across multiple servers is tricky. However, many workloads are read-dominated and can be effectively accelerated by caching the query results.

In-memory caching is becoming increasingly cost effective, as opposed to storing cached data on a hard disk or SSD.

## Overview

Memcached is an in-memory key-value store for small chunks of arbitrary data from results of database calls, API calls, or page rendering.

The basic idea is simple: keep frequently used data in memory so it can be returned very quickly, instead of recomputing it or fetching it from a slower backend database.

## Why Cache?

A cache is useful when:

* the same data is requested repeatedly;
* the data is expensive to compute or fetch;
* read traffic is much heavier than write traffic;
* the backend database is the bottleneck.

The cache sits in front of the database and stores results keyed by a lookup value such as a user ID, product ID, or query string.

If the requested value is present in cache, the system returns it immediately. This is called a **cache hit**. If it is not present, the system fetches it from the database and stores it for future requests. This is a **cache miss**.

The effectiveness of a cache is often measured by **hit rate**:

$$
\text{hit rate} = \frac{\text{cache hits}}{\text{cache hits} + \text{cache misses}}
$$

A higher hit rate means the cache is helping more often.

## Memcached Basics

Memcached stores data as a mapping from a key to a value:

```text
key -> value
```

Example keys might be:

```text
user:42
product:1001
homepage:newsfeed
```

The value can be a string, serialized object, HTML fragment, JSON blob, or any small chunk of data. Memcached does not know the semantics of the value; it just stores bytes and returns them later.

## Common Operations

The basic Memcached operations are:

```text
set key value flags exptime bytes
```

```text
get key
```

```text
delete key
```

A typical example is:

```text
set user:42 {"name":"Alice","role":"student"} 0 0
get user:42
```

Memcached also supports:

* `add`: only stores the value if the key does not already exist;
* `replace`: only stores the value if the key already exists;
* `incr` / `decr`: increment or decrement numeric values;
* `append` / `prepend`: concatenate data to an existing value.

## Expiration and Eviction

Cached values are not permanent. Memcached supports a time-to-live (TTL), also called `exptime`, so items can expire automatically.

This is important because cached data may become stale. For example:

* a user's profile changes;
* inventory changes after a purchase;
* a database record is updated but the old cached copy remains.

When memory fills up, Memcached evicts old or less recently used entries. This is a form of cache replacement policy, not a database consistency mechanism.

## Distributed Caching

Memcached is designed for horizontal scaling. A cache cluster can distribute keys across many servers by hashing the key to determine which server stores it.

This allows a system to grow by adding more cache nodes. The system can then treat the cache as a larger pool of memory without changing application logic.

In a distributed Memcached setup:

* the application computes a hash of the key;
* the hash determines the server responsible for the key;
* the server stores or retrieves the corresponding value.

This is useful when many application servers read the same data concurrently.

## Why Memcached Is Useful

Memcached is especially effective for read-heavy workloads such as:

* session data;
* user profiles;
* shopping cart data;
* frequently used database query results;
* rendered pages or fragments.

For example, instead of running the same SQL query over and over again for a popular page, the system can compute it once and cache the result.

This reduces:

* CPU usage on the database;
* network round trips;
* latency for users;
* load on backend systems.

## Trade-offs and Limitations

Memcached is fast and simple, but it is not a replacement for a database.

Important limitations include:

* data is stored in memory, so it is lost on restart unless reloaded;
* Memcached is not durable; it is a cache, not a primary data store;
* cache invalidation can be tricky when data changes in the database;
* stale cached values may be returned unless invalidated or expired;
* it is optimized for small objects, not very large datasets;
* it does not support complex relational queries.

## Typical Use Pattern

A standard caching workflow is:

1. Application receives a request.
2. It looks up the item in Memcached using a key.
3. If the key is found, the value is returned immediately.
4. If the key is missing, the application reads from the database.
5. The result is written to Memcached for future requests.

This pattern can dramatically reduce the database read load for popular items.

## Example: Query Result Caching

Instead of repeatedly executing:

```sql
select *
from product
where product_id = 1001;
```

an application may cache the result under a key like:

```text
product:1001
```

Then future requests for the same product can be served from memory, without another database query.

## Summary

Memcached is a caching layer designed to improve latency and reduce backend load. It is especially useful for read-heavy workloads and for storing repeated results from expensive operations.

The key ideas are:

* cache frequently used data in memory;
* use a key-value lookup model;
* expire or invalidate entries when data changes;
* accept that caches are temporary, not authoritative data stores.

Memcached is a practical tool for scaling web applications, but it works best alongside a real database that remains the system of record.
