## Table of Contents

* [Pagination](#pagination)
* [Recursive associations](#recursive-associations)
* [Exercises](https://github.com/hackerschoolmty/the-web-developer/blob/master/Semana%204.-%20Manejo%20de%20usuarios%20en%20una%20aplicacion%20web/01/exercises.md)

## Pagination

Let's suppose we have an e-commerce, and your catalog grew pretty quickly, you have at least a hundred products, each one belongs to a category, have many photos, comments and many more...

So... that's a lot of content to place in one screen, let's take a look at the amazon product listing page...

![amazon](https://jcdev.s3.amazonaws.com/amazon.png "Amazon product listing example")

Take a look at the bottom of the page.

Almost every single e-commerce as well as cms or any other information system uses pagination

##### How it works? the theory?

Ok, let's go to the console

```ruby
> Product.all.to_sql
# to_sql show us how the sql query is composed
> Product.where(['name LIKE ?', '%idea%']).to_sql
=> "SELECT \"products\".* FROM \"products\" WHERE (name LIKE '%idea%')"

# Counting results
> Product.count
=> 7
> Product.order(:id).limit(2).offset(0)
```

With the `limit` and the `offset` methods we can compose our pagination

```ruby
> page = 1
> per_page = 2
> Product.order(:id).limit(per_page).offset(per_page*page-per_page)
> total_pages = (Product.count / per_page) + (1 if Product.count % per_page)
```

Ok ok, but let's face it, doing our own pagination? really?

Luckily, nowadays we don't need to do it from scratch.

### Pagination gems

[Will Paginate](https://github.com/mislav/will_paginate)

[Kaminari](https://github.com/amatsuda/kaminari)

We're going to work with Kaminari

```ruby
# Gemfile
gem 'kaminari'
```

Then, of course...

```shell
$ bundle install
```

Here comes the magic...

```ruby
> Product.page(7).per(50)
```

```html
<!--You can use this in your views-->
<%= paginate @products %>
```



