---
layout: post
title: Bootstrapping Chef in one command with Babushka
---

**TL;DR:** Install chef-server automatically with one command.

{{ page.title }}
=================================================

For those who don't know; Chef is:
  *"A systems integration framework, built to bring the benefits of configuration management to your entire infrastructure"*
  
But what is Babushka?
---------------------

[Babushka](http://babushka.me) is a test-driven system administration tool written by [Ben Hoskings](http://github.com/benhoskings), it uses an omnipotent approach to configure your servers by running deps written in a familiar Ruby DSL.

Why would you do this?
----------------------

If you have ever had to install chef-server by hand you will know it requires some work, and has the potential to break very easily. 
If you have trouble installing one version you have to ditch your instance and start from scratch, or spend considerable time reversing changes.

To solve this problem I wrote a collection of Babushka deps to do this job for us, and it even allows you to specify a version on the fly. 

It assumes you have a bare Ubuntu instance (Tested on 10.04) running somewhere with root access over ssh. While there is no real requirement for it to be bare, these deps will customise things considerably so it is advisable
to use a dedicated instance for this setup.

What does it do?
----------------

You might think installing chef-server is a simple bootstrap command documented on the [Chef Wiki](http://wiki.opscode.com/display/chef/Bootstrap+Chef+RubyGems+Installation). 
This is partly true, until you spend considerable time setting hostnames, creating users, uploading public keys, disabling root, installing default software and extensions, and not to mention installing Ruby and Rubygems 
from source.

So these deps take care of all of this. Here's a run list:

`babushka ivanvanderbyl:'chef user'`

* Sets a FQHN of your choosing
* Installs core software
  - vim
  - curl
  - htop
  - jnettop
  - screen
  - nmap
  - tree
  