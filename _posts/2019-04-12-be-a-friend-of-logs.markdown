---
layot: post
title:  "Be A Friend of Logs"
date:   2019-04-12 17:00:00 -0400
categories:
tags: [ruby]
comments: true
---

# Overview
Good utilization of logging can make developers' lives better.

Because logs are good sources of
* Debugging
* Finding out the utilization of features
* Users' behaviors analysis
* Providing data for product improvements
* etc

*Sumologic is a great tool for log inquiry.*

I am going to give a brief introduction to using Sumologic at the end of this essay.

# Airbrake or Rails logger?

**When should we use Airbrake?**
For those system exceptions or errors which can cause dysfunction on a feature and need our attention immediately. Like fatal errors.

**When should we use Rails logger?**
For that less important stuff.

# Get started
# Step 1 - Compose Stories
The first thing is trying to compose some stories which can help you understand what you need from the logs.

Let's say there is a clinic reservation system
e.g. I want to see how many patients get abandoned from the reservation flow and why they were failed

**Find a good standpoint of a story**
* I want to monitor all failures from a new API
* I want to see how many people are using this newly launched feature
* I want to see the avg. attempts before a user successfully signs up an account
* I want to see how many times there is no result showing up in the search a doctor after users click the search button
* etc

**Bad stories**

* I want to see whether this prod is successful

**Find the keywords from stories**
To understand the keywords better, we need to parse the story and figure out the code implementation for that story.

Use one of the above examples - I want to see the avg. attempts before a user successfully submits the sign-up form.

The keywords are

* A user - Need to track logs by unique user ID or session
* Registration form - Add logs to consultation submission function at front-end after validation and after API response, it should be caught in the front-end
* Reason - missing part of the story. I need to know why a user repeatedly submit the form with errors.
* After refining the story, it becomes - I want to see the avg. attempts for a member to submit the sign-up form and the reasons for failing at submission.

**Document the story**
* Unique Keyword which can be easily located to the source code. e.g. "SignUp Form Failure"
* Description
* Place of the code base where the log will be dwelling.
* Sumologic Query

# Step 2 - Implementation
Don't miss anything defined in the stories

Use the same example as above

* User ID / Session
* User-submitted params
* Filter sensitive info

# Step 3 - Release and Test and Refine the Logs
QA by yourselves

Search the newly added logs in Sumologic

Adjust the logs until it works for you

# Step 4 - Set Up Sumologic Queries
Here is an example query of Sumologic

```
SignUp Error | parse "\"uuid\"=>\"*\"" as uuid | parse regex "\"referrer_url\"=>(?<referrer_url>(?:nil|\"(?:\w+|\S+)\"))" | parse "\"errors\"=>[*]" as error | where !(referrer_url matches "nil") | timeslice 1d | count(uuid), count_distinct(uuid) by _timeslice | sort by +_timeslice
```

It does
* look up the keyword
* look up the unique user session
* group by time and user session
* sort by time
* filter "referrer" is not nil

# Step 5 - Diagnosis and Analysis
When you get all that information from the previous step, you can get the number of failures for a day. Then you can set up an email notification when the number is higher than the threshold.

You can diagnosis the error to find out the root causes.
