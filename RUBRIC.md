# Rails Blog Nested Forms Rubric

## README

- Change the permitted parameters to accept `tags_attributes`.
- Add `accepts_nested_attributes_for` to Post model.
- Build the nested form on the Post form.
    + Hint: `fields_for`
- Select previously existing tags.
- Add validation on the presence of `name` in the tag form.

## RSpec

- As it stands currently, we have one test passing, which is the first test.
- The second test is currently failing: `can create a post with a new tag (FAILED - 1)`.
- This is the first error message: `Unable to find field "Name"`. This refers to the form element. Let's go ahead and add that to the form.

    ```ruby
    # app/views/posts/_form.html.erb

    <div class="field">
    <%= f.fields_for(:tags, Tag.new) do |tag_form| %>
      <%= tag_form.label :name %>
      <%= tag_form.text_field :name %>
    <% end %>
  </div>
    ```

- Once we add that form in, we get a second error message in our RSpec test: `expected to find text "witty" in "Post was successfully created. Name: post title Content: post content Tags: Edit | Back"`. It is expecting to find a tag with the name "witty" on the page. Since we have code on our post show template that is rendering tags, it's probably not persisting the data to our controller. Let's go ahead and fix that up.

    ```ruby
    # app/controllers/posts_controller.rb

    def post_params
      params.require(:post).permit(:name, :content, :tag_ids => [], :tags_attributes => [:name])
    end
    ```

- It's still not showing the code hitting the `post_params` in the posts controller. We still have to add the `accepts_nested_attributes_for` to the Post model.

    ```ruby
    # app/models/post.rb

    class Post < ActiveRecord::Base
      belongs_to :user
      has_many :comments
      has_many :post_tags
      has_many :tags, :through => :post_tags

      validates_presence_of :name, :content

      accepts_nested_attributes_for :tags
    end
    ```

- Now all tests should be passing.
