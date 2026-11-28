---
layout: post
title: "Hudson Tip #1 - Doing a release on Hudson using Maven 2"
date: 2008-09-21
---

# Hudson Tip #1 - Doing a release on Hudson using Maven 2

## Introduction

If you haven't figured out how to do a release using Maven 2 within Hudson this will explain it to you so you can set it up and then have a one button release process.

*-- Update -- this is one way of setting up the Maven 2 release process, while it works it requires a bit more setup. Hudson now also has a Maven 2 release plugin. Consider using that instead!*

## Requirements

- Hudson
- Hudson Batch Task plugin
- Maven 2

## Setup

Make sure that the user account Hudson is using has access to a locally installed Maven 2. Verify this by logging in with that account and issuing `mvn`. If this does not work, make sure you get this to work first.

Within Hudson verify if your installation has the "batch-task" plugin installed. See "Manage Hudson" and then "Manage Plugins" for more information. If the "batch-task" plugin is not installed, install it and restart Hudson.

And now it is time to get this done, open a project's configuration and look for the "Batch Tasks" checkbox. Click it and then fill something similar in like what you see in the settings below. Since we use Subversion we need to make sure we pass in the Subversion username and password for the release plugin as you can see in the snippet below.

```
Name:    Release
Script:  mvn --batch-mode -Dusername=username  
         -Dpassword=password release:prepare release:perform
```

This completes the configuration. And if you want to do a release, then go to the project, click on "Task" and then on the "Execute" button for the task and voila! If the release fails for some reason you can see what happened in the Output of the Task.

You are now able to release using Maven 2 on Hudson.

(Hudson 1.252+)

*Posted September 21, 2008*

[Up](../../../)
