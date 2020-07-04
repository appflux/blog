---
layout: post
comments: true
title:  "How to write Rails helpers - Best Practices"
date:   2020-07-04 16:21:00 +0530
category: Web
tags: [Rails 6, Ruby on Rails]
author: shubham
excerpt: Think of helpers as a mechanism to write chunks of views that are present everywhere in your application. For example - to show user avatar icon that appears everywhere on your site or to show formatted time across your app. This ensures a consistent UI that goes with the spirit of your application.
---

Hello everyone!

Rails relies heavily upon helpers to write interfaces of the application in a consistent way. However, a lot of time helpers are (ab)used causing errors and even bugs on the UI. This post is about the best practices we follow at Appflux and how they help us deliver a consistent user interface.

> **Pro Tip:**<BR/>
Think of helpers as a mechanism to write chunks of views that are present everywhere in your application. For example: to show user avatar icon that appears everywhere on your site or to show formatted time across your app.
This ensures a consistent UI that goes with the spirit of your application.

## What really are helpers?

Helpers are a way to define methods that are available to all view templates by default. What this means is that you can define a method `short_humanized_time` inside `DateTimeHelper` and use across all your views to display time in a consistent format in all your pages.

A few helper guidelines we follow at Appflux to keep things tight and tidy.

1. Avoid helpers in favour of decorators or presenters.
2. Never use instance variables inside helpers.
3. Never fire any query inside helpers.
4. Never write business logic inside helper.

# #1: Avoid helpers in favour of decorators or presenters.

There are very few legitimate cases where using a helper is justified. It is almost always better to go for decorators or presenters.


# #2: Never use controller instance variables inside a helper.

All data that a helper method needs must be supplied via params. This reduces coupling between helpers and other parts of your app.

{% highlight ruby %}
## Bad
module PostsHelper
  def formatted_title
    content_tag(:strong, @post.title).html_safe
  end
end

## Good: Do not use instance variables set in controller actions
module PostsHelper
  def formatted_title(post)
    content_tag(:strong, post.title).html_safe
  end
end

## Best: Pass simpler objects
module PostsHelper
  def formatted_title(title)
    content_tag(:strong, title).html_safe
  end
end
{% endhighlight %}


# #3: Never fire any queries inside a helper

Regard helpers as an extension to the view templates. The reason why you don't fire queries in your views should be equally applicable here as well. It is always recommended to load all your data in one go inside your controller in a service class object.

# #4: Never write business logic inside a helper

It is okay to write presentational logic inside an helper, but never write business logic in your helpers.

{% highlight ruby %}

## app/helpers/date_time_helper.rb
##Bad
module PostsHelper
  def top_posts
    Post.top_rated.map do |post| ## NOT OKAY
      content_tag(:option, post.title)
    end.join(', ').html_safe
  end
end

## Good
module DateTimeHelper
  def short_humanized_time(time)
    if time.today? ## THIS IS OKAY
      "#{ time.strftime('%I:%M %p') }"
    elsif time.year == Time.current.year
      "#{ time.strftime('%b %d, %I:%M %p') }"
    else
      "#{ time.strftime('%A, %B %d, %Y') }"
    end
  end
end
{% endhighlight %}


# Final words of wisdom (for today!)

With great power comes great responsibility, and Rails helpers are no exception. They are expressive tools but also come with drawbacks. We are generally more inclined towards decorators and restrict their usage to very specific use cases only. If you follow above tips, maintaining helpers in long term should be a little easier.

