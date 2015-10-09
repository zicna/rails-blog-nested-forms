##Guide to Solving and Reviewing Rails Blogs Nested Forms

###Objective
- Implement a form with nested attributes.

###Step 1: Set up your app
When working with an existing codebase, it's always a good idea to see what our app looks like in our browser. To do this we need to run a few commands in our console.

- `bundle install` Get all our gems in order
- `bin/rake db:migrate RAILS_ENV=test` Setup the DB
- `rails s` then visit `localhost:3000` Start our server
- `rake db:seed` Seed our database


If you visit `http://localhost:3000/posts/new`, you can see we have a form that will let you create a post, along with tags that we can select. Let's go ahead and submit a post to make sure it works. It does, great! So we have a huge selection of tags, but what if we want to add a new one?

###Step 2: Set up nested attributes
The first thing we want to do is set up our `Post` model so it can accept our nested attributes, which will be tags. Rails gives us a method called `#accepts_nested_attributes_for`. "Nested attributes allow you to save attributes on associated records through the parent. By default nested attribute updating is turned off and you can enable it using the `#accepts_nested_attributes_for` class method. When you enable nested attributes an attribute writer is defined on the model."

The attribute writer is named after the association, which means that in the following example, the follow method is added to your model:
`tags_attributes=(attributes)`.

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

Here we are using `fields_for`, which will generate an input for our form. If you inspect the `html` of our form, you can see that our params are set up nicely, thanks to `accepts_nested_attributes_for`, which we have included in our `Post` model.

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

####Before we go any further, let's break down what is happening above.

We are passing the `@post` variable to the `form_for` method. This method also takes in a block with a single argument to the block. That argument is a FormBuilder object. Take a look at <a href="http://guides.rubyonrails.org/form_helpers.html">this</a> documentation to read more. The FormBuilder object that is yielded to the block then contains all the information about the model that is passed in, and has an assortment of useful instance methods.


###Step 3: Set up your PostsContrller

### PostsController

Ok so now we have our model set up and our form in place, let's visit `localhost:3000/posts/new` and see what our updated form looks like. Hmmm, we were expecting a text field so that we can add our new tag, which is what this code should be creating.

[no input screenshot]

```
<div class="field">
  <%= f.fields_for(:tags) do |tag_form| %>
    <%= tag_form.label :name %>
    <%= tag_form.text_field :name %>
  <% end %>
</div>
```
So what is going on? There is actually another thing we need to do for our form to work properly, which is update the `new` action of our `Posts` controller.

```ruby
def new
  @post = Post.new
  @tag = @post.tags.build
end
```
Now if we refresh our browser we will see our updated form.

[new form screenshot]

Our controller is going to set up the data so that our view can render our form. But what is this `build` method? Our `build` method was given to us via our model associations.

It creates a blank object in memory and sets up the association between `Post` and `Tag`. In our case, since we are using `fields_for`, it will display fields for our tags. If you delete `@tag = @post.tags.build` from your controller, your field would disappear. The `build` method creates a single instance, which `fields_for` then iterates over. If it did not exist in your controller, there would be nothing to iterate over.

Additionally, if you called `build`, three times in your controller, it would create three different objects which would in turn give you three separate field in your form.

Ok, so we can build our form dynamically, but how does Rails know where to submit it to? Since we are trying to create a new post and we have our routes set up correctly, Rails knows to submit to `/posts` via a `post` method. Let's take a look at the HTML that was generated for us by `form_for`.

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

    <label for="post_tags_attributes_1_name">Name</label>
    <input type="text" name="post[tags_attributes][1][name]" id="post_tags_attributes_1_name" />

    <label for="post_tags_attributes_2_name">Name</label>
    <input type="text" name="post[tags_attributes][2][name]" id="post_tags_attributes_2_name" />
  </div>

  <div class="actions">
    <input type="submit" name="commit" value="Create Post" />
  </div>

</form>
```

In the first section of our form, we can see our form helper generated `post[name]` and `post[content]`. This is great, we have access to our post name and content.

#### `fields_for`
Let's focus on our tag fields for a second, how did Rails know to create such a nicely nested structure?

If we look at our form, we will see `<%= f.fields_for(:tags) do |tag_form| %>`. Our `fields_for` accepts a `:tags` object and introspects into our `Post` model because the `FormBuilder` object already knows about Post from your form_for. It would be impossible for the fields_for to know it's talking to `Post` without the f thing

If we create three new tags and look at params in our console, we will see the following in our `attributes_hash`.

```ruby
{
    "0" => {
        "name" => "tag1"
    },
    "1" => {
        "name" => "tag2"
    },
    "2" => {
        "name" => "tag3"
    }
}
```
Great! It looks like our `tags_attributes=` method is iterating over our new tags, grabbing the values and building our tags. Then when `@post` is saved, it calls our `tag_attributes=` method in the `Post` model.


Let's take a look at what our params would look like if we submit this form.

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
            },
            "1" => {
                "name" => "tag2"
            },
            "2" => {
                "name" => "tag3"
            }
        }
    },
                "commit" => "Create Post",
            "controller" => "posts",
                "action" => "create"
}
```
As you can see, along with our post `name` and `content` we have a nested hash called `tags_attributes`. These attributes will be written via our `accepts_nested_attributes_for` or custom `tags_attributes=` method.

In order for us to successfully save our new post with its tags, we have to remember to whitelist our new data.

##Step 4: Update `post_params` in your `PostsController`


```ruby
def post_params
  params.require(:post).permit(:name, :content, :tag_ids => [], :tags_attributes => [:name] )
end

```
At this point you should be able to create a new post with related tags.
