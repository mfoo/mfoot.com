---
layout: post
title: "Setting up a personal wiki with AWS and Gollum"
date: 2013-05-19 09:52
comments: true
categories: 
---

This post explains how to get a personal wiki using [Gollum](https://github.com/gollum/gollum)
running on Amazon Web Services (EC2) on a micro instance with the free usage
tier.

For a long time I've been using [TiddlyWiki](http://tiddlywiki.com/) as a
personal wiki for note taking. I used DropBox to sync the wiki html file between
machines. I have wanted to upgrade this for a while for a system that has
proper versioning and better syntax highlighting. I've also wanted to test out
using AWS as I think it will be a useful experience for the future.

# Step 1 - Get a domain name.
Any domain name will do. I recently learned about [Internationalised Domain
Names](http://en.wikipedia.org/wiki/Internationalized_domain_name), which are
URLs that contain language-specific Unicode characters. Browsers interpret the
Unicode characters in a system called
[Punycode](http://en.wikipedia.org/wiki/Punycode), which encodes the Unicode
characters into a restricted ASCII character set, allowing DNS entries for IDN
sites in the ASCII character set.

For instance, hover your mouse over the following URL in a modern browser and
you will see a domain beginning with `xn--`: <a href="http://☁→❄→☃→☀→☺→☂→☹→✝.ws">☁→❄→☃→☀→☺→☂→☹→✝.ws</a>

I had a browse through [a Unicode character table](http://unicode-table.com/en/),
found a glyph that I liked (it looks like a skull with a monocle),
then found the punycode URL for the character, then registered it with
[NameCheap](http://www.namecheap.com). Despite the name, the customer support
was very good.

I now had the URL http://ௐ.com.

# Step 2 - Set up an Amazon Micro Instance
I signed up for the free usage tier of AWS, started up a micro instance,
assigned an elastic IP to the instance, then pointed the DNS A record to that
IP using NameCheap's DNS server.

I used all of the default settings, and the micro instance image is the Ubuntu
13.04 server image. The next step was to install all required dependencies and
Gollum itself:

{% highlight sh %}
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install vim build-essential ruby ruby-dev libxslt1-dev git-core libxml2-dev
sudo gem install redcarpet gollum
{% endhighlight %}

The Gollum wiki has a list of other gems that you might find useful for
different types of language markup and syntax highlighting. After the
dependencies had been installed, I opened up TCP port 4567 on the web server
security group, and created the wiki itself:

{% highlight sh %}
mkdir wiki
cd wiki
git init .
{% endhighlight %}

I added a file at `config.ru` to enable HTTP Basic Auth on the Rack application,
based on the file I found [here](http://res0nat0r.github.io/blog/2012/07/23/add-authentication-to-gollum/).

{% highlight ruby %}
require 'rubygems'
require 'gollum/app'

use Rack::Auth::Basic, "Restricted Area" do |username, password|
   [username, password] == ['user', 'password']
end

gollum_path = File.expand_path(File.dirname(__FILE__))
Precious::App.set(:gollum_path, gollum_path)
Precious::App.set(:wiki_options, {})
run Precious::App
{% endhighlight %}

I could then see the server running with HTTP basic auth (see more authentication
options for Rack::Auth [here](https://github.com/roman/rack-auth)) on port 4567,
after running the following:

{% highlight bash %}
rackup -p 4567 config.ru
{% endhighlight %}

Hitting <Control+Z>, then running the following will let you log out of the
session and the server will continue to run.

{% highlight bash %}
bg
disown
{% endhighlight %}

You can install an Upstart script to have Upstart manage your Gollum process.
A quick Google found one that looks like it will do the trick
[here](https://gist.github.com/leon/2643936).

Note: This git repository has no remote, so this is your *only copy*. I suggest
using an EBS volume to store your git repo, or for instance a nightly Cron job
that pushes to another server.

