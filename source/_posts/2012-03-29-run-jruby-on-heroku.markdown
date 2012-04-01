---
layout: post
title: "Run JRuby on Heroku"
date: 2012-03-29 14:22
comments: true
categories: 
- jruby
- heroku
---

A lot of different hacks has existed for a while now to run [JRuby](http://jruby.org) on [Heroku](http://www.heroku.com) but thanks a new Heroku feature called [build packs](https://devcenter.heroku.com/articles/buildpacks) is now easy to implement and run your own runtime on Heroku. 

I did therefore create a build pack which basically downloads JRuby, install Bundler and run ```bundle install```. After that it behaves exactly like a normal Heroku Ruby application! 

``` sh Create a JRuby backed Heroku app 
$ heroku create -s cedar --buildpack https://github.com/carlhoerberg/heroku-buildpack-jruby.git 
```
Read more about the JRuby build pack at [github.com/carlhoerberg/heroku-buildpack-jruby](https://github.com/carlhoerberg/heroku-buildpack-jruby)

Take a look at an example application here: [github.com/carlhoerberg/heroku-jruby-example](https://github.com/carlhoerberg/heroku-jruby-example)

