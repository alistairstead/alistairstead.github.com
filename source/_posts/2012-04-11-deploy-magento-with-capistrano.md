---
layout: post
title: "Deploy Magento with Capistrano"
author: Alistair Stead
date: 2012-04-11 23:46
updated: 2012-04-16 21:40
comments: true
categories:
- Magento
- Capistrano
---

So you have a Magento project and there must be a better way than you currently have to deploy your code to a server? Well there are many ways to do this from FTP/SFTP, rsync or SCM checkouts. You could even build a package and deploy using a package manager. However there is one options that from experience fits all eventualities and can be turned into a repeatable process where you don't have to remember a list of manual steps. It can even have inbuilt functionality to revert a deployment if for some reason you need to.

## What would we like from a deployment system ##

* Fast deployment.
* Repeatable process.
* Remove all manual steps or tasks.
* Easy and quick roll back to a previous state.
* Simple to use once setup.

## Extra credits ##

* Extensible to cover your specific needs.
* Work across a cluster of servers .
* Work across servers with different roles for the application.

{% pullquote %}
Capistrano originally developed to deploy Ruby on Rails applications has evolved through a number of iterations to become a general purpose deployment tool that can be used for deploying any type of code base or packaged application. {"Capistrano - provides all of the features we are looking for, plus all the extra credit features and more."} It has been used for many projects and is tried and trusted in the PHP community too. You will find many existing Capistrano scripts or pre-packaged Gems that provide functionality to deploy specific PHP applications and to deal with their specific requirements including Magentify.
{% endpullquote %}

<!--more-->

## Magentify ##

I have created a specific Gem extension to Capistrano that will provide the specific functionality you need to deploy a Magento project with Capistrano. This can be easily installed and configured to you exact deployment requirements.

### Prerequisites ###

1. You must have SSH access to each of the machines you will be deploying to.
2. You must have the same user and password on each of these machines but ideally you will have an SSH Key setup to provide access.

### Install ###

To use Capistrano it needs to be installed on a single machine. The machine that will be running the deployment. For my personal projects that is my laptop but for work projects we have a central deployment server that is all setup for this purpose.

    $ gem install capistrano
    $ gem install magentify

***You may need to add `sudo` to these install commands depending upon your user and access levels.***

### Magentify your Magento project ###

Although you can use an alternate structure by modifying the paths in the generated configuration file. I prefer to use a directory structure that allows me to keep my project tools within a project codebase. Maintaining them in source control but outside of the public document root and away from harm. This article is written with the assumption that your code repository looks something like this but you are free to use the structure that suite you:

    .
    ├── README.md
    ├── public # This is our Magento src code and eventual web server doc root
    │   ├── LICENSE.html
    │   ├── LICENSE.txt
    │   ├── LICENSE_AFL.txt
    │   ├── RELEASE_NOTES.txt
    │   ├── app
    │   ├── cron.php
    │   ├── cron.sh
    │   ├── downloader
    │   ├── errors
    │   ├── favicon.ico
    │   ├── get.php
    │   ├── includes
    │   ├── index.php
    │   ├── index.php.sample
    │   ├── install.php
    │   ├── js
    │   ├── lib
    │   ├── mage
    │   ├── media
    │   ├── php.ini.sample
    │   ├── shell
    │   ├── skin
    │   └── var
    └── tools
        ├── capistrano # This is where we will create our Capistrano scripts
        ├── chef
        └── vagrant

Run the following command to initialize your project with the base Magento specific Capistrano configuration that we will update to your specific requirements.

    $ cd tools/capistrano
    $ magentify .
    
This will generate the following files:

    .
    ├── capistrano
    │   ├── Capfile
    │   └── config
    │       └── deploy.rb

In order to configure and customize the deployment to your specific needs you will need to edit `deploy.rb`

``` ruby deploy.rb
set :application, "set your application name here"
set :domain,      "#{application}.com" 
set :deploy_to,   "/var/www/#{domain}"

set :repository,  "git@github.com:/user_name/#{application}.git" # Enter the path to your SCM repository e.g. Github
set :scm,         :git
# Or: `accurev`, `bzr`, `cvs`, `darcs`, `subversion`, `mercurial`, `perforce`, `subversion` or `none`

role :web, 111.111.111.111   # Your HTTP server, Apache/etc
role :app, 111.111.111.111   # This may be the same as your `Web` server or a separate administration server

set  :keep_releases,  3

# Turn off use of sudo if your user does not need elevated permissions
set :use_sudo, false

set :app_symlinks, ["public/media", "public/var", "public/sitemaps", "public/staging"]
set :app_shared_dirs, ["public/app/etc", "public/sitemaps", "public/media", "public/var", "public/staging"]
set :app_shared_files, ["public/app/etc/local.xml"]
```

Once you have configured the details of your application and the servers you wish to deploy your code to you can run the Capsitrano commands. You should run the Capistrano commands from the `tools/capistrano` directory so that the configuration you have created will be used.

### Configure your servers ###

