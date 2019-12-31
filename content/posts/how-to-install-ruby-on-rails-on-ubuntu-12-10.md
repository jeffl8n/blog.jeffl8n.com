---
title: "How To Install Ruby On Rails on Ubuntu 12.10"
date: 2012-11-20T20:55:26-05:00
---

After much searching online, and a bit of trial and error, I have come up with the following commands, which when run should result in a working Ruby on Rails environment on Ubuntu 12.10.  I haven’t tested these steps on any other version of Ubuntu, but I imagine it would be at least similar for other recent versions of Ubuntu.

First you need to make sure your system is up to date:

```
sudo apt-get update
```

Next, you’ll need to install curl and NodeJS.  Curl is required to download the rvm bash script and NodeJS is needed for your local javascript execution environment.  If you don’t install NodeJS (or another javascript execution environment), you will see an error when you try to run a web app.

```
sudo apt-get install curl nodejs
```

The next commands will install RVM (Ruby Version Manager), Ruby 1.9.3, and all other dependencies needed for Ruby.

```
curl -L get.rvm.io | bash -s stable --auto
source ~/.rvm/scripts/rvm
. ~/.bash_profile
rvm requirements
# The next line should be what rvm requirements said to install for Ruby
sudo apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion pkgconfig
rvm install 1.9.3
```

This last command will initially show a screen which tells you to run the earlier apt-get install command.  You should press **q** to skip this screen.

When ruby has finished installing (which may take a few minutes), you should see something like:

Install of ruby-1.9.3-p327 - #complete
The part I have made bold is the version & patch level you should use in the next command.  This command will make sure Ruby is the correct version by default.

```
rvm --default use 1.9.3-p327
```

Now that you have installed Ruby, you can install Rails.  This command installs Rails 3.2.13

```
gem install rails -v 3.2.13
```

You should now have a working version of Ruby on Rails on Ubuntu 12.10 setup.

To test this, you can run:

```
rails new blog
cd blog
rails s
```

The first command creates the basic Ruby on Rails application framework, the second command just moves your terminal into the new directory, and the third command starts a local development server.

If these 3 commands work, you will be able to open http://localhost:3000 in your favorite browser and view the default ruby web app.