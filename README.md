#Objectives:
1. Submit a form with nested attributes.
2. Identify the relationship between form_for and fields_for class methods.

# Rails Blog: Complex Nested Forms

We're going to build off our previous iteration of our Blog App, where we created new models for Users and Tags (and applicable associations) and wrote validations. We want to clean up our tagging feature. Our ability to add tags to a new post is super useful, but what if when we're making a new post, we want to add a new tag that isn't in the list? Let's build that out.

***NOTE***: As with much of our Rails curriculum, remember to always use the `--no-test-framework` flag when you generate models, controllers, etc. That way, the Rails generators will not create additional tests on top of the test suite that already comes with the lesson. E.g., `rails g model User username:string email:string --no-test-framework`. However, it is not needed for this lab as we've provided the starter files.

## Tags

1. We need to change the permitted params in our `PostsController` to accept another attribute, `:tags_attributes`, which contains the tag attributes that we need to create a new tag.
2. We also need an `accepts_nested_attributes_for` macro on our `Post` model, which will permit tags to be nested in our new `Post` form.
3. Now we can build a nested form in our `Post` form. Check out the documentation on [Nested Forms](http://guides.rubyonrails.org/form_helpers.html#nested-forms) for help.
4. We should be able to select previously created tags as well as create a new tag.
5. Remember, because we have a uniqueness validation on the name of tag, we will need to account for that.
6. A user shouldn't have to submit a new tag every time they submit a post.

  ```ruby
  class User < ActiveRecord::Base
    has_many :posts
    accepts_nested_attributes_for :posts, reject_if: proc { |attributes| attributes['title'].blank? }
  end
  ```
7. To allow a user to create a new tag, the controller action for a new post should instantiate a new tag. Check out the documentation for the [`fields_for` tag](http://apidock.com/rails/ActionView/Helpers/FormBuilder/fields_for).

[fields_for tag](http://apidock.com/rails/ActionView/Helpers/FormBuilder/fields_for)

[Preventing Empty Records](http://guides.rubyonrails.org/form_helpers.html#preventing-empty-records)