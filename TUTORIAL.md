# Guide to Solving and Reviewing Playlister On ActiveRecord

## Overview

The objective of this lab is to gain familiarity with setting up models and associations using ActiveRecord migrations and macros like `belongs_to` and `has_many`.

In addition to the basic `belongs_to` and `has_many` associations, this domain model will require a `has_many through`. 

## Steps

### 1) Creating the `songs`, `genres`, and `artists` tables without associations

The first three tests require basic migrations to create these three tables, each having just a `name` column.

There are already some empty migrations created for us in `db/migrate`. In `01_create_songs`, we'll need to add the following:

```ruby
def change
  create_table :songs do |t|
    t.string :name
  end
end
```
Except for the table name, the same code should be added to `02_create_artists` and `03_create_genres`.

Run `rake db:migrate` to run the migrations, then run `rspec` and watch the first three tests pass.

### 2) Connecting Songs and Artists

Next, we need to tell our `Song` model about genres and artists. The spec tells us that a song "has a genre" and "can have an artist". To translate this into code, let's think about _how_ these relationships exist.

First, let's think about songs and artists. Does a song really "have" an artist? If that were the case, an artist would belong to a song. Following that logic, an artist could only be associated with a single song. That does not seem right. In fact, a song `belongs_to` an artist and an artist `has_many` songs.

In order to make that connection in the database, each song will have to have a foreign key, `artist_id`.

Using the fourth migration file `04_add_artist_to_songs`, let's add that column:

```ruby
def change
  add_column :songs, :artist_id, :integer
end
```

Run `rake db:migrate` and you'll see the following in terminal:

```
==  AddArtistToSongs: migrating ===============================================
-- add_column(:songs, :artist_id, :integer)
   -> 0.0009s
==  AddArtistToSongs: migrated (0.0009s) ======================================
```

So we've seen that the migration ran, but when we run `rspec` we have the same number of failures. In order to make this association complete, we need to tell our `Song` and `Artist` models about each other using ActiveRecord macros! 

In the `Song` class:

`belongs_to :artist`

In the `Artist` class:

`has_many :songs`

Now when we run `rspec`, we can see that all of the tests dealing with the association between songs and artists are passing.

Let's take note that by adding the `belongs_to` and `has_many` macros, a whole bunch of new methods were added to our models. Looking at the tests, we can see some of this new functionality:

1. a song can be created with an artist as an attribute (there's an `artist=` method): 

  ```ruby
  artist = Artist.create(name: "The Beatles")
  song = Song.create(name: "Yellow Submarine", artist: artist)
  ```

2. an artist can build or create a song:

  ```ruby
  @prince = Artist.create(name: "Prince")
  
  song = @prince.songs.build(name: "A Song By Prince") # song not persisted yet
  song.save # now the song is persisted along with its association with its artist
  
  song = @prince.songs.create(name: "A Different Song By Prince") # the song is built and saved in one step
  ```
3. an artist can add many songs at the same time

  ```ruby
  song_1 = Song.create(:name => "A Song By Prince")
  song_2 = Song.create(:name => "A Song By Prince 2")
  @prince.songs << [song_1, song_2]
  ```

### 2) Connecting Songs and Genres

If we think about the relationship between songs and genres, we need to answer the question of whether a song can be associated with more than one genre. Though we could build out our schema to support songs having multiple genres, if we look at the readme and the spec, it's clear that this lab only requires that a song be associated with one genre.

The relationship between songs and genres will be the same as the one between songs and artists - a song `belongs_to` a genre and a genre `has_many` songs.

To make this happen, we need a migration to add `genre_id` to the songs table.

Make a new file `05_add_genre_to_songs.rb` in `db/migrate`. The numbers attached to the file names will determine the order in which the migrations get run, so it's important for them to be sequential.

Also remember that ActiveRecord requires that the class name for the migration matches the last part of the file name, so add this code to the new migration file:

```ruby
class AddGenreToSongs < ActiveRecord::Migration
end
```

And here's the code to add inside this migration to give us the `genre_id` column we need:

```ruby
def change
  add_column :songs, :genre_id, :integer
end
```

After this migration runs, we'll need to add the appropriate macros to the `Song` an `Genre` models.

In the `Song` model:
```ruby
belongs_to :genre
```

In the `Genre` model:
```ruby
has_many :songs
```

Now we're down to just 2 failures. Check out the newly passing tests to see all the functionality we got by creating the associations between songs and genres!

### 3) Connecting Artists and Genres

The `Song` model, now that it belongs to both an artist and a genre, has become the central model that connects the other two - in fact, it is a join model.

So how will we complete the connection and have artists know about their genres and genres know about their artists? They will have to go **through** the `songs` table!

An artist is only associated with a genre when it has a song that belongs to that genre, and a genre is only associated with an artist through a song that belongs to that artist. Let's think about it like this: as an artist, I can't claim to be a jazz musician if I've never written or performed any jazz songs! The artist becomes associated with the jazz genre as soon as he or she writes or performs a jazz song. This is a `has_many through` relationship. 

Since we already have `has_many :songs` in our `Artist` and `Genre` models, we can simply add one more macro to each of these models:

In the `Artist` model:

```ruby
has_many :genres, through: :songs
```

And in the `Genre` model:

```
has_many :artists, through: :songs
```

Now the connections between all of our models are complete, and all of the tests pass!