For full details of the options you can use in the configuration file it might be worth reading [Capistrano Getting Started](https://github.com/capistrano/capistrano/wiki/2.x-Getting-Started) or if you have time a slightly longer article that will cover all of your questions, [Capistrano from the Beginning](https://github.com/capistrano/capistrano/wiki/2.x-From-The-Beginning).

#### Hosts ####

Capistrano is designed specifically to deploy to many servers. It can deploy code to 1 to 'n' servers with minimal configuration. In the configuration above we are deploying to two servers one public web server and one admin server. If you need to deploy to more servers you simply need to add config lines for each of the additional servers.

``` ruby deploy.rb
role :web, 111.111.111.101   # Web server 1
role :web, 111.111.111.102   # Web server 2
role :web, 111.111.111.103   # Web server 3
role :web, 111.111.111.104   # Web server 4
role :app, 111.111.111.110   # Database and administration server
```

#### Roles ####

You don't simply have to deploy code to all of your servers in your cluster. You can define and configure roles applied to your servers so that the correct actions are carried out on each of the servers when you run your deployment tasks.

#### SCM ####

Capistrano will integrate directly with your preferred source control system. It has specific features for each system providing alternate strategies to limit the amount of data that is transferred.

``` ruby deploy.rb
set :repository,  "git@github.com:/user_name/magento_application.git"
set :scm,         :git
```

### Setup ###

The first task you should run is the `deploy:setup` this will connect to each of the servers you configured in the previous steps. For those servers with roles that should include a code checkout it will create a set of directories into which your code will be checked out.

    $ cap deploy:setup
    
You can then run `deploy:check` to confirm that each of the servers you configured previously have all of the prerequisite software, access and setting to complete the code checkout and all of the deployment tasks.

    $ cap deploy:check

### Deploy ###

Once you are ready to deploy your code to the server(s) you can run the default `deploy` task. This will connect to each of the servers you have configured and create a new release of your application code on the server. Once all of the servers have the complete version of the code and a synchronized, the symlink for the current release will be updated. This final step takes milliseconds, meaning that this deployment can be run on production servers and in most cases users will not see the deployment happen as it is so quick and ensures not users can access an incomplete release.

    $ cap deploy
    
Files that are located within the shared directory e.g. `local.xml` configuration or uploaded images are maintained between releases. Images uploaded by users should not be maintained in source control so it is key that they are kept in a safe location not changed between releases.

### Roll Back ###

The previous releases of the application code are maintained on the servers so if you need to revert to a previous version for some reason you can simply roll back the deployment. This a quick and repeatable process that means you always have a rollback for each deployment. A safety net included with each deployment has to be a good thing right?

    $ cap deploy:rollback

### Cleanup ###

If you want to clean up all these past releases and only keep the last five releases, you can run the cleanup command and automatically remove the older files.

    $ cap deploy:cleanup

### Magento Tasks ###

The Magentify scripts provide additional tasks that can be run on the servers. These tasks provide access to run a number of remote tasks that are specific to Magento. To see the full list of available tasks you can run the following command.

    $ cap -T
    
You can see the Magento specific tasks in the `magento` namespace.

#### Maintenance Page ####

If you need to place the maintenance page on one or more of your servers you can use the extended Capistrano task:

    $ cap mage:disable
    
The will create a `maintenance.flag` file in the document root of your magento store. Once in place magento will route all traffic through the 503 error page that you can theme to your specific needs. This will prevent access to any part of your magento store including the admin.

*You can of course customise this behaviour to meet your needs or to provide access from your specific IP address.*

To remove the maintenance page and allow general customers to access your store again. Run the following Capistrano command:

    $ cap mage:enable

#### Clear Cache ####

This command currently clears the Magento file based cache but can be extended to clear the specific cache system you are using with your installation.

    $ cap mage:cc

#### Compiler ####

The compiler commands provide a mechanism to enable and disable the Magento compiler. There is also a step in deployment that will compile Magento on each deployment.

    $ cap mage:enable_compiler

    $ cap mage:disable_compiler
    
#### Indexes ####

The indexer command will run the Magento indexers using the shell script. This will populate the index tables following any data changes during deployment.

    $ cap mage:indexer

### What next? ###

You can extend the script to include your own specific requirements. You will find a wealth of information available to you about [Capistrano](https://github.com/capistrano/capistrano/wiki) and creating custom tasks.

{% pullquote %}
If you want to add a GUI to your deployment so that none technical people can run the deployment processes or any of the additional tasks that you make available however, {"be careful giving your clients the tools to make bad things happen."} Tool such as [Webistrano](https://github.com/peritor/webistrano/wiki) or [Strano](https://github.com/joelmoss/strano) provide a graphical interface on top of the command line functionality of Capistrano.
{% endpullquote %}

This is an open source project feel free to [fork](https://github.com/alistairstead/Magentify/fork_select) the code and extend it with new features. Just send me pull requests and I will integrate them with the project. If you don't have the time to create the features then [submit feature requests](https://github.com/alistairstead/Magentify/issues) and maybe the community can help.