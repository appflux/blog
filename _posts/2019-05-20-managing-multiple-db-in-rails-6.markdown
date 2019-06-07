---
layout: post
comments: true
title:  "Managing Multiple Databases in Rails 6 - Part I"
date:   2019-05-20 14:42:00 +0530
category: Web
tags: [Rails 6, Multiple Databases]
author: shubham
excerpt: Rails 6 provides native support for multiple databases. In this post we'll discuss about how to setup your Rails 6 app with multiple databases and how it works. This is part I of this series, we'll discuss more implementation details in next parts.
---
This is part 1 of our Rails 6 series specifically aimed at multiple database support that comes with it. Pleae stay tuned for more posts in this series.

<h1>Overview</h1>

Rails has always been infamous for being slow. To the point that there once lived a website <a href="https://www.reddit.com/r/ruby/comments/71nlq/httprailscantscalecom/" target="_blank">railscanscale.com</a>. One of the major focus areas of Rails 6 was to be scalable by default and the core team introduced several interesting features to make it possible. In this series we'll discuss how Rails 6 supports multiple databases, be it for adding more read-only databases or for connecting to multiple databases at the model level.

Large applications often interact with multiple databases apart from the primary database. These databases can be a read-replica, or several entirely different databases that stores some of the data that the application needs to access. To do this, Rails did three things:

1. Extend the database.yml to support declaring multiple databases per environment.
2. An API for the AR models to define the database(s) it can connect to. This API is similar to the `establish_connection` but supersedes it. This is the `connects_to` API.
3. An API to specify the db to connect to while writing a query. This is the `connected_to` API.

We'll discuss all three of them in this post.

## database.yml

First, let's see how you can configure your `config/database.yml` with multiple databases per environment.

{% highlight yaml %}
  default: &default
  adapter: postgresql
  encoding: unicode
  pool: 50
  production:
    primary:
      <<: *default
      url: <%= ENV['PRIMARY_DATABASE_URL'] %>
    primary_replica:
      <<: *default
      replica: true
      url: <%= ENV['REPLICA_DATABASE_URL'] %>
    animals:
      <<: *default
      url: <%= ENV['ANIMALS_DATABASE_URL'] %>
{% endhighlight %}

Here, we have defined three databases which the application can connect to while running in production environment: primary, primary_replica and animals.

## connects_to
Once you have declared the databases you want to connect to in database.yml, you have to tell what all databases can an ActiveRecord model connect to. This is pretty neat. Lets look at a few code snippets to understand how it works.

{% highlight ruby %}
  class User < ApplicationRecord
    connects_to database: { writing: :primary, reading: :primary_replica }
  end
{% endhighlight %}

{% highlight ruby %}
  class Cat < ApplicationRecord
    connects_to database: { writing: animals, reading: :animals }
  end
{% endhighlight %}

Here we specify connection information for individual models. *User* model will connect to *primary* database for writing and *primary_replica* for reading while the *Cat* with use a separate *animals* database because there simply are too many cats out there. This has no default implications as all your queries (read or write) will go to the *writing* database by default. Rails doesn't automagically send read queries to the *reading* database. To route our read queries to `primary_replica` (*reading* database, we have to explicitly connect to it at the time of calling. This is done by wrapping the queries in a `connected_to` block.

## connected_to

{% highlight ruby %}
  ActiveRecord::Base.connected_to(role: :reading) do
    puts Cat.count
    puts User.count
  end
{% endhighlight %}

> Here, for counting Cats AR will use the `animals` DB and for counting Users, it will use the `primary_replica`.

Now, this is a real problem if you have to do this explicitly everywhere in your application. AR does provide a way to by default connect to the *reading* role if the request type is GET or HEAD. However, if you are doing some INSERTs or UPDATEs in GETs or HEADs, you'll have to be careful to put them in a `connected_to` block. This again is not very optimal. We'll discuss about these in our next post.

## A few things that it doesn't support yet

1. It can not be used to switch databases to handle failovers. For example, lets say, your primary database is down and can not serve any queries, Rails will not automatically try to switch to my *primary_replica*.

2. It will not automatically pick the *writing* role when AR is doing INSERTs and UPDATEs and *reading* role when AR is doing SELECTs. *writing* role is the default that AR uses.

3. It does not support JOINs across databases.

4. It does not monitor replication lag. So, you have to be proactive about your infrastructure and tell AR to use the replica if no writes happened in last n seconds.

## Credits

While there are definitely many gaps in current implementation but it surely is a huge step in the right direction. You can still use it to route your long running reports to a slow DB explicitly or things like that.

This work was majorly done by <a href="https://github.com/eileencodes">Eileen Uchitelle</a> and <a href="https://github.com/tenderlove">Aaron Patterson</a>. Below are the related pull requests. I referred them extensively to write this post.

1. <a href="https://github.com/rails/rails/pull/32274" target="_blank">pull/32274</a>
2. <a href="https://github.com/rails/rails/pull/33637" target="_blank">pull/33637</a>
3. <a href="https://github.com/rails/rails/pull/33770" target="_blank">pull/33770</a>
4. <a href="https://github.com/rails/rails/pull/34052" target="_blank">pull/34052</a>
5. <a href="https://github.com/rails/rails/pull/34491" target="_blank">pull/34491</a>
6. <a href="https://github.com/rails/rails/pull/34495" target="_blank">pull/34495</a>
7. <a href="https://github.com/rails/rails/pull/34505" target="_blank">pull/34505</a>
8. <a href="https://github.com/rails/rails/pull/35073" target="_blank">pull/35073</a>

Please share your thoughts and what you think about this new feature in Rails. Thanks!
