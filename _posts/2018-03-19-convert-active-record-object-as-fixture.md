---
layout: post
title:  "Convert ActiveRecord object into fixture record"
---

Sometimes in your development environment you might have created a valid active
record object which you want to use it as a fixture in your test so
you don't need to waste your time from creating fixture manually.

To do this, you just need to open your Rails console and fetch that object:

```ruby
task = Task.last

# =>
# #<Task:0x00007fbf11919f60> {
#                :id => 28,
#           :subject => "Arrange Photographs",
#    :assigned_to_id => 3,
#       :due_date_at => Sat, 17 Mar 2018 01:00:20 EDT -04:00,
#         :completed => true,
#     :taskable_type => "Lead",
#       :taskable_id => 5,
#           :user_id => 3,
#        :created_at => Wed, 14 Mar 2018 01:00:20 EDT -04:00,
#        :updated_at => Mon, 19 Mar 2018 01:00:20 EDT -04:00,
#      :completed_at => Sun, 18 Mar 2018 01:00:20 EDT -04:00,
#   :type_associated => nil,
#   :completed_by_id => 3,
#    :is_next_action => nil
# }
```

Then convert that object into fixture record list like this,

```ruby
puts task.attributes.to_yaml

# =>
# id: 28
# subject: Arrange Photographs
# assigned_to_id: 3
# due_date_at: !ruby/object:ActiveSupport::TimeWithZone
#   utc: 2018-03-17 05:00:20.233297000 Z
#   zone: &1 !ruby/object:ActiveSupport::TimeZone
#     name: America/New_York
#   time: 2018-03-17 01:00:20.233297000 Z
# completed: true
# taskable_type: Lead
# taskable_id: 5
# user_id: 3
# created_at: !ruby/object:ActiveSupport::TimeWithZone
#   utc: 2018-03-14 05:00:20.233605000 Z
#   zone: *1
#   time: 2018-03-14 01:00:20.233605000 Z
# updated_at: !ruby/object:ActiveSupport::TimeWithZone
#   utc: 2018-03-19 05:00:20.245432000 Z
#   zone: *1
#   time: 2018-03-19 01:00:20.245432000 Z
# completed_at: !ruby/object:ActiveSupport::TimeWithZone
#   utc: 2018-03-18 05:00:20.233475000 Z
#   zone: *1
#   time: 2018-03-18 01:00:20.233475000 Z
# type_associated:
# completed_by_id: 3
# is_next_action: false
# nil

```

Then copy these values, paste in your fixture file, and give a proper name to
the fixture. Handle primary keys, foreign keys, date columns and remove unnecessary records as
per your requirement.

Also, if you want to select specific attributes from the object, then use it like
this,

```ruby
puts task.attributes.slice("subject", "completed").to_yaml

# =>
# subject: Arrange Photographs
# completed: true
# nil
```

I find this helpful when we create big active record objects from parsing our
user email data in our database and creating fixture for it manually is
cumbersome.

