---
layout: default
title:  "Secure logging in Rails"
date:   2019-04-30 12:49:57 +0530
category: rails
tag: rails-security
author: shubham
excerpt: 'Facebook is probing a series of security failures in which employees built applications that logged unencrypted password data for Facebook users and stored it in plain text on internal company servers. Very similar incidents have struck Twitter, Github and many others in the past.'
---
{% assign author = site.data.authors[page.author] %}

# {{ page.title }}
<span style="margin-left: 10px; font-size: 15px;">
  Written by <a href="{{ author.web }}" target="_blank">{{ author.name }}</a>
  in {{ page.tag }}
  on <time datetime="{{ page.date | date: "%Y-%m-%d" }}">{{ page.date | date_to_long_string }}</time>
</span>

Facebook is <a href="https://krebsonsecurity.com/2019/03/facebook-stored-hundreds-of-millions-of-user-passwords-in-plain-text-for-years/">probing</a> a series of security failures in which employees built applications that logged unencrypted password data for Facebook users and stored it in plain text on internal company servers. Very similar incidents have struck <a href="https://www.theverge.com/2018/5/3/17316684/twitter-password-bug-security-flaw-exposed-change-now">Twitter</a>, <a href="https://www.theinquirer.net/inquirer/news/3031566/github-bug-exposed-user-passwords-in-plaintext">Github</a> and <a href="https://plaintextoffenders.com/">many others</a> in the past.

Debug logs are an important tool for engineering teams to troubleshoot issues when other direct methods, like exception monitors, APMs etc fails. For those hard-to-replicate bugs, it is often a natural course of path for resolution.

Generally, people tend to log everything that is available, right from complete requests, responses, timestamps, user clicks and what not. However, often the logs are not as heavily secured as the main application data. Everyone in the team has access to them leaving the sensitive information open to get exposed inadvertently or otherwise. There have been instances when information such as passwords, account details and even credit card number have been exposed in similar fashion. Since the logs are in plain-text, anyone with access to the files can see the data.

Some of the stuff that should never be logged includes:

1. User Name
2. Email Addresses
3. Passwords / PINs / OTPs / Authentication tokens
4. Credit card details
5. IP Addresses
6. Social Security Number
7. Birth date
8. Address
9. Any personally identifiable information

## How to do this in Ruby on Rails

Rails supports filtering of parameters. Any parameter in this list will get masked while logging. See file `config/initializers/filter_parameter_logging.rb`. In below example, we are filtering `password` and `otp`. What this means is that when ever a user sends a request to the server with parameters matching values in this list, its values will get masked in the logs.

{% highlight ruby %}
  Rails.application.config.filter_parameters += [:password, :otp]
{% endhighlight %}

By default Rails only filters `password`. The logs will now never store any request parameter submitted by user with these names. For Example:

{% highlight ruby %}
  {
    user: { age: '27', country: 'India', password: [FILTERED], OTP: [FILTERED] }
  }
{% endhighlight %}

Stay safe. For the night is long and full of terrors!
