layout: true
.logo[![IFTTT](../ifttt.svg)]

---
class: middle, main

# Why I hate .blue[Caching]
## and the things I'm trying to do to .blue[change] it
.right[@nettofarah]

---
class: fake-middle

# nettofarah

## github.com/nettofarah
## @nettofarah

http://bit.ly/kashmir-netto

---
class: fake-middle
# Senior Product Engineer @ IFTTT

---
class: fake-middle
# IFTTT

Connect the Apps you Love

---
class: fake-middle
# IFTTT

- Millions of Users
- Website (desktop & mobile)
- 8 Apps (IF & DO)
- Billions of Requests and Transactions every day

---
class: fake-middle
# Rails doesn't Scale

Said someone in the Past.
For whatever reason.

---
class: fake-middle
# Scaling Web (Rails) Apps
You can do it!

---
class: fake-middle
# Background Jobs

---
class: fake-middle
## With a lot of Processes

or

## With Threads

---
class: fake-middle
# Scaling Web (Rails) Apps
with

## `Rails.cache`

---
class: fake-middle
# Easy and beautiful at first

---
class: fake-middle
# Incredibly tricky after some time

---
class: fake-middle
# Caching is like Cocaine

---
class: fake-middle
# What is Caching Anyway?

---
class: fake-middle
# The 2 Hardest Problems in Computer Science

---
class: fake-middle

## Cache Invalidation
and
## Naming Things

---
class: fake-middle
# Ok, so you're still not convinced

---
class: fake-middle
# Oops, we forgot to remove it from the cache

---
class: fake-middle
# Just ask them to clear the browser cache

---
class: fake-middle
# Oh, that number is off because of our cache

---
class: fake-middle
# No worries, our cache should be clear in about 4 hours

---
class: fake-middle
# Oh shoot! I forgot to add the line that removes it from the cache

---
class: fake-middle
# Which leads to frustration...

---
class: fake-middle, no-logo
background-image: url(images/cache2.png)
---
class: fake-middle, no-logo
background-image: url(images/cache3.png)
---
class: fake-middle, no-logo
background-image: url(images/cache4.png)
---
class: fake-middle, no-logo
background-image: url(images/cache5.png)

------
class: fake-middle, no-logo, image
background-image: url(images/cache1.png)

---
class: fake-middle
# And Some other design problems like

---
class: fake-middle
# Cache Invalidation in callback hooks

---
class: fake-middle
# Inconsistent TTLs

---
class: fake-middle
# Inconsistent Cache Keys

---
class: fake-middle
# Nested Caching

---
class: fake-middle
# N+1 Queries
from cache

---
class: fake-middle
# Unrealistic Experience

---
class: fake-middle
# What happens if your cache box dies?

---
class: fake-middle
# I'm not saying you should abandon caching altogether
But maybe there is a better way

---
class: fake-middle
# What if I told you
That some times looking up `memcached` could be slower than performing a lookup in your database

---
class: fake-middle
# Step 0

## Understand your bottlenecks

---
class: fake-middle
# Most performance challenges are IO bound

---
class: fake-middle
# That might not be true sometimes, though

---
class: fake-middle
# Step 1
## Understand your Database

---
class: fake-middle
# `EXPLAIN` and `SLOW LOG` are your best friends

---
class: fake-middle
# Spend some time analyzing your indexes
specially compound ones

---
class: fake-middle
# Search Engines (SOLR, ElasticSearch)

Try and understand how indexes are constructed. When commit operations are performed,
`analyze` your indexes at write and read time.

---
class: fake-middle
# Redis

`SLOWLOG` can save you

---
class: fake-middle
# Step 2
## Use the best tool for the job

---
class: fake-middle
# Still not convinced?

---
class: fake-middle
# How to Cache?

---
class: fake-middle
# HTTP Caching
My favorite kind. If used wisely.

- Headers
- Varnish / Instart Logic

---
class: fake-middle
# Template Caching
IMO, Doesn't sound like a great practice.

---
class: fake-middle
# Controller Caching
Very tricky and when it comes to state.

---
class: fake-middle
# Model Caching
Hard to design

---
class: fake-middle, image, kashmir
.kashmir-logo[![Kashmir](images/kashmir.jpg)]

---
class: code-slide, fake-middle
```ruby
class Person
  include Kashmir

  def initialize(name, age)
    @name = name
    @age = age
  end

  representations do
    rep :name
    rep :age
  end
end

Person.new('Netto Farah', 27).represent(:name, :age)

=> {:name=>"Netto Farah", :age=>"27"}
```
---
class: fake-middle, code-slide, long-code

.left-column[```ruby
class Recipe < OpenStruct
  include Kashmir

  representations do
    rep(:title)
    rep(:chef)
  end
end


class Chef < OpenStruct
  include Kashmir

  representations do
    rep(:name)
  end
end
```]

.right-column[```ruby
netto = Chef.new(
  name: 'Netto Farah'
)

beef_stew = Recipe.new(
  title: 'Beef Stew',
  chef: netto
)

beef_stew.represent(
  :title,
  { :chef => :name }
)

=> {
  :title => "Beef Stew",
  :chef => {
    :name => 'Netto Farah'
  }
}
```]

---
class: fake-middle
# And Many More Examples

https://github.com/IFTTT/kashmir

---
class: fake-middle
# Cache Layers

---
class: image, no-logo, layers
background-image: url(images/layers.png)

---
class: fake-middle
# Database Query Planner
Levarages `ActiveRecord::Associations::Preloader`


---
class: fake-middle
# Prevents N+1 Queries

---
class: fake-middle, code-slide

```ruby
Chef.all.each do |chef|
  chef.recipes.to_a
end
```
generates

```SQL
SELECT * FROM chefs

SELECT "recipes".*
  FROM "recipes" WHERE "recipes"."chef_id" = 1

SELECT "recipes".*
  FROM "recipes" WHERE "recipes"."chef_id" = 2

  ...

SELECT "recipes".*
  FROM "recipes" WHERE "recipes"."chef_id" = N
```

---
class: fake-middle, code-slide

```ruby
Chef.all.represent([:recipes])
```

generates

```SQL
SELECT "chefs".* FROM "chefs"

SELECT "recipes".* FROM "recipes"
  WHERE "recipes"."chef_id" IN (1, 2)
```

---
class: fake-middle
# Expiration
Trickiest part.

---
# Advanced Use Cases

- Rest APIs
- GraphQL like endpoints

---
class: fake-middle
# Questions?
