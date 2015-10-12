##Guide to Solving and Reviewing Rails Blogs Nested Forms

###Objective
- Submit a form with nested attributes.
- Identify the relationship between `form_for` and `fields_for` class methods.


###Step 1: Set up your app
When working with an existing codebase, it's always a good idea to see what our app looks like in our browser. To do this we need to run a few commands in our console.

- `bundle install` Get all our gems in order
- `bin/rake db:migrate RAILS_ENV=test` Setup the DB
- `rails s` then visit `localhost:3000` Start our server
- `rake db:seed` Seed our database


If you visit `http://localhost:3000/posts/new`, you can see we have a form that will let you create a post, along with tags that we can select. Let's go ahead and submit a post to make sure it works. It does, great!

So we have a huge selection of tags, but what if we want to add a new one?
<img src="https://github.com/learn-co-curriculum/rails-blog-nested-forms/blob/ea837ba87a44f7cb39e8d10233dcd68984f9b40a/app/assets/images/nobuild.jpg?raw=true" width="50%">

###Step 2: Set up nested attributes
The first thing we want to do is set up our `Post` model so it can accept our nested attributes, which will be tags. Rails gives us a method called `#accepts_nested_attributes_for`. "Nested attributes allow you to save attributes on associated records through the parent. By default nested attribute updating is turned off and you can enable it using the `#accepts_nested_attributes_for` class method. When you enable nested attributes an attribute writer is defined on the model."

The attribute writer is named after the association, in our case the following method is added to our `Post` model:
`#tags_attributes=(attributes)`. You can read more about it <a href="http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html">here</a>.

Our model should now look like this:
####`post.rb`

```
class Post < ActiveRecord::Base
  belongs_to :user
  has_many :comments
  has_many :post_tags
  has_many :tags, :through => :post_tags
  accepts_nested_attributes_for :tags
  validates_presence_of :name, :content
end
```

###Step 3: Set up our form
The next piece of this puzzle is our form. We have our attribute writer set up, so now let's add an input to our form so we can actually add a new tag.  

####`posts/_form.html.erb`

Notice we are using `#fields_for`. This method creates a scope around a specific model object, which is `#form_for`, but doesn't create the form tags themselves. This makes `#fields_for` suitable for specifying additional model objects in the same form. Reade more <a href="http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-fields_for">here</a>.

If you inspect the `html` of our form, you can see that our params are set up nicely, thanks to `#accepts_nested_attributes_for`, which we have included in our `Post` model.

`<input type="text" name="post[tags_attributes][0][name]" id="post_tags_attributes_0_name">`

```ruby
<div class="field">
  <%= f.fields_for(:tags) do |tag_form| %>
    <%= tag_form.label :name %>
    <%= tag_form.text_field :name %>
  <% end %>
</div>
```

If you delete `accepts_nested_attributes_for :tags` from your `Post` model and refresh your browser, you will see the params have changed to the following.

`<input type="text" name="post[tags][name]" id="post_tags_name">`

####Completed Form

```ruby
<%= form_for(@post) do |f| %>
  <% if @post.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@post.errors.count, "error") %> prohibited this post from being saved:</h2>
      <ul>
      <% @post.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= f.label :name %><br>
    <%= f.text_field :name %><br>
    <%= f.label :content %><br>
    <%= f.text_area :content %><br>
    Tags:
  </div>

  <div class="field">
    <%= f.fields_for(:tags) do |tag_form| %>
      <%= tag_form.label :name %>
      <%= tag_form.text_field :name %>
    <% end %>
  </div>

  <div class="field">
    <%= f.collection_check_boxes :tag_ids, Tag.all, :id, :name %><br>
  </div>

  <div class="actions">
    <%= f.submit %>
  </div>

<% end %>
```

###Step 3: Set up your PostsContrller

### PostsController

Ok so now we have our model set up and our form in place, let's visit `localhost:3000/posts/new` and see what our updated form looks like. Hmmm, we were expecting a text field so that we can add our new tag, which is what this code should be creating.

