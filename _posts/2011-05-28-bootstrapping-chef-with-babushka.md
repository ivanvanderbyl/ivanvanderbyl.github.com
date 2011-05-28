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

Getting started
===============

So lets get into it, you will need the following if following on at home:

1. 1024MB+ Ubuntu instance. We're using a [OrionVM Cloud](http://orionvm.com.au) instance
2. Your ssh public key
3. About 10 minutes spare time.

Once you have created an instance, login as root using the console and ensure you have a password set, if not run: `passwd`

Now open a Terminal window and connect to this instance over SSH as root.

Install wget if not already installed:

    apt-get install wget
    
Bootstrap babushka using babushka:

    bash -c "`wget -O - babushka.me/up`"
    
Follow the default prompts and you should be good to go.

Next we run my `'chef user'` dep:

    babushka ivanvanderbyl:'chef user'
    
Notice how we don't even have to tell Babushka where to get it from? By default Babushka looks for a Github repository called <username>/babushka-deps.git and clones it, then loads it into its internal list.
[My babushka-deps repo](http://github.com/ivanvanderbyl/babushka-deps) contains the dep `chef user`

This will run the list documented below, and towards the end create a new user account. Make sure you don't call the user `chef` because it will cause things to fail in the next step. I usually use `deploy`

Now logout from root, and reconnect as the chef user you just created to complete the next part.

What does it do?
----------------

You might think installing chef-server is a simple bootstrap command documented on the [Chef Wiki](http://wiki.opscode.com/display/chef/Bootstrap+Chef+RubyGems+Installation). 
This is partly true, until you spend considerable time setting hostnames, creating users, uploading public keys, disabling root, installing default software and extensions, and not to mention installing Ruby and Rubygems 
from source.

So these deps take care of all of this. Here's a run list:

`babushka ivanvanderbyl:'chef user'` - run this as `root`.

* Sets a FQHN of your choosing
* Installs core software
  - vim
  - curl
  - htop
  - jnettop
  - screen
  - nmap
  - tree
* Creates an `admin` group and adds it to sudoers list
* Creates a non-privileged user to install chef-server under and manage the system
  - Adds passwordless sudo
  - Installs your public key in `~/.ssh/authorized_keys`
* Disables password logins
* Disables `root` login

After this you should logout and login as your non-privileged user (`deploy` by default)

`babushka ivanvanderbyl:'bootstrapped chef'` - run this as your non-privileged user (`deploy` by default)

* Ensures we have a FQHN
* Installs Ruby 1.8.7
* Installs chef dependencies
  - irb
  - build-essential
  - wget
  - ssl-cert
* Compiles Rubygems 1.7.2 from source
  - Disables gem docs (--no-ri --no-rdoc)
* Installs `chef` gem with the version you specify
* Installs `ohai` gem
* Creates a chef-solo configuration
* Creates a chef bootstrap configuration
* Bootstraps Chef Server using Chef Solo

After this you will have a complete chef setup as per the [Rubygems Chef Bootstrap guide](http://wiki.opscode.com/display/chef/Bootstrap+Chef+RubyGems+Installation)