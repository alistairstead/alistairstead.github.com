---
layout: post
title: "Symfony APC Cache and Memcache Session Storage"
date: 2009-11-30 23:22
author: Alistair Stead
comments: true
categories:
- PHP
- Symfony
---

{% img /images/posts/symfony-memcache.jpg 'symfony memcache' %}

In a previous post I described some of the reasons why you would want to store your session data in an alternate location to temporary files on your server. I explained the<a title="symfony Doctrine database session storage" href="/2009/11/symfony-doctri…ession-storage/"> setup of database session storage using Doctrine</a> and how this would allow you to take advantage of storing session data in a database. However depending upon your application requirements storing your session data in a database may cause excessive load on you database. You could of course use a separate dedicated database server to store your session data. However there is an alternate setup that will allow you to centralise your session storage ready for horizontal scaling of your hosting e.g. adding additional web servers.

<!--more-->

symfony provides a very easy way for you to configure your application to use a number of alternate session storage strategies. In this article I will explain the steps needed to setup your application to use first APC as your session storage and then Memcache. Both of these strategies result in the session data being stored in volatile memory on your server and a cookie is stored on the client to maintain reference to the session data. You can set the lifetime of the cookie to control your session timeout.

## Install APC and Memcached modules ##

First we need to ensure that your server has the required packages installed. I am using Ubuntu 9.04 in this example but the packages are available for your preferred linux distro. To install the required packages run the following commands.

Install the required PHP modules and the correct version of apache to successfully install APC, then install the Pecl APC module.

``` sh
$ sudo apt-get install php5-dev php-pear apache2-threaded-dev
$ sudo pecl install apc
```

Install memcache

``` sh
$ sudo apt-get install memcached
```

## Configure APC session storage ##

Once you have your server setup with the required packages you need to configure symfony to use your preferred session storage strategy. To do this you simply need to modify the factories.yml file for you application. You need to modify the section of the factories.yml that defines your application storage strategy e.g.

Amend your factories.yml to include the following settings then change the parameters to reflect you setup such as the domain and session name.

``` yml
all:
  storage:
    class: sfCacheSessionStorage
    param:
      session_name: sfproject #[required] name of session to use
      session_cookie_path: / #[required] cookie path
      session_cookie_domain: example.com #[required] cookie domain
      session_cookie_lifetime: +30 days #[required] liftime of cookie
      session_cookie_secure: false #[required] send only if secure connection
      session_cookie_http_only: true #[required] accessible only via http protocol
      cache:
        class: sfAPCCache #[required] define the cache strategy
        param: ~ #[required] this empty key is required or you will get warnings
```

Once you have made this change clear the symfony cache and try your application. You will not see any noticeable difference unless you check the symfony toolbar and look for the additional cookie stored on the client.

This example defines the use of sfAPCCache as the session storage strategy. APC stores the session data in memory on your web server making it quicker to access than reading a file or database record.

## Configure Memcache session storage ##

To setup memcache session storage it is almost as simple. Amend your factories.yml to include the following settings then change the parameters to reflect your setup including any specific values for your memcache server if you set it up differently.

``` yml
all:
  storage:
    class: sfCacheSessionStorage
    param:
      session_name: sfproject #[required] name of session to use
      session_cookie_path: / #[required] cookie path
      session_cookie_domain: example.com #[required] cookie domain
      session_cookie_lifetime: +30 days #[required] liftime of cookie
      session_cookie_secure: false #[required] send only if secure connection
      session_cookie_http_only: true #[required] accessible only via http protocol
      cache:
        class: sfMemcacheCache #[required] define the cache strategy
        param:
          servers: # Array of servers
            localserver:
              host: localhost # hostname or IP of mamcache server
              port: 11211 # default memcache port
```

Once you have made this change clear your symfony cache and test your application.

Memcache is a volatile storage strategy very similar to RAM in you PC or Mac. It is very quick to access as the session data is held in the server memory rather than in a file or database. Memcache can also be centralised onto a dedicated memcache server allowing your application to be scaled. The example above uses localhost as the memcache server but the host value of the configured server could be modified to be an alternate dedicated memcache server.

I will be running some performance tests using these strategies and also compare the default file based session storage and also the database session storage strategy. If you have already run these tests I would be keen to see your results.

I hope these examples prove useful. If you have any queries or questions please leave a comment.