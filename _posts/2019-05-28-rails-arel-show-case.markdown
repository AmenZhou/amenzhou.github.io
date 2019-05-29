---
layot: post
title:  "A RoR Arel Show Case"
date:   2019-05-2 13:00:00 -0500
categories:
tags: [ruby]
comments: true
---

# What is Arel?

Arel is a Ruby version SQL decorator, it is the real functions behind the Active Record since Rails 3.0.
e.g.

If you call `User.all`, it will be translated to `SELECT * FROM users`. This is what Arel does.

# What is the difference between Arel and ActiveRecord?

Arel is the guy doing the SQL translation job, ActiveRecord uses Arel to finish the tough job and then execute the translated query and instantiates the returned data by proper ActiveRecord models.

# Why do we need to care Arel?

If ActiveRecord actually uses Arel, why do we need to write Arel directly? ActiveRecord has some limits on composing SQL queries, especially for some old version Rails.

For example, "outer join" is not directly supported by ActiveRecord. You have to write something like `User.joins('left join posts on posts.user_id = users.user_id')`. It works but won't be bettter to just write Ruby code?

Some other cases like comparison time stamps and using subqueries .etc.

# A simple example

We want to get all users who have new posts within 24 hours

```ruby
user = User.arel_table
post = Post.arel_table

query = user
  .join(post)
  .on(
    post[:user_id].eq(user[:user_id])
  )
  .where(
    post[:created_at].gt(Time.current - 24.hours)
  )
```

This is only half done. It does join and where clause.

```ruby
query.project(Arel.sql('users.*')
query.distinct
```

The above code does `SELECT DISTINCT *`. You may have noticed that `project` and `distinct` do not chain after query. You can choose chain or not chain, no difference here.

Please be noticed, `distinct` returns `Arel::Nodes::Distinct` in Rails 3, it can not be chained with other `SelectManager` methods.

I got an issue on `query.project(Arel.star)`, it was fixed by using `Arel.sql('users.*')`

```ruby
User.scoped.find_by_sql(query.to_sql)
```

Fill instantiated Users by returned data. Don't forget to chain `scoped` after AR model.

# An advanced example

We want to get users with a flag - new_post, when a user has updated posts recently, set the flag to 1. Front-end apps can display an icon next to the users who have new posts.

```ruby
user = User.arel_table
post = Post.arel_table

sub_query = user
  .join(post, Arel::Nodes::OuterJoin)
  .on(
    post[:user_id].eq(user[:user_id])
  )
  .where(
    post[:created_at].gt(Time.current - 24.hours)
  )
sub_query.project(user[:user_id], '1 as new_post')
sub_query.as('user2')

query = user
  .join(sub_query)
  .on(
    user[:user_id].eq(user2[:user_id])
  )
query.project(Arel.sql('users.*'), users[:new_post])
query.distinct

users_with_flags = User.scoped.find_by_sql(query.to_sql)
```

```ruby
users_with_flags.first.new_post #=> 1
users_with_flags.second.new_post #=> nil
```

