# Playlister on ActiveRecord

## Objectives

1) Become familiar with ActiveRecord syntax for models and migrations.

2) Learn about ActiveRecord macros for creating associations between models.

## Description

In this lab, we'll be recreating the basic functionality that we already built out in [Playlister-rb](http://learn.co/lessons/940) but this time, using ActiveRecord associations.

We'll have three models: Artists, Songs, and Genres. By writing a few migrations and making use of the appropriate ActiveRecord macros, we want to end up with three models that are associated with one another in a way that makes sense. I should be able to:
- ask an Artist about its songs and genres
- ask a Song about its genre and its artist
- ask a Genre about its songs and artists.

## How ActiveRecord works

As an ORM, ActiveRecord works just like the ORM you've built already in labs like School Domain with DB. It provides its own abstractions and a similar API for interacting with the Database through your object model.

We're going to be using ActiveRecord to do two things:
- store the data in a database (through a schema which we generate with migrations)
- create associations between our objects (through ActiveRecord macros like `belongs_to` and `has_many`).

## Steps

### 1) Models

The first thing you should do is run `rspec` from your command line. You'll probably be able to knock out a few errors pretty quickly here.

It may be useful to go ahead and define your models at this point. You'll need to do this in the `app/models` directory. Two of the files have already been created for you, but you'll need to make one for your `Song` model. Remember, since we're using ActiveRecord, models should inherit from `ActiveRecord::Base`. A sample definition would look like this:

  ```ruby
  class SomeClass < ActiveRecord::Base
  end
  ```

### 2) Migrations

You'll probably get some errors now that are related to the database. This would probably be a good time to write your migrations. Four of the files for these migrations have been created for you in `db/migrate`, but you'll need to add a fifth to make all of the specs pass. Notice that there is a very strong naming convention at play here. In the file `01_create_songs.rb`, there is a migration defined called `CreateSongs`. The filename of the migration, excluding the version number in the first position, `create_songs`, must match up to the migration class defined within the file, `CreateSongs`. If you don't follow this convention, ActiveRecord will throw an error. `separate_words_with_underscores_and_join_them_together_in_a_class_with_capitals` becomes `SeparateWordsWithUnderscoresAndJoinThemTogetherInAClassWithCapitals`.

Take a look at the Rakefile. There are a few rake tasks that will help us with our migrations. These tasks mimic the rake tasks that Rails gives you for free. There's `rake db:migrate` which takes the migrations you've made and applies them to the database. `rake db:drop` drops the tables in the database.

You can run `rake db:migrate` to migrate the database and `rake db:drop` if you need to drop it.

Take a look in `spec_helper.rb` which in a `config.before` block resets the database. In each spec, in an `after` block, the database is cleaned. This creates a clean database environment for each test, to avoid test errors.

Once you've set up your migrations, it's time to create the associations between your models.

### 3) Associations

There are a bunch of ActiveRecord association macros available. Let's work through our domain here to figure out the best ones to use.

I like to think of Song as a connector between Artist and Genre (Taylor Swift isn't in the genre pop unless she writes pop songs, right?). So therefore:

* An Artist has many Songs, and it has many Genres, through Songs.
* A Genre has many Songs, and it has many Artists, through Songs.
* Because of the relationship between Artist/Song and Genre/Song, a Song belongs to an Artist and belongs to a Genre

ActiveRecord has some great macros to achieve the above associations:

* [`has_many`](http://guides.rubyonrails.org/association_basics.html#the-has-many-association)
* [`has_many through`](http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association)
* [`belongs_to`](http://guides.rubyonrails.org/association_basics.html#the-belongs-to-association)

What does this look like in our schema, in our table definitions and structure?

If a Song belongs to an Artist, as well as a Genre, then that means that there are foreign keys on the Songs table that point to the id on the respective tables.

Song would look something like this:

|id |name        |artist_id |genre_id |
|---|------------|----------|---------|
|2  |Shake It Off|1         |1        |

And Artist:

|id |name         |
|---|-------------|
|1  |Taylor Swift |

And Genre:

|id |name |
|---|-----|
|1  |pop  |

The `artist_id` of 1 points to the row in the Artist table where the id is 1. And the `genre_id` of 1 points to the row in the Genre table where the id is 1.

These foreign keys, in conjunction with the ActiveRecord association macros (`belongs_to`, `has_many`, `has_many through`) will allow us query to get an artist's songs or genres, a song's artist or genre, and a genre's songs and artists entirely through ActiveRecord provided methods on our classes.

### Some Notes

1. If you're having issues with your migrations not running or not representing the state of what you think your database should be run:

```
rm db/playlister-dev.db
rm db/playlister-test.db
```

That should clear out your test and development database and force the migrations to run again with:

```
rake db:migrate SINATRA_ENV=development
rake db:migrate SINATRA_ENV=test
```

## Bonus

### Write More Tests

Once you've managed to make the test suite pass, write your own tests. Within these tests, you should exercise working with different ways of creating associations.


### Seed the Database

Write a `db:seed` rake task.

## Resources

* [Rails Guides](http://guides.rubyonrails.org/) - [Active Record Basics](http://guides.rubyonrails.org/association_basics.html)

<p class='util--hide'>View <a href='https://learn.co/lessons/playlister-on-activerecord'>Playlister On Activerecord</a> on Learn.co and start learning to code for free.</p>
