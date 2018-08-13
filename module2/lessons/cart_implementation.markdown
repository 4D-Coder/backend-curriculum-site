---
title: Cart Implementation
length: 120
tags: cart, order
---

## Goals

* represent a cart using a PORO in Rails
  * Start thinking about opportunities for using POROs to extract logic from the controller.
* use a `flash` to send messages to the view
* load an object to be used throughout the app using a `before_action` filter in the ApplicationController

## Structure

* Warm Up
* Code-along

## Vocabulary
* Session
* PORO

## Video

* [Cart and Order Lifecycle](https://vimeo.com/126190416)

## Warm Up

  * How do we make a class in Ruby?
  * How do we make an instance of a class?
  * Where does plain old Ruby data (like a class instance) "live"?

## Intro

We'll build out an app where a user should be able to add songs to their cart. The added songs are not saved to the database until the user has decided so.

## Code-Along

We are going to use the `SetList` project for this example.

### Writing a Test

```ruby
# spec/features/user_can_add_songs_to_their_cart_spec.rb

require 'rails_helper'

RSpec.describe "When a user adds songs to their cart" do
  it "a message is displayed" do
    song = Song.create(title: "Song 1")
    
    visit songs_path

    click_button "Add Song"

    expect(page).to have_content("You now have 1 copy of #{@song.title} in your cart.")
  end
end
```

### Creating a Cart and Adding a Flash Message

Run the test and it complains about not having an "Add Song" button. Let's make a button and talk about what the path should be. Inside of `views/songs/index.html.erb`:

```erb
  <%= button_to "Add Song", carts_path, class: "btn btn-danger" %>
```

Our error now is:

```
undefined local variable or method `carts_path'
```

So we need a cart resource because although we do not want to store the cart information (it changes a lot), it will still need a controller.

```ruby
# config/routes.rb
resources :carts, only: [:create]
```

Now we get a new error:

```
  1) User adds a pen to their cart a message is displayed
     Failure/Error: click_button "Add Song"

     ActionController::RoutingError:
       uninitialized constant CartsController
```

Make a controller: `touch app/controllers/carts_controller.rb`. If you run the test again, it will complain about missing the action `create`. So, inside of the controller file:

```ruby
class CartsController < ApplicationController
  def create
  end
end
```

And now our error is funky, harder to decipher:

```terminal
Unable to find visible xpath "/html"
```

This is because we are sending this action nowhere; Capybara cannot see any HTML to parse. There is no direct view that corresponds with `create` so we need to redirect it somewhere. Let's bring the user back to the songs index.


```ruby
class CartsController < ApplicationController
  def create
    redirect_to songs_path
  end
end
```

Does it work? Start up your server, visit the songs index page, and click the "Add Song" button. Does it redirect you back to the same page? Good.

When you run the test, it fails looking for the content "You now have 1 of #{@song.title} in your cart." This should be a flash message, but right now we don't know which song was added when we click the "Add Song" button. How do we pass a Song ID to the create action? Let's modify our button:

```erb
  <%= button_to "Add Song", carts_path(song_id: song.id), class: "btn btn-danger" %>
```

We can pass key/value pairs in our path helpers, and they get added as query params at the end of our URI path!

And modify our `create` action in the controller:

```ruby
class CartsController < ApplicationController
  def create
    song = Song.find(params[:song_id])
    flash[:notice] = "You now have 1 copy of #{song.title} in your cart."
    redirect_to songs_path
  end
end
```

And display the flash notice using a content tag in the `app/views/application.html.erb` file:

```erb
<% flash.each do |type, message| %>
  <%= content_tag :div, message, class: type %>
<% end %>
```

Great! That test is passing. But what happens if we add two songs?

### Adding Multiple Songs and Updating Our Flash

Let's update our test to check and see.

```ruby
spec/features/user_can_add_songs_to_their_cart_spec.rb

