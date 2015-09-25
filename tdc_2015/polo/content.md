layout: true
.logo[![IFTTT](../ifttt.svg)]

---
class: main, middle

# .blue[POLO]
## Working with Real World data in Development
.right[@nettofarah]

---
class: fake-middle
# nettofarah

## github.com/nettofarah
## @nettofarah

http://bit.ly/polo-rb

---
class: fake-middle
# Real World Production Data in Development

---
class: fake-middle
# Because Managing Data is hard
Especially across Different environments

---
class: fake-middle
# Data in `Rails.env.dev` is...

Ugly, Incomplete, Weird and Biased

<!--
Managing data across different environments is hard. Development data is usually
poor and incomplete, it looks ugly and weird.

Another problem with development data is that it usually doesn't refflect corner
cases. We (developers) know the codebase. We know the happy path.
We'll hardly ever be testing our features with the full picture in mind.

-->

---
class: fake-middle
# Some Consequences of Inconsistent Data

---
class: fake-middle
# 🐛🐞, 💻 misalignment, form validation, special characters 😖

<!--
Bugs will begin to show up. Elements will not look well aligned on the screen,
form validation is not always gonna pass, some special characters will break your
code, some things will only be testable in production. :O
-->

---
class: fake-middle
# The Worst Consequence

---
class: fake-middle
# `Rails.env.dev`
will look 😢

