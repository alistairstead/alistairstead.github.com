---
layout: post
title: "Propel doUpdate update multiple records in a single statement"
date: 2009-10-08 20:59
author: Alistair Stead
comments: true
categories:
- PHP
- Symfony
---

I've been very busy with a number of project launches lately. While working on a little problem today I re-discovered a little gem for use with Propel to update multiple rows in a single statement, rather than iterating through a collection updating and saving each object in turn.

This task is simple in SQL but after a while using any abstraction layer you may find like me you forget about the simple solutions as you spend you day working with complex objects and trying to hydrate custom objects. Blah blah blah...

The complex solution to a simple problem:

``` php
<?php
// Obtain the connection configured
$conn = Propel::getConnection(YourObjectPeer::DATABASE_NAME);
// Create a Criteria object that will select the correct rows from the database
$selectCriteria = new Criteria();
$selectCriteria->add(YourObjectPeer::COLUMB_TO_SELECT, 'value_to_match');
// Create a Criteria object includes the value you want to set
$updateCriteria = new Criteria();
$updateCriteria->add(YourObjectPeer::COLUMB_TO_CHANGE, 'value_to_be_set');
// Execute the query
BasePeer::doUpdate($selectCriteria, $updateCriteria, $conn);
?>
```

Thats it! I hope it helps...
