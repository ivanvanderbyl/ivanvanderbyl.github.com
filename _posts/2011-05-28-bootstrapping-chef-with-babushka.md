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

1. 1024MB+ Ubuntu instance. We're using an [OrionVM Cloud](http://orionvm.com.au) instance running 10.04 Lucid Lynx, but most Amazon EC2 and Rackspace Ubuntu images will work fine after enabling `root` (Don't worry we will disable it again later)
2. Your ssh public key
3. About 10 minutes spare time.

Once you have created an instance, login as root using the console and ensure you have a password set, if not run: `passwd`

Now open a Terminal window and connect to this instance over SSH as root.

Install `wget` if not already installed:

    apt-get install wget
    
Bootstrap babushka using babushka:

    bash -c "`wget -O - babushka.me/up`"

Follow the default prompts and you should be good to go.

Next we run my `'chef user'` dep:

    babushka ivanvanderbyl:'chef user'
  
Notice how we don't have to tell Babushka where to get it from? By default Babushka looks for a Github repository called `<GITHUB USERNAME>/babushka-deps.git` and clones it, then loads it into its internal list.
[My babushka-deps repo](http://github.com/ivanvanderbyl/babushka-deps) contains the dep `chef user`

This will run the list documented below, and towards the end create a new user account. Make sure you don't call the user `chef` because it will cause things to fail in the next step. I usually use `deploy`

Now logout from root and reconnect as the chef user you just created to complete the next step: actually installing chef-server.

Install Chef
---------------------

This is where the magic happens, run this dep and you will have a complete Chef Server installation, optionally with `chef-server-webui`

    babushka ivanvanderbyl:'bootstrapped chef'

This will verify that all services have started correctly and if so report success, otherwise it will output a log file so you can trace what happened and hopefully fix the problem. (Please report bugs with this process below)

If all components are installed correctly it will register this `node` with `chef-server` by running `knife configure -i`

What did it really do?
----------------------

You might think installing chef-server is a simple bootstrap command documented on the [Chef Wiki](http://wiki.opscode.com/display/chef/Bootstrap+Chef+RubyGems+Installation). 
This is partially true, until you spend considerable time setting hostnames, creating users, uploading public keys, disabling root, installing default software and extensions, and not to mention installing Ruby and Rubygems 
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

Next Steps
----------

You could complete the chef setup as per the [Rubygems Chef Bootstrap guide](http://wiki.opscode.com/display/chef/Bootstrap+Chef+RubyGems+Installation) or enlist the pro-skills of Babushka again, in which case; read on.

*From the Chef Wiki:*
When working with chef, you will spend a lot of time editing recipes and other files, and you'll find it much more convenient to edit them on your laptop/desktop, where you have your editor configured just to your liking. 
To facilitate this mode of working, we recommend you create a knife client to use knife on your development machine.

I've also created a dep for this, simply run (on the server):

    babushka ivanvanderbyl:'external admin client.registered'

This will register a new client for you to use locally, which means you need to copy this new client private key to your laptop/desktop. 
The command for this is: (on your laptop/desktop)

    mkdir ~/.chef
    scp deploy@chef.example.com:/tmp/my-username.pem ~/.chef/my-username.pem
    
Replacing `chef.example.com` with the FQHN of your chef server, and replacing `my-username` with the username you entered when running `external admin client.registered`

*Note:* If you haven't installed the chef gem locally please do so now `gem install chef`

Configure knife locally:

    your-laptop > knife configure
    No knife configuration file found
    
Knife looks for its configuration in ~/.chef/knife.rb by default:

    Where should I put the config file? [~/.chef/knife.rb] 
    Please enter the chef server URL: [http://localhost:4000] http://chef.example.com:4000

Now, enter your client name, exactly as you did when running knife client create above:

    Please enter an existing username or clientname for the API: [my-username] my-username

For these next settings, you can accept the defaults for now and update them later by editing your knife.rb file. The validation client name and key are used with knife's cloud computing commands:

    Please enter the validation clientname: [chef-validator] 
    Please enter the location of the validation key: [/etc/chef/validation.pem]

Verify your local installation
------------------------------

    your-laptop > knife client list
    [
      "chef-webui",
      "my-username",
      "bob",
      "chef-validator"
    ]


Bootstrapping a new server client
=================================

You will need to bootstrap each new server you add to your cluster, this is similar to the process above however it does not depend on running as root, however if you are starting with the same image
it is recommended that you run the `chef user` dep first and login as that user before bootstrapping as a client. This is a more secure approach.

To bootstrap this server as a client, simply run:

    babushka ivanvanderbyl:'bootstrap chef client'

Alternatively you can do all these steps manually using the [Chef Wiki guide on Bootstrapping a new client with Rubygems](http://wiki.opscode.com/display/chef/Walk-through+-+Bringing+up+a+new+chef+client+%28using+rubygems%29)

Connect to server
-----------------
Now we need to set up the client so it is authenticated to talk to the server. This is done by creating a certificate pair.

To do this, you will need to copy the file `/etc/chef/validation.pem` from the chef server to your chef client into the same path. This might seem a little tricky given that we have disabled password login, so I would recommend a simple 
⌘C +  ⌘V approach.

Then, simply run `sudo chef-client` and it will register itself with your chef server. After it is done, **be sure to delete the file /etc/chef/validation.pem on your client!** (It is the key to the universe)


Contributing
========================

I encourage you to contribute to these deps to fix issues you come across or add features you want to use.

* Check out the latest [master](https://github.com/ivanvanderbyl/babushka-deps) to make sure the feature hasn't been implemented or the bug hasn't been fixed yet
* Check out the issue tracker to make sure someone already hasn't requested it and/or contributed it
* Fork the project
* Start a feature/bugfix branch e.g. git checkout -b `feature/my-awesome-idea` or `support/this-does-not-work`
* Commit and push until you are happy with your contribution
* Issue a pull request

