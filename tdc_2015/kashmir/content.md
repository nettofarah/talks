class: center, middle

# Why I hate Caching
## and the things I'm trying to do to change it

---
class: center, middle

# nettofarah

## github.com/nettofarah
## @nettofarah

---
# Senior Product Engineer @ IFTTT

---
# IFTTT

Connect the Apps you Love

---
# IFTTT

- Millions of Users
- Website (desktop & mobile)
- 8 Apps (IF & DO)
- Billions of Requests and Transactions every day

---
# Rails doesn't Scale

Said someone in the Past.
For whatever reason.

---
# It actually does

---
# Scaling Web (Rails) Apps
Assuming everything is in place

--
### Background Jobs (hard)

--
### Queues (hard)

--
### Asset Pipeline (for front-end only)

--
### With a lot of Processes (easy)

--
### With Threads (could be tricky)

---
# Scaling Web (Rails) Apps

##`Rails.cache`

--
- Easy and beautiful at first

--
- Incredibly tricky after some time

---
# Let's Back Up a Bit

---
# What is Caching Anyway?

---
# The 2 Hardest Problems in Computer Science

## Cache Invalidation
## Naming Things

---
# Ok, So I still haven't convinced you

---
# The `Rails.cache` route

---
# Ops, we forgot to remove it from the cache

---
# Just ask her to clear the browser cache

---
# Oh, that number is off because of our cache

---
# No worries, our cache should be clear in about 4 hours

---
# Oh shoot! I forgot to add the line that removes it from the cache

---
# And Some other design problems like

--
## Cache Invalidation in callback hooks

--
## Inconsistent TTLs

--
## Inconsistent Cache Keys

--
## Nested Caching

---
# N+1 Queries (from cache)

---
# Unrealistic Experience

---
# What happens if your cache box dies?


---
# I'm not saying you should abandon caching altogether
But maybe there is a better way

---
# What if I told you
That some times looking up `memcached` could be slower than a lookup to your database

---
# Step 0
Understand your bottlenecks. Most performance challenges tend to live in the IO realm.
That might not be true sometimes, though.

---
# Step 1
Understand your Database.

---
# Step 1 - Understand your Database

## Relational Databases
### `EXPLAIN` and `SLOW LOG` are your best friends
### Spend some time analyzing your indexes (specially compound ones)

---
# Step 1 - Understand your Database

## Search Engines (SOLR, ElasticSearch)

Try and understand how indexes are constructed. When commit operations are performed,
`analyze` your indexes at write and read time.

---
# Step 1 - Understand your Database
## Redis

`SLOWLOG` can save you

---
# Step 2 - Use the best tool for the job

---
# Is your data composed by relationships?
Use SQL

---
# Key value?
You can use Redis for that

---
# Performing a Search?
ElasticSearch and SOLR might be just fast enough for you

---
# Still not convinced?

---
# How to Cache?

---
# HTTP Caching
My favorite kind. If used wisely.

- Headers
- Varnish / Instart Logic

---
# Template Caching
IMO. Doesn't sound like a great practice.

---
# Controller Caching
Very tricky and when it comes to state.

---
# Model Caching
Hard to design around caching in Models.

---
# Russian Doll Caching
Sounds very promising. But you'd have to do some upfront work.

---
# Kashmir
Open Source caching library by IFTTT.

---
# Cache Layers

---
# Database Query Planner
Levarages `ActiveRecord::Associations::Preloader`

---
# Expiration
Trickiest part.

---
# Advanced Use Cases

---
# Questions?