<!--
We know things are hard by now. Your list views look terrible, with a bunch of
weird characters just thrown around, some colors you just picked randomly.
Comments streams will not look like a conversation. Your dev environment will
look sad :(
-->

---
class: fake-middle
# But how do people solve these problems?

---
class: fake-middle
# We can test it in `PRODUCTION`

---
class: fake-middle
# `Rails.env.staging` ?

---
class: fake-middle
# Custom `db/migrations` for data

---
class: fake-middle
# `rake db:seed`

---
class: fake-middle
# CSVs and Spreadsheets

<!--

A few solutions will come to your mind:
- Let's just test these things in production (hope that is not really an option)
- Let's build a staging environment!
- We can write migrations
- `rake db:seed`
- CSVs and Google Spreadsheets!

These solutions might work if you're on a small team, or if your product hasn't
reached some scale. Unfortunately, most of these solutions will fall short
after some time. And all you have done was just to create some more headaches
for your future you, or even worse, your team.

I've worked in projects where people would spend hours trying to get their
code to a staging environment just so they could see how an html tag would render
in different scenarios.

The problem with pre-production enviroments though is that people will rarelly
love them as much as they do production environments. Which is not to be blamed.
(explore this part a bit more)
-->


---
class: image, main, fake-middle
.kashmir-logo[![POLO](https://raw.githubusercontent.com/IFTTT/polo/images/images/polo.png)]

---
class: fake-middle
# Sample Database Snapshots

<!--
Polo travels through your database and creates sample snapshots so you can work
with real world data in any environment.

-->

---
class: fake-middle
Your good ol'
###`ActiveRecord::Associations`
to
###`.sql`

<!--
Polo takes an ActiveRecord::Base seed object and traverses every whitelisted
ActiveRecord::Association generating SQL INSERTs along the way.

You can then save those SQL INSERTS to .sql file and import the data to your
favorite environment.
-->

---
class: fake-middle
# POLO lets you turn this...

---
class: fake-middle, code-slide

```ruby
class Chef < ActiveRecord::Base
  has_many :recipes
  has_many :ingredients, through: :recipes
end

class Recipe < ActiveRecord::Base
  has_many :recipes_ingredients
  has_many :ingredients, through: :recipes_ingredients
end

class Ingredient < ActiveRecord::Base
end

class RecipesIngredient < ActiveRecord::Base
  belongs_to :recipe
  belongs_to :ingredient
end
```

---
class: fake-middle
# Into this...

---
class: fake-middle, code-slide, short-code

```ruby
inserts = Polo.explore(Chef, 1)

# Chef -> ActiveRecord::Base object
# 1 -> Database ID. (Chef with ID 1)
```

```sql
INSERT INTO `chefs` (`id`, `name`)
  VALUES (1, 'Netto')
```

---
class: fake-middle
# It works with associations too

---
class: fake-middle, code-slide

```ruby
inserts = Polo.explore(Chef, 1, :recipes)

# :recipes ->
#   ActiveRecord::Associations::HasManyAssociation
#
# a Chef has many Recipes
```

```sql
INSERT INTO `chefs` (`id`, `name`)
  VALUES (1, 'Netto')

INSERT INTO `recipes`
    (`id`, `title`, `num_steps`, `chef_id`)
  VALUES
    (1, 'Turkey Sandwich', NULL, 1)

INSERT INTO `recipes`
    (`id`, `title`, `num_steps`, `chef_id`)
  VALUES
    (2, 'Cheese Burger', NULL, 1)
```

---
class: fake-middle
# It also works with nested associations

---
class: fake-middle, code-slide, long-code
```ruby
inserts = Polo.explore(Chef, 1, { :recipes => :ingredients })
# { :recipes => :ingredients } ->
#    load every recipe and ingredientes
```

```sql
...
INSERT INTO `recipes` (`id`, `title`, `num_steps`, `chef_id`)
  VALUES (1, 'Turkey Sandwich', NULL, 1)

INSERT INTO `recipes` (`id`, `title`, `num_steps`, `chef_id`)
  VALUES (2, 'Cheese Burger', NULL, 1)

*INSERT INTO `recipes_ingredients`
*  (`id`, `recipe_id`, `ingredient_id`)
*  VALUES (1, 1, 1)

*INSERT INTO `recipes_ingredients`
*  (`id`, `recipe_id`, `ingredient_id`)
*  VALUES (2, 1, 2)
...

INSERT INTO `ingredients` (`id`, `name`, `quantity`)
  VALUES (1, 'Turkey', 'a lot')

INSERT INTO `ingredients` (`id`, `name`, `quantity`)
  VALUES (2, 'Cheese', '1 slice')
...
```

---
class: fake-middle
#🎩
# Show me the magic!

---
class: fake-middle
# Collect all the objects

[lib/polo/collector.rb#L16](https://github.com/IFTTT/polo/blob/master/lib/polo/collector.rb#L16)

---
class: fake-middle, code-slide

```ruby
asn = ActiveSupport::Notifications
# Extracted so this can fit in a slide

asn.subscribed(collector, 'sql.active_record') do
# Set up ActiveRecord::Preloader
*  base_finder = @base_class.
*    includes(@dependency_tree)
*    .where(id: @id)

# Store SELECTS in some global storage
  collect_sql(@base_class, base_finder.to_sql)
end
```

---
class: fake-middle
#  🔨
# Transform them to SQL
[lib/polo/sql_translator.rb#51](https://github.com/IFTTT/polo/blob/master/lib/polo/sql_translator.rb#51)

---
class: fake-middle, code-slide
```ruby
attributes = record.attributes

keys = attributes.keys.map do |key|
  "`#{key}`"
end

values = attributes.map do |key, value|
  column = record.column_for_attribute(key)
  attribute = cast_attribute(record, column, value)
  connection.quote(attribute)
end

joined_keys = keys.join(', ')
joined_values = values.join(', ')

table_name = record.class.table_name

"INSERT INTO `#{table_name}`" +
" (#{joined_keys}) VALUES (#{joined_values})"
```

---
class: fake-middle
# PROFIT! 💸

---
class: fake-middle
# I need your help! 😬

---
class: fake-middle
# on Github
http://github.com/IFTTT/polo

---
class: fake-middle
# Pull Requests 💕

---
class: fake-middle
# PostgreSQL, SQLite and Oracle support

---
class: fake-middle
# ActiveRecord < 3.2 support

---
class: fake-middle
# Check out some other really cool Open Source projects
http://ifttt.github.io

---
class: fake-middle
# Questions?
