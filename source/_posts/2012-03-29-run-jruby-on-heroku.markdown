---
layout: post
title: "Run JRuby on Heroku"
date: 2012-03-29 14:22
comments: true
categories: 
- jruby
- heroku
---

A lot of different hacks has existed for a while now to run [JRuby](http://jruby.org) on [Heroku](http://www.heroku.com) but thanks to a new Heroku feature called [buildpacks](https://devcenter.heroku.com/articles/buildpacks) is now easy to implement and run your own runtime on Heroku. 

I did therefore create a buildpack which basically downloads JRuby, install Bundler and run ```bundle install```. After that it behaves exactly like a normal Heroku Ruby application! 

``` sh Create a JRuby backed Heroku app 
$ heroku create -s cedar --buildpack https://github.com/jruby/heroku-buildpack-jruby.git 
```
Read more about the JRuby buildpack at [github.com/jruby/heroku-buildpack-jruby](https://github.com/jruby/heroku-buildpack-jruby)

Take a look at an example application here: [github.com/carlhoerberg/heroku-jruby-example](https://github.com/carlhoerberg/heroku-jruby-example)

#### Update 2012-04-22
The buildpack is moved and now maintained under the JRuby organization: [github.com/jruby/heroku-buildpack-jruby](https://github.com/jruby/heroku-buildpack-jruby)

If you already have created an application with the old buildpack url you can just update the environment variable to the new url: 

``` sh Change buildpack url
$ heroku config:add BUILDPACK_URL=https://github.com/jruby/heroku-buildpack-jruby
```
