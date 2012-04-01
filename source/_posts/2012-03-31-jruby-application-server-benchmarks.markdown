---
layout: post
title: "JRuby application server benchmarks"
date: 2012-03-31 19:32
comments: true
publish: false
categories: [ jruby, heroku ]
---

Now that [JRuby is so easy to run on Heroku](http://carlhoerberg.github.com/blog/2012/03/29/run-jruby-on-heroku/) wanted I to see if there were any notable performance differences between the different server alternatives available.

### Entrants

The [JRuby](http://jruby.org) alternatives:

* [Trinidad](https://github.com/trinidad/trinidad) - A wrapper around [Tomcat](http://tomcat.apache.org/)
* [Mizuno](https://github.com/matadon/mizuno) - A wrapper around [Jetty](http://jetty.codehaus.org/jetty/)
* [Puma](http://puma.io) - A thread-based server written in Ruby 

I also included two CRuby alternatives for reference:

* [Thin](http://code.macournoyer.com/thin/) - Event-based Ruby web server
* [Unicorn](http://unicorn.bogomips.org/) - Process-based Ruby web server

## Benchmark approach
A small Rails application was deploy to [Heroku](http://www.heroku.com) and [Siege](http://www.joedog.org/siege-home/) was used to mesaure the transaction rate. 

The application was a one file Rails app:

``` ruby config.ru 
require "rails"
require "action_controller/railtie"

class MyApp < Rails::Application
  routes.append do
    match "/" => "hello#index"
  end
  config.threadsafe!
end

class HelloController < ActionController::Base
  def index
    @time = Time.now
  end
end

MyApp.initialize!

run MyApp
```

The rest of the code can be seen here: [github.com/carlhoerberg/heroku-jruby-example](https://github.com/carlhoerberg/heroku-jruby-example), each branch contains the specific configuration. 

I created a Heroku application for each branch/web server, with 1 dyno. [Siege](http://www.joedog.org/siege-home/) was then used to make the requests. The benchmark was performed from a small EC2 instance, with 100 simultaneous connections for 60 seconds.

Configuration:

* Puma - max 200 threads
* Trinidad - max 200 threads
* Mizuno - max 10 threads (more was not possible due to [this bug](https://github.com/matadon/mizuno/issues/22) I found)
* Thin - Default configuration
* Unicorn - 3 worker processes

## Results
The raw requests/second result can be seen in the following diagram:

![Transaction rate comparison](https://docs.google.com/spreadsheet/oimg?key=0AlUdaaoX8GlHdDRwczVDNVVxcWJpekVIUUhvcm9VWlE&oid=1&zx=zij8ut81d30z)

The full results can be evaluated in [this gist](https://gist.github.com/2262793/).

## Discussion
The application under test was a very rudimentary Rails application, a more complicated application, with more computation requirements might yield more intressenting results. 

A small EC2 instance only got moderate I/O performance, this might have an impact on the results and yield varying metrics. With a simple Sinatra application all the servers performed at about 1000 transactions/second, so that might be the upper limit the machine (or Heroku) is limited to. 

[Torquebox](http://torquebox.org/) is a notable application server not included in the comparison, the main reason is that Torquebox is so much harder to get running at Heroku. I might try to make a Torquebox specific build pack, but Torquebox also comes with a lot of features (like a built in MQ and key-value store) not usable on the Heroku platform.

## Summary
Trinidad has the best transaction rate among the JRuby web server alternatives at the moment. It also got more features than the other thanks to a extension system. [Trinidad Threaded Resque](https://github.com/carlhoerberg/trinidad_threaded_resque_extension) is just one example. 

Can't really recommend Mizuno at the moment due to the ["crash on high load" bug](https://github.com/matadon/mizuno/issues/22) I found.

But most notable is that Unicorn is still the best performer in terms of pure requests per second, this was a surprise as in theory a process-based model should be the slowest option. Also notable that Thin performed worst.

