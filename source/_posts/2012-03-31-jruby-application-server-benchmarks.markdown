---
layout: post
title: "JRuby application server benchmarks"
date: 2012-03-31 19:32
comments: true
categories: [ jruby, heroku ]
---

Now that [JRuby is so easy to run on Heroku](http://carlhoerberg.github.com/blog/2012/03/29/run-jruby-on-heroku/) wanted I to see if there were any notable performance differences between the different [JRuby server alternatives](https://github.com/jruby/jruby/wiki/Servers) available.

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

Siege was run from a m1-small EC2 instance in the US-East-1 region, the same as Heroku. Siege was executed with the following parameters:

```$ siege -b -c 100 -t 60S http://jruby-puma.herokuapp.com/``` 

Which means that Siege made 100 concurrent connections continously during 60 seconds and measured the number of requests per second, among other things. 

### Configuration

* Puma - max 200 threads
* Trinidad - max 200 threads
* Mizuno - max 10 threads (more was not possible due to [this bug](https://github.com/matadon/mizuno/issues/22) I found)
* Thin - Default configuration
* Unicorn - 3 worker processes (which according to Michael van Rooijen is [the optimal number](http://michaelvanrooijen.com/articles/2011/06/01-more-concurrency-on-a-single-heroku-dyno-with-the-new-celadon-cedar-stack/))

## Results
The requests/second result can be seen in the following diagram:

![Transaction rate comparison](https://docs.google.com/spreadsheet/oimg?key=0AlUdaaoX8GlHdDRwczVDNVVxcWJpekVIUUhvcm9VWlE&oid=1&zx=zij8ut81d30z)

The full results can be evaluated in [this gist](https://gist.github.com/2262793/).

## Discussion
The application under test was a very rudimentary Rails application, it tested the basic Rails stack, included ERB rendering, but no I/O operations. This has a hugh impact in real application performance so as always, benchmark against your own application too!

A small EC2 instance only got moderate I/O performance, this might have an impact on the results and yield varying metrics. With a simple Sinatra application all the servers performed at about 1000 transactions/second, so that might be the upper limit the instance (or Heroku) is limited to. 

I only tested a small number of server, but most of the other JRuby application servers are deprecated or not maintained, for a more complete list see the [JRuby wiki entry on servers](https://github.com/jruby/jruby/wiki/Servers).

I didn't include [Torquebox](http://torquebox.org/) as it's much harder to get running at Heroku. I might try to make a Torquebox specific build pack, but Torquebox also comes with a lot of features (like a built in MQ and a key-value store) which is not usable on the Heroku platform.


## Summary
[Trinidad](https://github.com/trinidad/trinidad) has the best transaction rate among the JRuby web server alternatives at the moment. It also got more features than the others thanks to its [extension system](https://github.com/trinidad/trinidad/wiki/Available-extensions).

Can't really recommend [Mizuno](https://github.com/matadon/mizuno) at the moment due to the ["crash on high load" bug](https://github.com/matadon/mizuno/issues/22) I found.

[Puma](http://puma.io) is an interresting new alternative, a thread-based Ruby application server primarly written for Rubinius, it's seems well maintained and very light. 

But most notable is that Unicorn is still the best performer in terms of pure requests per second, this was a surprise as in theory a single threaded process-based model should be the slowest option. Also notable that Thin performed worst, although a more I/O bound application might have shown totally different results.

