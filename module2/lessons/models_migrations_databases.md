---
layout: page
title: Models, Migrations, and Databases
tags: migrations, databases, relationships, rails, activerecord
---

## Models, Migrations, and Databases in Rails

In this lesson, we'll be adding to our new SetList Rails app to demonstrate a one-to-many and a many-to-many relationship.

We'll add two tables (`artists`, and `playlists`) to our database, and connect them to our existing `songs` table. What might the relationships look like? 

## Learning Goals

* Write migrations in Rails
* Create one-to-many relationships at the database level using foreign keys.
* Create many-to-many relationships at the database level using join tables with foreign keys.
* Use `has_many` and `belongs_to` to create one-to-many and many-to-many relationships at the model level.

## Vocab 
* Migration
* Schema
* Relationships

## WarmUp

* In your own words, what is a migration?
* What are some things that we can do with a migration?
* What is the relationship between a migration, our database, and our schema?
* What does `rake db:rollback` do? When **wouldn't** I want to use it?

## One-to-Many Relationships

### At the Database Level: Artist

We want to create some artists with a name. Let's add a test for that! Since this will be a model test, we need to first make a `/models` directory nested under `/spec` then create a `artist_spec.rb`

`mkdir spec/models`  
`touch spec/models/artist_spec.rb`


We're going to use the handy dandy gem [shoulda-matchers](https://github.com/thoughtbot/shoulda-matchers) to give us some streamlined syntax to use in testing our validations and relationships.

- Add `gem 'shoulda-matchers', '~> 3.1'` to `group :development, :test` in your `Gemfile`  
- run `bundle update`
- Put the following in `rails_helper.rb`:

```ruby 
Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec

    # Choose one or more libraries:
    with.library :active_record
    with.library :active_model
    with.library :action_controller
    # Or, choose the following (which implies all of the above):
    with.library :rails
  end
end
```

In `artist_spec.rb`

```ruby
require 'rails_helper'

describe Artist, type: :model do
  describe "validations" do
    it {should validate_presence_of(:name)}
  end 
end

```

When we run rspec, we get an error similar to this:

```ruby 
# --- Caused by: ---
     # PG::UndefinedTable:
     #   ERROR:  relation "artists" does not exist
     #   LINE 8:                WHERE a.attrelid = '"artists"'::regclass
     #                                             ^
     #   ./spec/models/artist_spec.rb:5:in `block (2 levels) in <top (required)>'
```

Let's make an Artist!:

1. First, we'll create a migration and a model:

`rails generate migration CreateArtists name:string`

or

`rails g migration CreateArtists name`

The migration generator creates a migration and if we follow the working convention for rails the migration will be pre-populated.

Let's look at the migration inside of `db/migrate`:

```ruby
class CreateArtists < ActiveRecord::Migration
  def change
    create_table :artists do |t|
      t.string :name

      t.timestamps
    end
  end
end
```

Now create a model file. `touch app/models/artist.rb`.
 Inside `artist.rb` add the code that hooks up our model to ActiveRecord.

```ruby
class Artist < ApplicationRecord
end
```

Let's run rspec again.  

```ruby 
Failures:

  1) Artist should validate that :name cannot be empty/falsy
     Failure/Error: it {should validate_presence_of(:name)}

       Artist did not properly validate that :name cannot be empty/falsy.
         After setting :name to ‹nil›, the matcher expected the Artist to be
         invalid, but it was valid instead.
     # ./spec/models/artist_spec.rb:4:in `block (2 levels) in <top (required)>'
```

The important part to read here is `Artist did not properly validate that :name cannot be empty/false.`

Let's add a validation to Artist!

```ruby 
class Artist < ApplicationRecord
 validates_presence_of :name 
end
```

Run rspec again and we get passing tests!


### What about Songs?

What's the relationship between song and artist? Draw this out in a diagram to help visualize the relationship.

Let's create a test to help us drive this out.  Add the following to your `artist_spec.rb` within the greater describe Artist block, but outside of the validations block.

```ruby 
describe 'relationships' do
  it {should have_many(:songs)
end 
```

When we run this test we get an error something like this:

```ruby
Failures:

  1) Artist relationships should have many songs
     Failure/Error: it {should have_many(:songs)}
       Expected Artist to have a has_many association called songs (no association called songs)
     # ./spec/models/artist_spec.rb:9:in `block (3 levels) in <top (required)>'
