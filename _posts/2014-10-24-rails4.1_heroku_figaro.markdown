---
layout: post
title:  "Rails 4.1, Heroku and Figaro"
date:   2014-10-24 00:12:30
categories: Rails 4.1
---

Yesterday, I spent some time scaffolding a new rails 4.1 application and was excited to see the inclusion of a Figaro-like setup for ENV variables.  I added several API keys to my secrets.yml file and then ignored the file in my .gitignore.  However, when I went to deploy my application to Heroku, it choked.  

First, I received the following error from devise, which I am using for authentication:

{% highlight console %}
Preparing app for Rails asset pipeline
   Running: rake assets:precompile
   rake aborted!
   Devise.secret_key was not set. Please add the following to your Devise initializer:
   config.secret_key = 'EXAMPLE_KEY_HERE'
   Please ensure you restarted your application after setting the key.
{% endhighlight %}

Adding the secret key to my devise.rb setup promptly resolved the issue, but I knew this wasn't the ideal setup, and the next deployment choked on the missing secret key base

{% highlight console %}
  Missing secret_key_base for 'production' environment, set value in config/secrets.yml
{% endhighlight %}

So, I decided to go back to the drawing board, and after unsuccessfully trying several configuration options including the heroku secrets gem, I returned to Figaro.  

First, I removed the key I had added to devise.rb.  Then, I installed Figaro and transferred my secret api keys and secret key bases for test and development from secrets.yml to application.yml.  If you use the CLI tool to install Figaro, it should add application.yml to your .gitignore, but it's a good idea to check.

Second, I removed secrets.yml from .gitignore, so that it would be checked into the repo.

Here is my secrets.yml:

{% highlight yaml %}

development:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>

test:
  secret_key_base: <%= ENV["SECRET_KEY_BASE_TEST"] %>

production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>

{% endhighlight %}

Here is my application.yml (substitute your keys as necessary)

{% highlight yaml %}

SECRET_KEY_BASE: {SUBSTITUTE YOUR KEY HERE}
SECRET_KEY_BASE_TEST: {SUBSTITUTE YOUR KEY HERE}

{% endhighlight %}

The secret key base for production will be generated automatically be heroku.

Then, when you want to add sensitive keys to your application, you can do that by adding them to application.yml and calling them as:

{% highlight ruby %}
  ENV['TOP_SECRET_KEY']
{% endhighlight %}

If you want to use the new Rails configuration for secrets, you can then add them to the secrets.yml file as such:

{% highlight yaml %}

development:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
  top_secret_key: <%= ENV["TOP_SECRET_KEY"] %>

test:
  secret_key_base: <%= ENV["SECRET_KEY_BASE_TEST"] %>
  top_secret_key: <%= ENV["TOP_SECRET_KEY"] %>

production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
  top_secret_key: <%= ENV["TOP_SECRET_KEY"] %>

{% endhighlight %}

Then, you can call 

{% highlight ruby %}
  Rails.application.secrets.top_secret_key
{% endhighlight %}

from anywhere within the app.  I prefer to just skip the last step and use 

{% highlight ruby %}
  ENV['TOP_SECRET_KEY']
{% endhighlight %}
