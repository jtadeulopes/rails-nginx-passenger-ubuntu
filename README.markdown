rails-nginx-passenger-ubuntu
============================

My notes on setting up a simple production server with ubuntu, nginx, passenger and mysql for rails.

Aliases
-------

    echo "alias ll='ls -l'" >> ~/.bash_aliases
    
edit .bashrc and uncomment the loading of .bash_aliases

If you have trouble with PATH that changes when doing sudo, see http://stackoverflow.com/questions/257616/sudo-changes-path-why then add the following line to the same file

    echo "alias sudo='sudo env PATH=$PATH'" >> ~/.bash_aliases
    

Update and upgrade the system
-------------------------------

    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    sudo apt-get autoremove
    sudo reboot

Configure timezone
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

Install mysql
---------------

This should be installed before Ruby Enterprise Edition becouse that will install the mysql gem.

    sudo apt-get install mysql-server libmysqlclient15-dev
    
    
Gemrc
-------

Add the following lines to ~/.gemrc, this will speed up gem installation and prevent rdoc and ri from being generated, this is not nessesary in the production environment.

    --- 
    :sources: 
    - http://rubygems.org/
    - http://gemcutter.org
    gem: --no-rdoc --no-ri


Ruby 1.9.2
----------

Check for newer version at [http://www.ruby-lang.org/en/downloads/](http://www.ruby-lang.org/en/downloads/)

Install package required by ruby enterprise, C compiler, Zlib development headers, OpenSSL development headers, GNU Readline development headers

    sudo apt-get install build-essential zlib1g-dev libssl-dev libreadline5-dev

Download and install Ruby

    wget ftp://ftp.ruby-lang.org//pub/ruby/1.9/ruby-1.9.2-p136.tar.gz
    tar xvfz ruby-1.9.2-p136.tar.gz 
    rm ruby-1.9.2-p136.tar.gz 
    cd ruby-1.9.2-p136.tar.gz/
    ./configure
    make
    sudo make install

For installing in /opt

    ./configure --prefix=/opt/ruby
    
Verify the ruby installation

    ruby -v
    ruby 1.9.2p136 (2010-12-25 revision 30365) [i686-linux]


Passenger 
---------

    sudo gem install passenger --no-rdoc --no-ri


Installing git
----------------

Check for newer version at [http://git-scm.com/download](http://git-scm.com/download)

Download and install git

    sudo apt-get build-dep git-core

    wget http://kernel.org/pub/software/scm/git/git-1.7.5.1.tar.bz2
    tar -jxvf git-1.7.5.1.tar.bz2
    cd git-1.7.5.1
    ./configure
    make
    sudo make install

Nginx
-------

    sudo passenger-install-nginx-module

Select option 1. Yes: download, compile and install Nginx for me. (recommended)

When finished, verify nginx source code is located under /tmp

    $ ll /tmp/
    drwxr-xr-x 8 deploy deploy    4096 2009-04-18 17:48 nginx-0.6.36
    -rw-r--r-- 1 root   root    528425 2009-04-02 08:49 nginx-0.6.36.tar.gz
    drwxrwxrwx 7   1169   1169    4096 2009-04-18 17:56 pcre-7.8
    -rw-r--r-- 1 root   root   1168513 2009-04-18 17:51 pcre-7.8.tar.gz
    
Run the passenger-install-nginx-module once more if you want to add --with-http_ssl_module 

    $ sudo /opt/ruby/bin/passenger-install-nginx-module
    
Select option 2. No: I want to customize my Nginx installation. (for advanced users)

When installation script ask, "Where is your Nginx source code located?" Enter:

    /tmp/nginx-0.6.36

On, extra arguments to pass to configure script add

     --with-http_ssl_module
     
     
Nginx init script
-------------------

More information on [http://wiki.nginx.org/Nginx-init-ubuntu](http://wiki.nginx.org/Nginx-init-ubuntu)

    cd
    git clone git://github.com/jnstq/rails-nginx-passenger-ubuntu.git
    sudo mv rails-nginx-passenger-ubuntu/nginx/nginx /etc/init.d/nginx
    sudo chown root:root /etc/init.d/nginx
    
Verify that you can start and stop nginx with init script

    sudo /etc/init.d/nginx start
    
      * Starting Nginx Server...
      ...done.
    
    sudo /etc/init.d/nginx status
    
      nginx found running with processes:  11511 11510
    
    sudo /etc/init.d/nginx stop
    
      * Stopping Nginx Server...
      ...done.
    
    sudo /usr/sbin/update-rc.d -f nginx defaults
    
If you want, reboot and see so the webserver is starting as it should.

Installning ImageMagick and RMagick
-----------------------------------

If you want to install the latest version of ImageMagick. I used MiniMagick that shell-out to the mogrify command, worked really well for me.

    # If you already installed imagemagick from apt-get
    sudo apt-get remove imagemagick

    sudo apt-get install libperl-dev gcc libjpeg62-dev libbz2-dev libtiff4-dev libwmf-dev libz-dev libpng12-dev libx11-dev libxt-dev libxext-dev libxml2-dev libfreetype6-dev liblcms1-dev libexif-dev perl libjasper-dev libltdl3-dev graphviz gs-gpl pkg-config

Use wget to grab the source from ImageMagick.org.

Once the source is downloaded, uncompress it:


    wget ftp://ftp.imagemagick.org/pub/ImageMagick/ImageMagick.tar.gz
    tar xvfz ImageMagick.tar.gz


Now configure and make:

    cd ImageMagick-6.6.2-10
    ./configure
    make
    sudo make install

To avoid an error such as:

convert: error while loading shared libraries: libMagickCore.so.2: cannot open shared object file: No such file or directory

    sudo ldconfig

Install RMagick
 
    sudo /opt/ruby/bin/ruby /opt/ruby/bin/gem install rmagick

Installing Vim 7.2 From Source
----------------------------------

    svn co https://vim.svn.sourceforge.net/svnroot/vim/branches/vim7.2
    cd vim7.2/
    ./configure --with-features=huge --enable-gui=gnome2 --enable-cscope --enable-pythoninterp
    make
    sudo make install

Add plugins and snippets

    git clone git://github.com/cassiomarques/cmarques-vimfiles.git ~/.vim    
    cp .vim/vimrc ~/.vimrc

    sudo apt-get install exuberant-ctags

It is we would recommend changing the file .vimrc and change the colorscheme ir_black to desert.

Monit
-----

    sudo apt-get install monit

Make sure all of the monit files are in place — /etc/default/monit and /etc/init.d/monit — and then we can remove the package:

    sudo apt-get remove monit

Install dependencies

    sudo apt-get install bison flex

Once we’ve got the dependencies, we can download the source:

    wget http://mmonit.com/monit/dist/monit-5.2.1.tar.gz
    tar xzvf monit-5.2.1.tar.gz
    cd monit-5.2.1
    ./configure
    make
    sudo make install

Now Monit 5 should be installed. We’ll need to update the init script and the configuration file to use the new Monit installation path:

    sudo vim /etc/init.d/monit

Look for the PATH and DAEMON variables — change them so they look like this:

    PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin
    DAEMON=/usr/local/bin/monit    

Configure monit and then edit /etc/default/monit and set the "startup" variable to 1 in order to allow monit to start

    startup=1

Edit monit configuration file in /etc/monit/monitrc. Inside it, add something like this:

    set daemon  120
    set logfile syslog facility log_daemon

    set mailserver smtp.gmail.com port 587 username "user@domain.com" password "userpassword" using tlsv1,
      with timeout 15 seconds

    set eventqueue
        basedir /var/monit
        slots 1000

    set mail-format { from: monit@mydomain.com }

    set alert admin@domain.com

    set httpd port 2812 and
       allow admin:monit

       check system localhost
          if loadavg (1min) > 3 then alert
          if loadavg (5min) > 2 then alert
          if memory usage > 60% then alert
          if cpu usage (user) > 70% then alert
          if cpu usage (system) > 30% then alert
          if cpu usage (wait) > 20% then alert

       ## nginx ##
       check process nginx with pidfile /opt/nginx/logs/nginx.pid
          start program = "/etc/init.d/nginx start"
          stop program  = "/etc/init.d/nginx stop"
          group server

       ## mysql ##
       check process mysql with pidfile /var/run/mysqld/mysqld.pid
          start program = "/etc/init.d/mysql start"
          stop program = "/etc/init.d/mysql stop"
          group database

       ## redis ##
       check process redis-server with pidfile /var/run/redis.pid
          start program = "/usr/bin/redis-server /etc/redis.conf"
          stop program = "/usr/bin/killall -9 redis-server"
          group redis

Start monit:

    sudo /etc/init.d/monit start
    => Starting daemon monitor: monit.

If monit is working, you will watch it automatically restart. Don't forget to check out the web interface on http://youdomain.com:2812

For more information about configure Monit, see [http://rfw.posterous.com/monit-101-an-developers-guide-to-system-monit](http://rfw.posterous.com/monit-101-an-developers-guide-to-system-monit)

Test a rails applicaton with nginx
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
    
Add a new virutal host

    server {
        listen 80;
        # server_name www.mycook.com;
        root /home/deploy/testapp/public;
        passenger_enabled on;
    }
    
Restart nginx

    sudo /etc/init.d/nginx restart
    
Check you ipaddress and see if you can acess the rails application
        