RSpec.describe "When a user adds songs to their cart" do
  it "a message is displayed" do
    ...
  end

  it "the message correctly increments for multiple songs" do
    song = Song.create(title: "Song 1")
    
    visit songs_path

    click_button "Add Song"

    expect(page).to have_content("You now have 1 copy of #{@song.title} in your cart.")

    click_button "Add Song"

    expect(page).to have_content("You now have 2 copies of #{@song.title} in your cart.")
  end
end

```

If we run this now it fails because even though we've added two songs, our flash message will always say that we have one song. We need a way to store information about how many songs have been added.

Thinking through this a little bit, we could store something in the database every time someone adds a songs to their cart, but there are a few drawbacks to that approach:

* It would require multiple updates (and potentially deletions) while someone makes up their mind about what to actually keep in their cart. That's a lot of extra database work.
  * It would also require the user to log in before they could add anything to their cart so that we could create that association in our database anyway.
* It could potentially result in some abandoned database records if a user decides not to finalize their cart contents.

Instead, we need to find a way to store the state of a cart (which songs have been added to our cart and how many of each type). Can we store data in a session?

Let's go back into the CartsController:

```ruby
class CartsController < ApplicationController
  def create
    song = Song.find(params[:song_id])
    session[:cart] ||= Hash.new(0)
    session[:cart][song.id] = session[:cart][song.id] + 1
    flash[:notice] = "You now have #{session[:cart][song.id]} copy of #{song.title} in your cart."
    redirect_to songs_path
  end
end
```

This is close. Our test will still fail, but it looks like our number is incrementing correctly. Our error likely looks something like:

```
  1) User adds a song to their cart the message correctly increments for multiple songs
     Failure/Error: expect(page).to have_content("You now have 2 copies of Song 1 in your cart.")
       expected to find text "You now have 2 copies of #{song.title} in your cart." in ""You now have 2 copy of #{song.title} in your cart.""
     # ./spec/features/user_adds_song_to_cart_spec.rb:25:in `block (2 levels) in <top (required)>'
```

It's the plural of "copy/copies" that's giving us a hard time. Luckily we have a tool for that with `#pluralize`. This is a view helper method, so in order to use it here in our controller to set a flash message, we'll need to include `ActionView::Helpers::TextHelper` explicitly in our controller.

```ruby
class CartsController < ApplicationController
  include ActionView::Helpers::TextHelper

  def create
    song = Song.find(params[:song_id])
    session[:cart] ||= Hash.new(0)
    session[:cart][song.id] = session[:cart][song.id] + 1
    quantity = session[:cart][song.id]
    flash[:notice] = "You now have #{quantity} #{pluralize(quantity, "copy")} of #{song.title} in your cart."
    redirect_to songs_path
  end
end
```

And there we have another passing test.

That's nice, but wouldn't it be better to see how many songs total we have in our cart? Yes. Yes it would.

### Adding a Cart Tracker

First, let's update our feature test.

```ruby
require 'rails_helper'

RSpec.feature "When a user adds songs to their cart" do
  scenario "a message is displayed" do
    ...
  end

  scenario "the message correctly increments for multiple songs" do
    ...
  end

  scenario "the total number of songs in the cart increments" do
    song = Song.create(title: "Song 1")
    
    visit songs_path

    expect(page).to have_content("Cart: 0")

    click_button "Add Song"

    expect(page).to have_content("Cart: 1")
  end
end

```

If we run our test now, we'll see that we have not included "Cart: 0" in our view in any place. It's easy enough to get past that error. Let's open the `app/views/layouts/application.html.erb` file and add a paragraph with the cart information just below our flash message. For now let's hardcode it so that we can see what happens and think through how we want to implement.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>SetList</title>
    <%= csrf_meta_tags %>

    <%= stylesheet_link_tag    'application', media: 'all' %>
    <%= javascript_include_tag 'application' %>
  </head>

  <body>
    <% flash.each do |type, message| %>
      <%= content_tag :div, message, class: type %>
    <% end %>

    <p>Cart: 0</p>

    <%= yield %>
  </body>
