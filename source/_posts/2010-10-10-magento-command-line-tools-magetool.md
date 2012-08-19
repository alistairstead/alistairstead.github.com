---
layout: post
title: "Magento command line tools MageTool"
date: 2010-10-10 00:05
author: Alistair Stead
comments: true
categories:
- PHP
- Magento
---

If you have spent any time developing with Magento I expect that like me you have found a number of tasks you find yourself repeating many many times. While developing new modules or themes there are many times when you need to clear the Magento cache or even disable it completely. Now of course you can log into the admin system and carry out these actions but have you every thought there must be an easier way?

Many development frameworks or other open source projects have tool that can be used by developers to issue commands quickly and easily. Magento however does not have any built in tools to improve the development workflow. This is where MageTool fits in!

## MageTool ##

How can <a href="http://github.com/alistairstead/MageTool">MageTool</a> help? Well hopefully in a number of ways. MageTool extends the existing Zend Framework command line tool zf, adding new commands that are specific to Magento development workflow. Rather then navigating the Magento admin system to clear cache or the enable or disable caching you can issue a simple command in your terminal window and achieve the same results. You can also modify the Magento configuration quickly or issue commands that will selectively update many configuration values.

In my standard development practices I maintain a number of Magento installs on different servers. These are used for Development, Staging and UAT each using different domains to access the stores. This means however that I often need to move databases between servers, resulting in the ‘base_url’ configuration being incorrect. This can be easily updated if you have database access and is fairly simple if you only have a couple of sites setup. However if you have more sites setup this can become very very difficult to maintain and calls into question the idea of working with many environment servers during development.

MageTool can help again in any situation where you need to move a Magento install and then update the ‘base_url’ configuration. You can construct a command that will quickly and easily update all URLs in the Magento config.

<!--more-->

## Install ##

First install ZF:

``` sh
$ sudo pear channel-discover pear.zfcampus.org
$ sudo pear install zfcampus/zf
```

Then install MageTool:

``` sh
$ sudo pear channel-discover pear.magetool.co.uk
$ sudo pear install magetool/magetool
```

Once you have installed ZF and MageTool you will need to create configuration for your user by creating the following file:

``` sh
$ vim ~/.zf.ini
```

Add the following lines to load the additional MageTool commands making them available to zf:

``` ini
basicloader.classes.1 = "MageTool_Tool_MageApp_Provider_Manifest"
basicloader.classes.2 = "MageTool_Tool_MageExtension_Provider_Manifest"
```

After the install has completed successfully if you run zf you will see the additional commands available for Magento.

``` sh
  MageAdminUser
    zf show mage-admin-user
    zf create mage-admin-user username email password firstname[=Admin] lastname[=User]

  MageCoreCache
    zf clear mage-core-cache
    zf enable mage-core-cache
    zf disable mage-core-cache

  MageCoreResource
    zf show mage-core-resource
    zf delete mage-core-resource path

  MageCoreConfig
    zf show mage-core-config path scope
    zf set mage-core-config path scope value
    zf replace mage-core-config match value path scope

  MageApp
    zf version mage-app

  MageExtension
    zf create mage-extension code-pool vendor-name extension-name
```

## Example Usage ##

### Cache ###

To clear the entire Magento cache simply run the following command from within the Magento project root directory:

``` sh
$ zf clear mage-core-cache
```

To disable the entire Magento cache:

``` sh
$ zf disable mage-core-cache
```

### Core Config ###

MageTool provides commands to show existing configuration values and to set their values. It also has an additional command that can be used to perform a str_replace on the config values. This can be used if you have moved the database from another server with a different domain to update the config values in a single command:

``` sh
$ zf replace mage-core-config --match www.current-domain.com --value shop.new-domain.co.uk
```

This will update all the config value that include the string “www.current-domain.com” and substitute it for “shop.new-domain.co.uk”. This can also be further refined to only affect specific config paths or scopes.

### Core Resource ###

During the development of a new extension you sometimes need to remove the resource from the internal registry to force it to re-run the install scripts. MageTool can help you first see which versions of each extensions are installed:

``` sh
$ zf show mage-core-resource --code mage_log
```

You can also remove specific entries:

``` sh
$ zf delete mage-core-resource --code mage_log
```

### Admin User ###

If you inherit a Magento project you may need to create a new admin user to gain access to the system. MageTool can list existing admin users and email addresses or it can create a new user for you to use.

``` sh
$ zf create mage-admin-user --username newadmin --email newadmin@project.com --password newpassword
```

## Roadmap ##

I plan to add additional functionality to create skeleton files for new extensions or themes, removing the repetitive creation of directories and configuration files when creating new extension.

If you have additional suggestions please let me know or alternatively you can fork the <a href="http://github.com/alistairstead/MageTool">MageTool</a> project on github and add the commands before sending me a pull request.

## Feedback and Bugs ##

If you have any problems using MageTool or if you have any feedback or suggestions please <a href="http://github.com/alistairstead/MageTool/issues">submit them here</a>.