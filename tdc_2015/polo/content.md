class: center, middle

# POLO
## An easy way to work with real world data in development.

---
class: center, middle

# nettofarah

## github.com/nettofarah
## @nettofarah

---
class: center, middle

# Real World Production Data in Development

---

# The Problem

- Managing Data is hard
- Different environments

---

# Data in `Rails.env.dev`

- ugly
- incomplete
- weird
- biased

<!--
Managing data across different environments is hard. Development data is usually
poor and incomplete, it looks ugly and weird.

Another problem with development data is that it usually doesn't refflect corner
cases. We (developers) know the codebase. We know the happy path.
We'll hardly ever be testing our features with the full picture in mind.

-->

---
# Some Consequences

- Bugs will begin to show up
- Misaligment (I'm talking about the screen)
- Form validation will go crazy
- Special Characters :S

<!--
Bugs will begin to show up. Elements will not look well aligned on the screen,
form validation is not always gonna pass, some special characters will break your
code, some things will only be testable in production. :O
-->

---

# The Worst Consequence

## `Rails.env.dev` will look sad :(

<!--
We know things are hard by now. Your list views look terrible, with a bunch of
weird characters just thrown around, some colors you just picked randomly.
Comments streams will not look like a conversation. Your dev environment will
look sad :(
-->

---
# Some Solutions

- Tests in PRODUCTION
- `Rails.env.staging` ?
- Custom `db/migrations` for data
- `rake db:seed`
- CSVs and Spreadsheets

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
# Enters POLO

- Sample Database Snapshots

<!--
Polo travels through your database and creates sample snapshots so you can work
with real world data in any environment.

-->

---

# POLO
## `ActiveRecord::Associations` to `.sql`

<!--
Polo takes an ActiveRecord::Base seed object and traverses every whitelisted
ActiveRecord::Association generating SQL INSERTs along the way.

You can then save those SQL INSERTS to .sql file and import the data to your
favorite environment.
-->

---

# POLO lets you turn this:

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
# Into this:

```ruby
inserts = Polo.explore(Chef, 1)
```

```sql
INSERT INTO `chefs` (`id`, `name`) VALUES (1, 'Netto')
```

---
# It works with associations too

```ruby
inserts = Polo.explore(Chef, 1, :recipes)
```

```sql
INSERT INTO `chefs` (`id`, `name`) VALUES (1, 'Netto')

INSERT INTO `recipes` (`id`, `title`, `num_steps`, `chef_id`)
  VALUES (1, 'Turkey Sandwich', NULL, 1)

INSERT INTO `recipes` (`id`, `title`, `num_steps`, `chef_id`)
  VALUES (2, 'Cheese Burger', NULL, 1)
```

---

# It also works with NxN associations

```ruby
inserts = Polo.explore(Chef, 1, :recipes => :ingredients)
```

```sql
...

INSERT INTO `recipes` (`id`, `title`, `num_steps`, `chef_id`)
  VALUES (1, 'Turkey Sandwich', NULL, 1)

INSERT INTO `recipes` (`id`, `title`, `num_steps`, `chef_id`)
  VALUES (2, 'Cheese Burger', NULL, 1)

*INSERT INTO `recipes_ingredients` (`id`, `recipe_id`, `ingredient_id`)
*  VALUES (1, 1, 1)

*INSERT INTO `recipes_ingredients` (`id`, `recipe_id`, `ingredient_id`)
*  VALUES (2, 1, 2)

...

INSERT INTO `ingredients` (`id`, `name`, `quantity`)
  VALUES (1, 'Turkey', 'a lot')

INSERT INTO `ingredients` (`id`, `name`, `quantity`)
  VALUES (2, 'Cheese', '1 slice')
...
```

---
# Show me the magic!

---
# Collect all the objects

- https://github.com/IFTTT/polo/blob/master/lib/polo/collector.rb#L16

```ruby
ActiveSupport::Notifications.subscribed(collector, 'sql.active_record') do
*  base_finder = @base_class.includes(@dependency_tree).where(id: @id)
  collect_sql(@base_class, base_finder.to_sql)
end
```

---
# Transform them to SQL

- https://github.com/IFTTT/polo/blob/master/lib/polo/sql_translator.rb#51

```ruby
attributes = record.attributes

keys = attributes.keys.map do |key|
  "`#{key}`"
end

values = attributes.map do |key, value|
  column = record.column_for_attribute(key)
  connection.quote(cast_attribute(record, column, value))
end

joined_keys = keys.join(', ')
joined_values = values.join(', ')

table_name = record.class.table_name

"INSERT INTO `#{table_name}` (#{joined_keys}) VALUES (#{joined_values})"
```

---
# PROFIT!

---
# I need your help!

- github.com/IFTTT/polo
- ifttt.github.io
- Pull Requests
- PostgreSQL, SQLite and Oracle implementations
- ActiveRecord < 3.2 support


