#Guide to Solving and Reviewing Rails Blogs Nested Forms
##Overview
This lab will teach you how to implement nested forms. In this example, we will nest tags under posts, so a user can submit a post that has many tags.

###Step 1: Set up your app
- `bin/rake db:migrate RAILS_ENV=test`
- `bundle install`
- `rails s` then visit `localhost:3000`
- `rake db:seed`

At this point we should have a form that will let us create a post, along with tags that we can select. Now we need a way to add a new tag, so let's add that to our form.

###Step 2: Use fields_for to accept tags

```ruby
<div class="field">
  <%= f.fields_for(:tags) do |tag_form| %>
    <%= tag_form.label :name %>
    <%= tag_form.text_field :name %>
  <% end %>
</div>

```
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

Our `form_for` accepts `@post` as an argument, which is an instance of our `Post` class. We then pass it to the variable `f` to build our form.

Let's take a look at `@post` params in our console and see what is being returned. 


```ruby
#<Post:0x007fb3ad9e4e50> {
            :id => 10,
          :name => "Hello World!",
       :content => "We are learning about fields_for!",
       :user_id => nil,
    :created_at => Thu, 09 Jul 2015 16:59:41 UTC +00:00,
    :updated_at => Thu, 09 Jul 2015 16:59:41 UTC +00:00
}
```
When we call `<%= f.label :name %>` in our form above, we are accessing the `:name` key in our hash, which is going to return the name of that post, same thing for `:content`, or any other key that is available in our params.

###Step 3: Set up your PostsContrller

### PostsController

```ruby
def new
  @post = Post.new
  @tag = @post.tags.build
end
```

Our controller is going to render our form, but what is this `build` method? Our `build` method was given to use via our model associations, which we will take a look at later. It creates a new object in memory, so that our view can use it to display something. In our case, since we are using `fields_for`, it will display fields for our tags! If you delete `@tag = @post.tags.build` from your controller, your fields would disappear.

Ok, so we can build our form dynamically, but how does Rails know where to submit it to? Luckily for us, Rails is really smart. When we passed the `@post` object to `form_for`, it was able to check for an instance of that post in the database. Since it was a new instance, by convention it decided that we are trying to create a new post and should be submitted to `/posts` via a `post` method. Let's take a look at the HTML that was generated for us by `form_for`.

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

If we look at our form, we will see `<%= f.fields_for(:tags) do |tag_form| %>`. Our `fields_for` accepts a `:tags` object and by convention introspects into our `Post` model to see if it has a way to accept this new object. It is specifically looking for a macro called `accepts_nested_attributes_for :tags`. If it finds it, it will generate the nicely nested form that we see above.

In order to understand `accepts_nested_attributes_for`, we need to take a look at our models associations and the methods that were made available to us.

***note that we have added `accepts_nested_attributes_for :tags` to our `Post` model**

##Step 4: Set up Post model
### Models

```ruby
class Post < ActiveRecord::Base
  has_many :post_tags
  has_many :tags, :through => :post_tags

  accepts_nested_attributes_for :tags
end

class PostTag < ActiveRecord::Base
  belongs_to :post
  belongs_to :tag
end

class Tag < ActiveRecord::Base
  has_many :post_tags
  has_many :posts, :through => :post_tags
end
```
By creating these associations, Rails gives us the methods below.

```ruby
tag_ids()                    Post (Post::GeneratedAssociationMethods)
tag_ids=(ids)                Post (Post::GeneratedAssociationMethods)
tags(*args)                  Post (Post::GeneratedAssociationMethods)
tags=(value)                 Post (Post::GeneratedAssociationMethods)
tags_attributes=(attributes) Post (Post::GeneratedAssociationMethods)
```
What we are interested in right now is `tags_attributes=(attributes)`

As mentioned before, our `fields_for` is looking for `accepts_nested_attributes_for :tags`. But what exactly does this macro give us? Let's define our own and see.

```ruby
def tags_attributes=(attributes_hash)
  attributes_hash.values.each do |attributes|
    self.tags.build(attributes)
  end
end
```
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

##Step 5: Update `post_params` in your `PostsController`


```ruby
def post_params
  params.require(:post).permit(:name, :content, :tag_ids => [], :tags_attributes => [:name] )
end

```
At this point you should be able to create a new post with related tags.