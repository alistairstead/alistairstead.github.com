---
layout: post
title: "sfPokaYokePlugin symfony Client Side Validation"
date: 2009-01-14 22:34
author: Alistair Stead
comments: true
categories:
- JavaScript
- Symfony
---

The `sfPokaYokePlugin` (pronounced with the 'e' on the end e.g. poka-yoki).  Simply put it is the use of simple mechanisms that stop mistakes being made.   sfPokaYoke provides client-side form validation based on existing `action.yml` validation files.  This validator will provide fully configurable inline errors on blur events and listed errors once the form is submitted.

This plugin was built after reading "Designing the Obvious" by Robert Hoekman, jr. and working with a number of unsatisfactory  validation libraries. It is intended that this plugin will allow you to configure the validation and feedback to make your forms poka yoke devices i.e. impossible for users to make any errors while entering data.

Thanks to Alon Noy for a starting point for the validators! The following symfony validators have been ported to JavaScript implemented in the plugin:

* sfStringValidator
* sfNumberValidator
* sfRegexValidator
* sfEmailValidator
* sfCompareValidator

I also plan to implement a client side callback validator that will allow you to call your own client side validators or even call ajax function to run server side validation in real time.

<!--more-->

## Installation ##

To install the plugin you can either download the latest files from the symfony wiki or you can checkout the SVN version.

### Install the plugin ###

``` sh
$ symfony plugin-install http://plugins.symfony-project.com/sfPokaYokePlugin
```

### Add the sfPokaYoke filter to your app filters.yml ###


``` yml filters.yml
rendering: ~
web_debug: ~
security: ~

# generally, you will want to insert your own filters here

poka_yoke:
class: sfPokaYokeFilter

cache:     ~
common:    ~
flash:     ~
execution: ~
```
### Clear you cache ###

``` sh
$ symfony cc
```

You're done. Any forms that have validation rules will have client-side rules applied too. sfPokaYoke looks for validation rules that match the action name the form will be submitted to. This will also now work with named routes.

## Usage Example ##

Once any validation rules are applied to the actions for your form, rules will be created for the client-side validation.

You can configure the client side validation to be triggered on the form submit or when the individual field loses focus. This is done by adding config values to your app.yml and will define how sfPokaYokePlugin responds to the users interaction

If the input value fails any validation rules that error will be inserted next to the input.

``` html
<?php echo form_error('name') ?>
```

You do not need to add any `<div>` tags to capture these in-line errors unless you have a specific location in you form markup where you wish the error to be displayed. sfPokaYoke will look for the symfony form error locations and use those `<div>` tags if available.

If your template includes the following symfony form helper, sfPokaYoke will insert the errors into this location.

On form submission a list of all error is inserted as the first child of the form. Each listed error item has an onclick event attached to allow the user to click on the error to focus the form field and resolve the problem.

``` html
<ul id="pkykGlobalErrors">
   <li class="globalErrorTitle">The following form information has been completed but it contains errors:</li>
   <li class="errors" id="pkykGlobal_username">Please enter your username.</li>
   <li class="errors" id="pkykGlobal_password">Please enter your password.</li>
</ul>
```

As each input error is corrected by the user the in-line and list errors are removed.

## Example configuration ##

The code block below is an example app.yml file entry to configure the interactions of sfPokaYoke

``` yml
all:
  # Example app.yml config for sfPokaYoke
  pokayoke:
    # Turn PokaYoke on and off
    on: true
    # Turn on the debuging this will cause alerts at each validation action
    debug: false
    # Event hooks - validation called on both field blur and form submit
    validate_onblur: true
    validateon_submit: true

    # Define which events should display the inline and global form errors
    display_inline_onblur: true
    display_inline_onsubmit: false
    display_global_onblur: false
    display_global_onsubmit: false

    # Inline error id="error_for(_name)" and class="form_error"
    inline_id_prefix: error_for_
    inline_class: form_error

    # &lt;li class="form_error"&gt;&lt;/li&gt; the global error lost class
    global_class: form_error
    # The text used to introduce the global errors list
    global_title: 'The following form information has been completed but it contains errors:'
    global_titleclass: pkyk_global_title

    # Add an onClick event to the items in the global list - click to focus the field with the error
    global_onclick_focus: true;
```