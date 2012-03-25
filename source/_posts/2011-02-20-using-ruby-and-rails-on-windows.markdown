---
layout: post
title: Using Ruby and Rails on Windows
tags:
  - Ruby
  - Windows
status: publish
type: post
published: true
meta:
_edit_last: '1'
---

Here are some quick notes that will hopefully help you get started with Ruby on Windows.

This post relates to the 7-zip Ruby version available from [http://rubyinstaller.org/](http://rubyinstaller.org/)
and lists the additional libraries that I required to get the Rails application that I work on to
run and pass all  of the tests.

### Libraries
I needed the following libraries in addition to those distributed in the default Ruby installation.

  * gdbm.dll – GNU database manager – [http://gnuwin32.sourceforge.net/packages/gdbm.htm](http://gnuwin32.sourceforge.net/packages/gdbm.htm)
  * iconv.dll – API used to convert between different character encodings –
    [http//www.gnu.org/software/libiconv/documentation/libiconv/iconv.1.html](http://www.gnu.org/software/libiconv/documentation/libiconv/iconv.1.html)
  * libeay32.dll – Open SSL for Windows –
    [http//gnuwin32.sourceforge.net/packages/openssl.htm](http://gnuwin32.sourceforge.net/packages/openssl.htm)
  * pdcurses.dll – PDCurses is an implementation of the curses library for X11 –
    [http//gnuwin32.sourceforge.net/packages/pdcurses.htm](http://gnuwin32.sourceforge.net/packages/pdcurses.htm)
  * readline.dll –
    [http//gnuwin32.sourceforge.net/packages/readline.htm](http://gnuwin32.sourceforge.net/packages/readline.htm)
  * ssleay32..dll – Open SSL for Windows –
    [http//gnuwin32.sourceforge.net/packages/openssl.htm](http://gnuwin32.sourceforge.net/packages/openssl.htm)
  * zlib.dll –
    [http://http//zlib.net/](http://zlib.net/)

I have packaged these DLLs into an archive available [here](/downloads/ruby_dep_dlls.zip)

Once you have unzipped the Ruby installation, you need to place these DLLs in the /bin directory.

I hope this relieves some the frustration that can occur when first installing Ruby on Windows.
