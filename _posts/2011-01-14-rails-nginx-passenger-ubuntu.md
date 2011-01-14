My Rails, Nginx, REE, Passenger, Ubuntu stack
============================

My notes on setting up a simple production server with Ubuntu 10.04 LTS, NGINX, Passenger, Ruby Enterprise Edition 1.8.7 and Mysql for Rails 2.3.8/3.0

This guide assumes you have installed Ubuntu 10.04 LTS (with no modules (optional)) on a server somewhere and you have root access via sudo.

Aliases
-------

    echo "alias ll='ls -l'" >> ~/.bash_aliases
    
edit .bashrc and uncomment the loading of .bash_aliases

If you have trouble with PATH that changes when doing sudo, see [http://stackoverflow.com/questions/257616/sudo-changes-path-why](http://stackoverflow.com/questions/257616/sudo-changes-path-why) then add the following line to the same file

    echo "alias sudo='sudo env PATH=$PATH'" >> ~/.bash_aliases

Update and upgrade the system
-------------------------------

    sudo apt-get update
    sudo apt-get upgrade

Configure timezone (If you didn't already when installing Ubuntu)
-------------------

    sudo dpkg-reconfigure tzdata
    sudo apt-get install ntp
    sudo ntpdate ntp.ubuntu.com # Update time
    
Verify that you have to correct date and time with

    date

Configure hostname
-------------------

    sudo hostname your-hostname

Add 127.0.0.1 your-hostname

    sudo vim /etc/hosts
    
Write your-hostname in 
    
    sudo vim /etc/hostname
    
Verify that hostname is set
    
    hostname

Gemrc
-------

Add the following lines to ~/.gemrc, this will speed up gem installation and prevent rdoc and ri from being generated, this is not nessesary in the production environment.

    ---
    :sources:
    - http://gems.rubyforge.org
    - http://gems.github.com
    gem: --no-ri --no-rdoc


Install mysql
---------------

This should be installed before Ruby Enterprise Edition becouse that will install the mysql gem.

    sudo apt-get install mysql-server libmysqlclient15-dev    

Ruby Enterprise Edition
------------------------

Check for newer version at [http://www.rubyenterpriseedition.com/download.html](http://www.rubyenterpriseedition.com/download.html)

Install package required by ruby enterprise, C compiler, Zlib development headers, OpenSSL development headers, GNU Readline development headers

    sudo apt-get install build-essential zlib1g-dev libssl-dev libreadline5-dev

Download and install Ruby Enterprise Edition

    wget http://rubyforge.org/frs/download.php/71096/ruby-enterprise-1.8.7-2010.02.tar.gz
    tar xvfz ruby-enterprise-1.8.7-2010.02.tar.gz
    rm ruby-enterprise-1.8.7-2010.02.tar.gz
    cd ruby-enterprise-1.8.7-2010.02/
    sudo ./installer
    
    
Change target folder to /opt/ruby for easier upgrade later on

Add Ruby Enterprise bin to PATH

    echo "export PATH=/opt/ruby/bin:$PATH" >> ~/.profile && . ~/.profile
    
Verify the ruby installation

    ruby -v
    ruby 1.8.7 (2010-04-19 patchlevel 253) [x86_64-linux], MBARI 0x6770, Ruby Enterprise Edition 2010.02

Update RubyGems
---------------
    
    sudo gem update --system

Installing git
----------------

    sudo apt-get install git-core

NGINX
-------

Automatically install NGINX compiled with Passenger & SSL into /opt/nginx/

    sudo /opt/ruby/bin/passenger-install-nginx-module --auto --prefix=/opt/nginx/ --auto-download --extra-configure-flags=--with-http_ssl_module


NGINX init script
-------------------

More information on http://wiki.nginx.org/nginx-init-ubuntu

This command will download the latest version of my init script, copy it to /etc/init.d/nginx and update permissions.

    sudo curl http://github.com/ivanvanderbyl/rails-nginx-passenger-ubuntu/raw/master/nginx/nginx -o /etc/init.d/nginx && sudo chmod +x /etc/init.d/nginx && sudo chown root:root /etc/init.d/nginx
    
Verify that you can start and stop NGINX with init script

    sudo /etc/init.d/nginx start
    
      * Starting nginx Server...
      ...done.
    
    sudo /etc/init.d/nginx status
    
      NGINX found running with processes:  11511 11510
    
    sudo /etc/init.d/nginx stop
    
      * Stopping nginx Server...
      ...done.

Add it to the startup routine:

    sudo /usr/sbin/update-rc.d -f nginx defaults
    
If you want, reboot and see so the webserver is starting as it should.

Installning ImageMagick and RMagick (Optional)
-----------------------------------

If you want to install the latest version of ImageMagick. I used MiniMagick that shell-out to the mogrify command, worked really well for me.

    # If you already installed imagemagick from apt-get
    sudo apt-get remove imagemagick

    sudo apt-get install libperl-dev gcc libjpeg62-dev libbz2-dev libtiff4-dev libwmf-dev libz-dev libpng12-dev libx11-dev libxt-dev libxext-dev libxml2-dev libfreetype6-dev liblcms1-dev libexif-dev perl libjasper-dev libltdl3-dev graphviz gs-gpl pkg-config

Download the latest source:
    
    wget ftp://ftp.imagemagick.org/pub/ImageMagick/ImageMagick.tar.gz

Once the source is downloaded, uncompress it:


    tar xvfz ImageMagick.tar.gz


Now configure and make:

    cd ImageMagick-6.6.3-4
    ./configure
    make
    sudo make install

To avoid an error such as:

convert: error while loading shared libraries: libMagickCore.so.2: cannot open shared object file: No such file or directory

    sudo ldconfig

Install RMagick
 
    sudo /opt/ruby/bin/ruby /opt/ruby/bin/gem install rmagick

Install Bundler (Optional)
---------------

Bundler is arguably the best ruby gem manager ever written. Install it!

    sudo gem install bundler

Install Nokogiri (Optional)
----------------

Nokogiri dependencies

    sudo apt-get install libxslt-dev libxml2-dev

Install Nokogiri gem

    sudo gem install nokogiri
    
Capistrano environment fixes
----------------------------

If your deploying with Capistrano, you must modify a few things to get it to use Ruby Enterprise and load the local users environment.

Edit /etc/ssh/sshd_config and add the following to the bottom of the file:

    PermitUserEnvironment yes
    
Then reboot sshd by running:

    sudo /etc/init.d/ssh reload
    
Configuring the PATH

Edit ~/.ssh/environment, and put something like this inside:

    PATH=/opt/ruby/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
    
**OR if it doesn't exist you can do this to add it with one command:**

    echo "PATH=/opt/ruby/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games" > ~/.ssh/environment
    
Test a rails application with NGINX
----------------------------------

    rails -d mysql testapp
    cd testapp
    
Enter your mysql password
    
    vim database.yml
    rake db:create:all
    ruby script/generate scaffold post title:string body:text
    rake db:migrate RAILS_ENV=production
    
Check so the rails app start as normal
    
    ruby script/server

    sudo vim /opt/nginx/conf/nginx.conf
    
Add a new virtual host

    server {
        listen 80;
        # server_name www.mycook.com;
        root /home/deploy/testapp/public;
        passenger_enabled on;
    }
    
Restart NGINX

    sudo /etc/init.d/nginx restart
    
Check your ip address and see if you can access the rails application
        
