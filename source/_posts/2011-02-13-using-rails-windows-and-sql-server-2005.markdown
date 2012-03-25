---
layout: post
title: Using Rails, Windows and SQL Server 2005
tags:
  - Ruby
  - SQL Server
  - Windows
status: publish
type: post
published: true
meta:
jabber_published: '1297593335'
_edit_last: '1'
---

I have recently had to install a copy of our Rails app  on Windows. Using Ruby on Windows presents its own set of challenges and adding a gem that has a native extension makes it just that little more painful. Here is what I did to get SQL Server 2005 and Ruby to play nicely together on Windows.

###The Requirements

To use SQL Server with ActiveRecord we use ODBC.

Here are the things that are required to get this working correctly.

  * A Windows install of Ruby, obviously. I used the [7-zip](http://www.7-zip.org/)
    package available from [http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/)
    and  installed it to C:\ruby.
  * A set of DLLs available [here](/downloads/ruby_dep_dlls.zip)
    I have [documented the process](/2011/02/using-ruby-and-rails-on-windows/) for installing Rails on Windows.
  * The activerecord-sqlserver-adapter gem. I used version 2.3.13 because our app is Rails 2.
  * The activerecord-odbc-adapter.
  * The compiled version of the Ruby-ODBC adapter for Windows. Available at
    [http://www.ch-werner.de/rubyodbc/](http://www.ch-werner.de/rubyodbc/)
  * A compiled version of the win32-api gem. This apparently comes with most Windows installations of Ruby, but it wasn't in mine.

###The Process
After installing Ruby and ensuring that at least irb is working correctly with some basic smoke tests,
put the ODBC library and the other required DLLs in the bin directory of your Ruby installation.

Set up a system [DSN](http://en.wikipedia.org/wiki/Database_Source_Name)
that provides access to your SQL Server installation.

Configure database.yml with an ODBC connection to your system DSN.

``` yaml
development:
  adapter: sqlserver
  mode: odbc
  dsn: your_dsn
  username: database_user
  password: database_password
```

That's it. It's time to fire up script/console and try to connect to your database.

``` ruby
Loading development environment (Rails 2.3.4)
ActiveRecord::Base.connection
```

If everything has gone well, you will get an object printed to the console that represents your database connection.
