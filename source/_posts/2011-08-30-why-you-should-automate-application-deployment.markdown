---
layout: post
title: Why you should automate application deployment
tags:
  - Capistrano
  - Ruby
status: publish
type: post
published: true
meta:
_edit_last: '1'
---

If you are deploying your applications by hand, you are doing it wrong.

I learned this very important lesson the hard way when the application that I was working on
went from being used by one client, to 6. Each client has their own virtual server with a
test and production version of the software, as well as configuration files that are
different for each test and production instance. That is a total of 24 deployments. 24.

Each code change would involve the following:

  * Make code changes
  * ssh to client server
  * Get latest code from GitHub
  * cd to target directory
  * Run deployment script
  * Restart web component

Configuration changes involved a similar number of steps.

Each deployment took around 2 minutes to complete, assuming it was successful. Adding in the time for changing servers and correcting mistakes, that is around 30 minutes just to deploy a code change.

I was frustrated with the amount of time wasted executing the same steps over and over.
That is what computers are good at, so I spent a few hours learning and implementing
[Capistrano](https://github.com/capistrano/capistrano)
tasks. I can now deploy to multiple servers at once and each deployment is just a matter of running
``` ruby
cap deploy:update HOSTS=<client IP addresses>
```
from my local machine. I don't even have to login to the client server.

I have gone from a ~30 minute deployment to ~1 minute, saving my time and sanity in the process.
