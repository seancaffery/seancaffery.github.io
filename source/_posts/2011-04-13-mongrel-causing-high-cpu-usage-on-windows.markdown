---
layout: post
title: mongrel causing high CPU usage on Windows
tags:
  - Ruby
  - Windows
status: publish
type: post
published: true
meta:
_edit_last: '1'
---

The [mongrel_service gem](http://rubygems.org/gems/mongrel_service)
is a great thing to have if you are deploying a Ruby on Rails application in a Windows environment.
However, you should be aware that is does not handle problems encountered when trying to start the
underlying mongrel server.

I was recently called to a client site to resolve an issue with our
application causing high CPU usage. I checked the log directory and
found that the mongrel.log was over 100MB. This is quite out the ordinary for mongrel.

Running
``` bash
tail -n 50 mongrel.log
```
revealed that mongrel was encountering a database connection issue when trying to start Rails.
This caused a serious problem as there were 5  services constantly trying to start and it
brought the server to its  knees.

The solution was simple; I rectified the database connection issue and restarted the mongrel_service instances.

If you encounter a similar issue when running mongrel_service on Windows,
RAILS_ROOT/log/mongrel.log should point you to the cause of the problem in no time.
