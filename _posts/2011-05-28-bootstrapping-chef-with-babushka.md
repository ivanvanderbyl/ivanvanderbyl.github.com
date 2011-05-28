Bootstrapping Chef with Babushka with one command
=================================================

For those who don't know, Chef is:
  "A systems integration framework, built to bring the benefits of configuration management to your entire infrastructure"
  
But what is Babushka?
---------------------

[Babushka](http://babushka.me) is a test-driven system administration tool written by [Ben Hoskings](http://github.com/benhoskings), it uses an omnipotent approach to configure your servers by running deps written in a familiar Ruby DSL.

Why would you do this?
----------------------

If you have ever had to install chef-server by hand you will know it requires some work, and has the potential to break very easily. 
If you have trouble installing one version you have to ditch your instance and start from scratch, or spend considerable time reversing changes.

To solve this problem I wrote a collection of Babushka deps to do this job for us, and it even allows you to specify a version on the fly. 

It assumes you have a bare Ubuntu instance (Tested on 10.04) running somewhere with root access over ssh.

What does it do?
----------------

Sure, you might think installing chef-server is a simple bootstrap command documented on the [Chef Wiki](http://wiki.opscode.com/display/chef/Bootstrap+Chef+RubyGems+Installation). This is partly true,
until you spend considerable time setting hostnames, creating users, installing default software you might expect which is missing from this bare install of Ubuntu, and not to mention installing Ruby and Rubygems 
from source.

