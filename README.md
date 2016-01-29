
# Rails Blog: Complex Nested Forms

We're going to build off our previous iteration of our Blog App, where we created new models for Users and Tags (and applicable associations) and wrote validations. We want to clean up our tagging feature. Our ability to add tags to a new post is super useful, but what if when we're making a new post, we want to add a new tag that isn't in the list? Let's build that out.

<em>Before anything</em>, note that when you generate models, controllers, etc, be sure to include this option, so that it skips tests (which we already have): `--no-test-framework`

## Tags

1. We need to change the permitted params in our post controller to accept another attribute: `:tags_attributes`, which permit the tag attributes that we need to create a new tag.
2. `accepts_nested_attributes_for` on Post model, which will permit tags to be nested in our new post form.
3. Now we can build a nested form on our Post form. Check out the documentation on [Nested Forms](http://guides.rubyonrails.org/form_helpers.html#nested-forms) for help.
4. We should be able to select previously created tags, as well as create a new tag.
5. Remember, because we have a uniqueness validation on the name of tag, we will need to account for that; a user shouldn't have to submit a new tag every time they submit a post.
  
  ```ruby
  class Member < ActiveRecord::Base
    has_many :posts
    accepts_nested_attributes_for :posts, reject_if: proc { |attributes| attributes['title'].blank? }
  end
  ```
6. To allow a user to create a new tag, the controller action for a new tag should instantiate a new tag. Check out the documentation for the [fields_for tag](http://apidock.com/rails/ActionView/Helpers/FormBuilder/fields_for).

## Resources
[Strong Params](http://edgeguides.rubyonrails.org/action_controller_overview.html#strong-parameters)

[Nested Forms](http://guides.rubyonrails.org/form_helpers.html#nested-forms)

[fields_for tag](http://apidock.com/rails/ActionView/Helpers/FormBuilder/fields_for)

[Preventing Empty Records](http://guides.rubyonrails.org/form_helpers.html#preventing-empty-records)

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/rails-blog-nested-forms' title='Rails Blog: Complex Nested Forms'>Rails Blog: Complex Nested Forms</a> on Learn.co and start learning to code for free.</p>
