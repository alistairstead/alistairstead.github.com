---
layout: post
title: "Package and Install your latest Magento 1.5+ extension"
author: Alistair Stead
date: 2011-03-26 21:40
comments: true
categories:
- Magento
---

After spending many hours crafting the perfect Magento module you need to package it either for easy distribution to your client/team or to upload to Magento Connect.

## Package the extension ##

First of all I would recommend reading the following wiki page;

* <a href="http://www.magentocommerce.com/wiki/7_-_magento_connect/packaging_a_magento_extension_in_1.5">Packaging a Magento Extension in 1.5+</a>

**Note - If your planning to upload your module to Magento Connect ensure the package name and the name you enter on Magento Connect match**

<!--more-->

## Installing a module from a local package ##

Internally Magento uses a localised and enhanced version of Pear to manage packages and to distribute and install extensions. Since version 1.4.* the internal pear script has been renamed to mage. In order to use the mage script to run any of the commands it provides you first need to ensure that the executable bit is set for your user.

``` sh
$ sudo chmod 775 ./mage
```

Once your user has permission to execute this script you can run it without any arguments to see a full list of commands;

``` sh
$ ./mage
```

This will list all of the available commands. You can use this script to install remote extensions from Magento Connect or to install local packages such as the one you have just built. To install the local package you need to use the following command, changing the path and package name for the extension to the location of your files.

``` sh
$ ./mage install-file var/connect/PackageName-0.1.12.tgz
```

Thats it your done… Or at least you should be. If you are testing the package in the version of Magento you used to build the package remember to remove the original files first or you will get a bunch of errors. The error output is generally very helpful, simply do what is suggested and try the install again.