</html>
```

Sure enough, that gets us to a new error where now our test is looking for "Cart: 1" and not finding it on our page (because we're still displaying "Cart: 0" since it's hard-coded in our view).

We could potentially pass in a count specifically to use in that slot, but it's starting to feel like we're doing a LOT of logic with this cart. More than should probably just be handled in the controller.

What we *really* want to do here is to have some sort of an object that I can call `#total` on to get the number of objects in the cart. I don't have a model to use since we're not saving the cart in the database, but that doesn't stop me from creating a `Cart` PORO. The question becomes where to start, and what to refactor.

Since we're thinking of implementing this here in the view, let's start there with how we'd *like* this object to behave. Change the new line that we added to the view to the following:

```
<p>Cart: <%= @cart.total_count %></p>
```

Run our test, and ... we've broken everything!!!

```
  3) User adds a song to their cart the total number of items in the cart increments
     Failure/Error: <p>Cart: <%= @cart.total_count %></p>

     ActionView::Template::Error:
       undefined method `total_count' for nil:NilClass`
```

That's okay! We can fix this!!

In our SongsController, let's go ahead and add the instance variable `@cart`. Let's also assume that we'll need to pass it the contents currently sitting in our session to make it work.

```ruby
  def index
    @items = Song.all
    @cart = Cart.new(session[:cart])
  end
```

When we run our tests again, we see another failing test telling us that we don't have a Cart class. At this point, we're going to have to create our PORO, so let's dip down into a model test.

### Creating Our Cart PORO

In our `spec/models` folder, and a new test for our Cart class: `spec/models/cart_spec.rb`. Within our new test file, add the following:

```ruby
require 'rails_helper'

RSpec.describe Cart do

  describe "#total_count" do
    it "can calculate the total number of items it holds" do
      cart = Cart.new({
        1 => 2,  # two copies of song 1
        2 => 3   # three copies of song 2
      })
      expect(cart.total_count).to eq(5)
    end
  end
end
```

Run that test and we find that we need to actually go create our Cart class. In our `app/models` folder, add `cart.rb` and insert the following code:

```ruby
class Cart
  attr_reader :contents

  def initialize(initial_contents)
    @contents = initial_contents
  end

  def total_count
    contents.values.sum
  end
end
```

And that makes our model test pass!

Unfortunately, our feature tests still don't pass for us.

```
  3) User adds a song to their cart the total number of songs in the cart increments
     Failure/Error: contents.values.sum

     ActionView::Template::Error:
       undefined method `values' for nil:NilClass`
```

We're trying to call `#values` on our initial contents, but the first time that we render our `index` our session hasn't been set, so `initial_contents` evaluates to `nil`. Let's fix that by adding some code to ensure that `@contents` is always defined as a hash. In your Cart class:

```ruby
class Cart
  attr_reader :contents

  def initialize(initial_contents)
    @contents = initial_contents || Hash.new(0)
  end

  def total_count
    contents.values.sum
  end
end
```

And now, our feature tests pass as well. Great!

### Refactoring CartsController

Let's take another look at the current status of our CartsController

```ruby
class CartsController < ApplicationController
  include ActionView::Helpers::TextHelper

  def create
    song = Song.find(params[:song_id])
    session[:cart] ||= Hash.new(0)
    session[:cart][song.id] = session[:cart][song.id] + 1
    flash[:notice] = "You now have #{quantity} #{pluralize(quantity, "copy")} of #{song.title} in your cart."
    redirect_to songs_path
  end
end
```

Currently we're doing a lot of work in this controller with our cart. Now that we have a Cart PORO, it seems like we could refactor this to have the PORO take on some of that load.

Let's refactor the CartsController so that it looks like this:

```ruby
class CartsController < ApplicationController
  include ActionView::Helpers::TextHelper

  def create
    song = Song.find(params[:song_id])

    @cart = Cart.new(session[:cart])
    @cart.add_song(song.id)
    session[:cart] = @cart.contents

    flash[:notice] = "You now have #{quantity} #{pluralize(quantity, "copy")} of #{song.title} in your cart."
    redirect_to songs_path
  end
end
```

