---
layout: post
title: "Symfony Doctrine Database Session Storage"
date: 2009-11-30 22:23
author: Alistair Stead
comments: true
categories:
- PHP
- Symfony
---

Sessions are normally managed by your server and depending upon how involved you get with the PHP setup you need never know how this data is stored. The default setup for PHP is to store session data in temporary files on the server. So why would we want to change how sessions are handled and what advantage can we gain from any changes we make.

## Why use database session management? ##

Sessions allow you maintain persistence within PHP applications. With every HTTP request a PHP application must re-assign every variable and object needed in your application. Sessions allow you to store data that is required to persist past each HTTP request allowing it to be easily retrieved and used.

<!--more-->

Sessions are commonly used when creating a secure application, used to store the state of a user e.g. if they are logged in or credentials to control access to content.

By storing session data in a database table you are able to create an application interface that will show information about the users that are logged in. This for example could show the number of users logged into the application or show administrators who is logged in.

If you need to scale your application by adding more than one server, saving your session data in temporary files can lead to major problems. By using database storage you can optimise and unify the session storage, ensuring that session data is always available and consistent.

Obviously there are other solutions for scalable session storage such as APC or MemCache but these solutions are for another post.

## How to setup Doctrine database session storage ##

You first need to add a table to you database schema that will be used to store your session data. Although you can create this table manually, adding it to your scheme also created the model classes that you can later use to interact with the table data.

``` yml
Session:
  columns:
    id: { type: string(32), primary: true, notnull: true }
    data: { type: string(4000), notnull: true }
    time: { type: integer(4), notnull: true }
```

Once you have added the table to your schema you can build your model and database:

``` sh
$ symfony doctrine:build-all
```

You then need to configure your symfony application to use this table to store all session data. To do this you need to modify the factories.yml file for your application. Update your file to include the following lines:

``` yml
all:
  storage:
    class: sfPDOSessionStorage
    param:
      db_table: session # Name of the table storing the sessions
      db_id_col: id # The primary key column
      db_data_col: data # The column where the session data will be stored
      db_time_col: time # The column where the timestamp of the session will be stored
      database: doctrine # Name of the database connection to use
```

Once you have modifies your factories.yml file you will need to clear the symfony cache. After that your application should run exactly as it did before except that your users session data will be stored in the database table.

The data column stores all the session data in a serialised form this is deserialized each time a user requests the data. There are many uses for this type of session storage and I hope this brief explanation helps you.
