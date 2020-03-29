---
layout: post
title: "Rails adds Enumerable#pick method"
---

Rails 6 [adds](https://github.com/rails/rails/pull/38760) a new method 
`pick` method in Enumerable module.

Let's see what benefits we can take from it.

Previously, if we wanted to extract the given keys from the first element, we 
were using `pluck` method on the array of hashes and applying `first` method on
it.

#### Example

```ruby
[{ id: 1, name: "David" }, { id: 2, name: "Rafael" }].pluck(:id, :name).first

# => [1, "David"]
```

Now by using this new `pick` method, we can avoid that extra method chaining.

```ruby
[{ id: 1, name: "David" }, { id: 2, name: "Rafael" }].pick(:id, :name)

# => [1, "David"]
```

`ActiveRecord::Relation#pick` method also got improved with `Enumerable#pick`
method, and now it does not fire another extra query for loaded results.

Here's the [pull request](https://github.com/rails/rails/pull/38760) for this
new method.