We're still letting the controller handle getting and setting the session, but we're putting our PORO in charge of managing the hash that we're storing there: both 1) adding songs to it, and 2) reporting on how many of a particular song we have.

Since we've added these methods to our controller, we now have a missing method error when we run our test. Let's add both methods to our model test to see that they do what we expect them to. The `subject` syntax is a nice RSpec feature to use if you're using a similar setup in many tests. In this case, we abstract the logic for creating a `Cart` instance, and can call that return value with `subject`. I like how this reads, but feel free to forego this pattern if you don't like it as much.

```ruby
require 'rails_helper'

RSpec.describe Cart do
  subject { Cart.new({1 => 2, 2 => 3}) }

  describe "#total_count" do
    it "calculates the total number of songs it holds" do
      expect(subject.total_count).to eq(5)
    end
  end

  describe "#add_song" do
    it "adds a song to its contents" do
      subject.add_song(1)
      subject.add_song(2)

      expect(subject.contents).to eq({1 => 3, 2 => 4})
    end
  end

  describe "#count_of" do
    it "reports how many of a particular song" do
      expect(subject.count_of(1)).to eq(2)
      expect(subject.count_of(2)).to eq(3)
    end
  end
end
```

In order to get these tests to pass, add the following two methods to our Cart PORO.

```ruby
def add_song(id)
  contents[id] = contents[id] + 1
end

def count_of(id)
  contents[id]
end
```

What if we ask for the count of a non-existent song?  Add `expect(subject.count_of(0)).to eq(0)` to your test for the `count_of` method. What's the matter, got `nil`? Let's **coerce** `nil` values to `0` with `#to_i`.

```
def count_of(id)
  contents[id].to_i
end
```

And all passing tests!

### Controller Cleanup

One more thing to look at in our Controllers. Notice that in both our CartsController and our SongsController we're setting `@cart` as one of our first actions. This is likely something that we'll continue to want to have access to in our controllers, particularly since we've put a reference to `@cart` in our `application.html.erb`. Let's move that action out to our ApplicationController to ensure that it always gets set.

In ApplicationController, add the following:

```ruby
before_action :set_cart

def set_cart
  @cart ||= Cart.new(session[:cart])
end
```

If for some reason we access the cart more than once in a given request/response cycle, the cart object is memoized with `||=`.

Then delete the following line from both your SongsController and CartsController:

```ruby
  @cart = Cart.new(session[:cart])
```

Double check to see that our tests are still passing, and we should be in good shape!

## Checkin and Review

* Why fuss with all of this PORO business?
* "Where" does PORO data live?
* How do I add a flash message to a view?
* How did we use `before_action` to refactor our controllers?

## Extensions

#### Showing the cart

Let's say that you wanted users to be able to click on "View Cart" (similar to "View Cart" on an e-commerce site).

* Which controller?
* What does the view look like?
* How can we make the cart have access to Cart objects instead of just iterating through a hash of keys and values?

#### Ending and saving the cart contents

What about allowing users to save their cart songs as a package? You might approach it something like this:

```erb
  Cart: <%= @cart.total_count %>
  <%= button_to "Save Package", packages_path %>
```

In your routes:

```ruby
  resources :packages, only: [:create]
```

Make a Packages Controller:

```ruby
class PackagesController < ApplicationController
  include ActionView::Helpers::TextHelper
  def create
    # the four lines below probably would be best delegated to a PackageCreator PORO
    package = Package.new(user_name: "Rachel")
    cart.contents.each do |song_id, quantity|
      package.songs.new(song_id: song_id, quantity: quantity)
    end

    if package.save
      session[:cart] = nil
      flash[:notice] = "Your bag is packed! You packed #{package.songs.count} songs."
      redirect_to songs_path
    else
      # implement if you have validations
    end
  end
end
```