```

The important part to read here `Expected Artist to have a has_many association called songs (no association called songs)` Tells us we are missing a relationship. Let's go make one!


```bash
rails g migration AddArtistsToSongs artist:references
```

Take a look at what this migration creates.

```ruby 
class AddArtistsToSongs < ActiveRecord::Migration[5.1]
  def change
    add_reference :songs, :artist, foreign_key: true
  end
end
```

What else do we need to make this work as expected?

## Associations

### One-to-Many Relationships at the Model Level: Song/Artist

Let's implement some model-level associations using our handy methods.

* `has_many`
* `belongs_to`

Why do we need a foreign key at the database level and the `belongs_to` method in the model? What do each of these things allow for?

*In the console*:

- Create a artist.
- Create a song. 

Did you get an error? Maybe `NameError: uninitialized constant Artist`? 

- Why are we getting this error? 
- What do we need to do to fix this error? 
  - Remember that creating a migration is a separate step from *running* the migration. 
- Hop out of the console and fix the error.

Hop back into the console:  

- What are different ways to associate songs with artists?

Before we move on, let's make sure to circle back and add a relationship validation to `Song`. You may also need to adjust your setup section of your `song_spec.rb` if you already have one.

---

## Many-to-Many: Songs and Playlists?

Let's first add to our diagram the relationship for `songs` and `playlists`.

A playlist can have many songs in it, but an song can ALSO be in many playlists. This is what constitutes a many-to-many relationship. Since neither the song nor the playlist has only ONE of the other (and therefore can't have a foreign key on it) we're going to create a join table `playlist_songs`.

### WAIT: does it matter what we call the join table??

The join table's name doesn't really matter, we could call it `song_playlists` or `playlist_songs`, it's really up to you as the developer. You could even choose to name it `happy_fun_times` but that would be confusing.

When you're thinking about what to call this table, think about how you're likely to use it most within your application. Since our app's goal will be to show a playlist of songs more often, we're going to call it `playlist_songs`.

### Now, where were we?

Let's create a test.

```ruby 
# spec/models/playlist_spec.rb

require "rails_helper"

describe Playlist, type: model do
  describe "relationships" do
    it {should have_many(:songs).through(:playlist_songs}
  end 
end 
```

When we run this, what error do we get? 

```ruby 
# --- Caused by: ---
     # PG::UndefinedTable:
     #   ERROR:  relation "playlist" does not exist
     #   LINE 8:                WHERE a.attrelid = '"playlist"'::regclass
     #                                             ^
     #   ./spec/models/playlist_spec.rb:5:in `block (2 levels) in <top (required)>'
```

Let's write a migration to create Playlists and PlaylistSongs.

```bash
rails g migration CreatePlaylists name:string
```

If we run rspec again, we'll likely get something like this:

```ruby 
# --- Caused by: ---
     # PG::UndefinedTable:
     #   ERROR:  relation "playlist_songs" does not exist
     #   LINE 8:                WHERE a.attrelid = '"playlist_songs"'::regclass
     #                                             ^
     #   ./spec/models/playlist_spec.rb:5:in `block (2 levels) in <top (required)>'
```

Let's create that join table now.

```bash
rails g migration CreatePlaylistSongs playlist:references song:references

rake db:migrate
```

Now create the models to go with these new tables.

How can we get access to another resource through our join table?  

* `has_many :plural_table_name, through: :name_of_joins_table`
* `belongs_to`

Run rspec again. Passing tests?

*In the console*:

Create a playlist.

What are different ways to associate playlists with songs?

Need a refresher on associations? Click [here](http://guides.rubyonrails.org/association_basics.html).

## Notes

* common column types: `boolean`, `string`, `text`, `integer`, `date`, `datetime`
* `rake db:migrate` migrates development
* you (generally) don't need to run `rake db:test:prepare`; running `rake test` will load the schema to the test database

## WrapUp

* What are three different types of table relationships that you might need to implement? In what scenario would you use each?
* What is the syntax for the following migrations in Rails?
  * Create a table
  * Add a column to a table, with or without a data type
  * Add a reference from one table to another
  * Create a joins table
