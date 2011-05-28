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

So lets get into it, you'll need the following if following on at home:

1. 1024MB+ Ubuntu instance. We're using an [OrionVM Cloud](http://orionvm.com.au) instance, but most Amazon EC2 and Rackspace Ubuntu images will work fine after enabling `root` (Don't worry we will disable it again later)
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
    
Notice how we don't have to tell Babushka where to get it from? By default Babushka looks for a Github repository called `\[GITHUB USERNAME\]/babushka-deps.git` and clones it, then loads it into its internal list.
[My babushka-deps repo](http://github.com/ivanvanderbyl/babushka-deps) contains the dep `chef user`

This will run the list documented below, and towards the end create a new user account. Make sure you don't call the user `chef` because it will cause things to fail in the next step. I usually use `deploy`

Now logout from root, and reconnect as the chef user you just created to complete the next step: actually installing chef-server.

Actually install Chef
---------------------

This is where the action happens, run this dep and you will have a complete Chef Server installation, optionally with `chef-server-webui`

    babushka ivanvanderbyl:'bootstrapped chef'
    
You should see the output like this if it worked:

    Updating git://github.com/ivanvanderbyl/babushka-deps.git... Already up-to-date at be78742, done.
    ivanvanderbyl:bootstrapped chef {
      ivanvanderbyl:bootstrap chef server with rubygems {
        ivanvanderbyl:hostname {
          hostname [chef.easylodge.com.au]? 
        } ✓ ivanvanderbyl:hostname
        ruby {
          'ruby' & 'irb' run from /usr/bin.
          ✓ ruby is 1.8.7, which is >= 1.8.6.
        } ✓ ruby
        ivanvanderbyl:chef install dependencies.managed {
          apt {
            main.apt_source {
            } ✓ main.apt_source
            universe.apt_source {
            } ✓ universe.apt_source
            'apt-get' runs from /usr/bin.
          } ✓ apt
          ✓ system has irb deb
          ✓ system has build-essential deb
          ✓ system has wget deb
          ✓ system has ssl-cert deb
          'wget', 'make', 'irb' & 'gcc' run from /usr/bin.
        } ✓ ivanvanderbyl:chef install dependencies.managed
        rubygems {
          ✓ ruby (cached)
          'gem' & 'ruby' run from /usr/bin.
          ✓ gem is 1.7.2, which is >= 1.7.2.
        } ✓ rubygems
        ivanvanderbyl:rubygems with no docs {
        } ✓ ivanvanderbyl:rubygems with no docs
        ivanvanderbyl:chef.gem {
          chef version [0.10.0]? 
          ✓ rubygems (cached)
          ✓ system has chef-0.10.0 gem
          'chef-client' runs from /usr/bin.
        } ✓ ivanvanderbyl:chef.gem
        ivanvanderbyl:ohai.gem {
          ✓ rubygems (cached)
          ✓ system has ohai-0.6.4 gem
          'ohai' runs from /usr/bin.
        } ✓ ivanvanderbyl:ohai.gem
        ivanvanderbyl:chef solo configuration {
        } ✓ ivanvanderbyl:chef solo configuration
        ivanvanderbyl:chef bootstrap configuration {
        } ✓ ivanvanderbyl:chef bootstrap configuration
        ivanvanderbyl:bootstrapped chef installed {
          The commands for 'bootstrapped chef installed' run from more than one place.
          'chef-client' runs from /usr/bin, but 'chef-server' & 'chef-solr' run from /usr/sbin.
          I don't know how to fix that, so it's up to you. :)
        } ✗ ivanvanderbyl:bootstrapped chef installed
      } ✗ ivanvanderbyl:bootstrap chef server with rubygems
    } ✗ ivanvanderbyl:bootstrapped chef
    You can view a more detailed log at '/home/deploy/.babushka/logs/ivanvanderbyl:bootstrapped chef'.

This will verify that all services have started correctly and if so report a success, otherwise it will output a log file so you can trace what happened and hopefully fix the problem.



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