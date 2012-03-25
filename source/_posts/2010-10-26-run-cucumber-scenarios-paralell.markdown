---
layout: post
title: Run Cucumber scenarios in parallel
tags:
- Cucumber
- delayed_job
- Rails
- Ruby
- Testing
status: publish
type: post
published: true
meta:
switch_like_status: '1'
jabber_published: '1288094735'
---

I have recently started to write code that relies on delayed_job for background processing.
This means testing delayed_job with Cucumber. After a bit of research, it
seems the most common way to test your background task is completing is to
use Delayed::Job.work_off. This worked out fine, until I needed to test
something that required a delayed_job task to be running when I tested another
part of the system. Integrated, like, you know, how it would be in a production environment.

The other problem with using Delayed::Job.work_off in your Cucumber steps, is that it is run serially,
and you don't get to test for potential race conditions
that manage to hide in your code until 6pm the day before go live.

This raised the question:
{% blockquote %}
How do you test multiple cucumber steps in parallel?
{% endblockquote %}
The answer is: you don't. At least that is the answer that I kept coming across.

Inspired by this [post](http://corner.squareup.com/2010/08/cucumber-and-resque.html)
by the people at [Square](http://www.squareup.com/)
I decided to run our delayed in a separate process.

Here is what I ended up with:
``` ruby
class CucumberDelayedJob
  class << self
    attr_accessor :pid

    def start

      self.pid = fork
      if pid
        Process.detach pid
      else
        start_worker
      end
    end

    def stop_worker
      `script/delayed_job stop -- #{Rails.env}`
    end

    def start_worker
      exec('script/delayed_job', 'start', '--', Rails.env)
    end
  end
end
```

Note that the Rails.env variable is being passed into script/delayed_job.
This is important because delayed_job will use the environment given in that parameter
to select which database to connect to when checking for new work.

The script above does have its limitations:

  * In its current form you can only run instance of delayed_job.
    This includes jobs running with a different Rails environment, so all
    other instances of delayed_job will need to be stopped before this is used.
  * It assumes that it can start and stop the delayed_job instance successfully every time.

This is just a start, it is a very basic way to start and stop a delayed_job instance cleanly.
