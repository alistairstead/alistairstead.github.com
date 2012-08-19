---
layout: post
title: "YUI Connect asynchronous file uploads"
date: 2009-11-23 14:26
author: Alistair Stead
comments: true
categories: JavaScript
---

I'm sure the problems with multi-part data that I have been working through this morning is very simple but there are a number of forum posts about this, so I will share my findings. When using YUI Connection Manager in conjunction with multi-part form data I have experienced problems creating the asynchronous request and canceling the form submission. Even after setting up the request and callback object and checking the YUI API repeatedly the problem resides elsewhere.

``` js
YAHOO.util.Event.addListener(forms, 'submit', this._onFormSubmit, this); 
_onFormSubmit: function(e, o) {    
    YAHOO>util.Event.preventDefault(e);
    // Set the form  YAHOO.util.Connect.setForm(e.currentTarget);
    // Configure the callback object
    callBackObj = {
        success: o._loadRemoteFormSubmissionTemplateComplete,
        failure: o._loadRemoteFormSubmissionTemplateFailed, 
        argument: {
            objType: objType,
            region: regionObj,
            mode: submitbutton.value
        }
    }
    // Make the request
    YAHOO.util.Connect.asyncRequest('POST', e.currentTarget.action, callBackObj);
}
```

With non multi-part forms you can hijack the submit event of the form and create an asynchronous request. However once you try with multi-part data no matter how you set this up and attempt to cancel the form submit action the form will still make an HTTP request in the main window.

## The YUI Multi-Part Data Solution ##

The solution is very simple once you find it but it is easy to overlook! In order to use YUI connect to create an asynchronous request you must remove the 'submit' button and replace it with a 'button' then assign it an eventListener that will create the asyncRequest.

``` js
var alternateButton = document.getElementById('alternateButtonId')
YAHOO.util.Event.addListener(alternateButton, 'click', this._onFormSubmit, this);
_onFormSubmit: function(e, o) {
    YAHOO.util.Event.preventDefault(e);
    // Set the form
    YAHOO.util.Connect.setForm(e.currentTarget);
    // Configure the callback object
    callBackObj = {
        success: o._loadRemoteFormSubmissionTemplateComplete,
        failure: o._loadRemoteFormSubmissionTemplateFailed,
        argument: { 
            objType: objType,
            region: regionObj,
            mode: submitbutton.value
        }
    }
    // Make the request
    YAHOO.util.Connect.asyncRequest('POST', e.currentTarget.action, callBackObj);
}
```

When trying to use multi-part data YUI Connection Manager will create an iframe in the document that your form will submit the data to. If you try and use the submit handler YUI currently does not change the target value of the submit event.

The YUI examples do all show the use of alternate buttons rather than a standard submit button however it can be easily overlooked. I hope this helps someone!