```
<div class="field">
  <%= f.fields_for(:tags) do |tag_form| %>
    <%= tag_form.label :name %>
    <%= tag_form.text_field :name %>
  <% end %>
</div>
```
<img src="https://github.com/learn-co-curriculum/rails-blog-nested-forms/blob/ea837ba87a44f7cb39e8d10233dcd68984f9b40a/app/assets/images/nobuild.jpg?raw=true" width="50%">

So what is going on? There is one more thing we need to do for our form to render properly, which is update the `new` action of our `Posts` controller.

```ruby
def new
  @post = Post.new
  @tag = @post.tags.build
end
```
Now if we refresh our browser we will see our updated form.

<img src="https://github.com/learn-co-curriculum/rails-blog-nested-forms/blob/ea837ba87a44f7cb39e8d10233dcd68984f9b40a/app/assets/images/build.jpg?raw=true" width="50%">

Our controller is now setting up the data so that our view can render our form using the`build` method. Our `build` method was given to us via our model associations.

"The `build` method returns one or more new objects of the collection type that have been instantiated with attributes and linked to this object through a foreign key, but have not yet been saved. Note: This only works if an associated object already exists, not if it‘s nil."

If you delete `@tag = @post.tags.build` from your controller, your tag input field would disappear. This is because the `build` method creates a single instance in memory, which `fields_for` then iterates over.

Additionally, if you called `build`, three times in your controller, it would create three different objects which would in turn give you three separate fields in your form.

If we take a look at our `params`, we can see they now have the following structure.
####Params

```ruby
{
                  "utf8" => "✓",
    "authenticity_token" => "VVPsqwm1XkDsJ1XQjf4vpeLEQdNXl+4hAqvU3gL/RIb5v8RKbcmA7r3gRT6eYM/g9wHY5Ymi6w/PPKuHea1XYg==",
                  "post" => {
                   "name" => "Hello World!",
                "content" => "We are learning about fields_for!",
        "tags_attributes" => {
            "0" => {
                "name" => "tag1"
            }
        }
    },
                "commit" => "Create Post",
            "controller" => "posts",
                "action" => "create"
}
```
As you can see, along with our post `name` and `content` we have a nested hash called `tags_attributes`. These attributes will be written via our `accepts_nested_attributes_for` method.

Now we can build our form dynamically, but how does Rails know where to submit it to? Since we are trying to create a new post and we have our routes set up correctly, Rails knows to submit to `/posts` via a `post` method. Let's take a look at the HTML that was generated for us by `form_for`.

####Generated HTML From Ruby

```html

<form class="new_post" id="new_post" action="/posts" accept-charset="UTF-8" method="post">
<input name="utf8" type="hidden" value="&#x2713;" />
<input type="hidden" name="authenticity_token" value="IRnbjg0Lr7XuZmk1wqEewWqPbOvU0j5ymcne3Rq3MV3yHzhC1VbZsySOGggFVgM1zwL6b+E2+nGsnaQCQQfu6g==" />

  <div class="field">
    <label for="post_name">Name</label><br>
    <input type="text" name="post[name]" id="post_name" /><br>
    <label for="post_content">Content</label><br>
    <textarea name="post[content]" id="post_content">
    </textarea><br>
    Tags:
  </div>

  <div class="field">
    <label for="post_tags_attributes_0_name">Name</label>
    <input type="text" name="post[tags_attributes][0][name]" id="post_tags_attributes_0_name" />
  </div>

  <div class="actions">
    <input type="submit" name="commit" value="Create Post" />
  </div>

</form>
```

In order for us to successfully save our new post with its tags, we have to remember to whitelist our new data.

###Step 4: Update `post_params` in your `PostsController`


```ruby
def post_params
  params.require(:post).permit(:name, :content, :tag_ids => [], :tags_attributes => [:name] )
end

```
At this point you should be able to create a new post with related tags.
