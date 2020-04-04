---
layout: post
title: "Primary or default flag to the Rails models"
tags:
- rails
---

**Update:** Please check updated changes <a href="https://github.com/mechanicles/set_as_primary/blob/master/CHANGELOG.md" target="_blank">here</a>.

In some client projects, I found that there was one common requirement: setting a
primary or default flag for models like EmailAddress, PhoneNumber, Address, etc.
which belong to User/Person model or polymorphic model. 

I found that in those projects, I had added almost the same backend code
which worked pretty good. I wanted to create a gem for this but didn't work on
it. Just recently I added the same code in another project, and then I decided
to create a gem for it so other developers can use it if they have
similar requirements.

Yesterday on Christmas :christmas_tree: day, I had released that [set_as_primary](
https://github.com/mechanicles/set_as_primary) gem.

Let's see how to use this gem in the Rails application.

## Backend:

### Setup:

Add this line to your application's Gemfile:

```
gem 'set_as_primary'
```

And then execute:

```
bundle
```

### Migration:

If your table does not have the primary flag column, then you can add it by
running following command in your rails project:


```shell
rails generate set_as_primary your_table_name
```

If you run above command for **email_addresses** table, then it creates migration like this:

```ruby
class AddPrimaryColumnToEmailAddresses < ActiveRecord::Migration[6.0]
  def change
    add_column :email_addresses, :primary, :boolean, default: false, null: false
  end
end
```

Don't forget to run `rails db:migrate` to create an actual column in the table.

### Usage:

```ruby
class User < ApplicationRecord
  has_many :email_addresses
  has_many :addresses, as: :owner
end

class EmailAddress < ApplicationRecord
  include SetAsPrimary
  belongs_to :user

  set_as_primary :primary, owner_key: :user
 end

class Address < ApplicationRecord
  include SetAsPrimary
  belongs_to :owner, polymorphic: true

  set_as_primary :primary, owner_key: :owner
end
```

You need to include `SetAsPrimary` module in your model where you want to handle
the primary flag. Then to `set_as_primary` class helper method, pass your primary
flag attribute with required association key `owner_key`.

**NOTE:** Default primary flag attribute is `primary`, and you can use another
one too like `default` but make sure that flag should be present in the table 
and should be a boolean column type.

Please make sure once again that you have added these settings correctly. Once you
do that, your application should work properly for handling primary flag.

#### force_primary option:

```ruby
class Address < ApplicationRecord
  include SetAsPrimary
  belongs_to :user

  set_as_primary :default, owner_key: :user, force_primary: false
 end
```

By default `force_primary` option is set to `true`. If this option is true, then it
automatically sets record as primary when there is only one record in the table.
If you don't want this flow, then set it as `false`.

## Frontend:

This gem only supports backend code. You need to handle your frontend part based
on your requirement. Based on my knowledge, there are two ways to handle the 
frontend part.

1. Using Non-JavaScript form: That means, you might have a parent form for a user
object and it accepts nested attributes for email addresses/phone numbers, or you 
might have separate email addresses/phone numbers form for a user. This can be done
by composing your form like this
[blog post](https://blog.yechiel.me/rails-radio-tags-in-nested-forms-4f252ae8cf53).
Based on that blog, I have created similar Rails application ([see it in
action](https://cryptic-lake-90495.herokuapp.com/)) for this gem. You can find
its complete code [here](https://github.com/mechanicles/set_as_primary_rails_app).

2. Using JavaScript (or using Ajax): You might have a single long-form with
nested attributes with one `Submit` button. Or you can have an implicit edit
functionality for nested attributes for email addresses/phone numbers. For more
information on these, please take a look at this [blog
post](https://www.pluralsight.com/guides/ruby-on-rails-nested-attributes) and
this [Cocoon](https://github.com/nathanvda/cocoon) gem.

Try out this gem and let me now your feedback. If you find any issues with this gem,
please raise an issue here [https://github.com/mechanicles/set_as_primary/issues](https://github.com/mechanicles/set_as_primary/issues).

Happy coding :)

