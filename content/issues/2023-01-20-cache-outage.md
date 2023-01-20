---
section: issue
title: Cache Outage
date: 2023-01-19T23:58:02.289Z
resolved: true
draft: false
informational: false
resolvedWhen: 2023-01-20T12:46:02.306Z
affected:
  - Application
  - API
  - Cache
  - Worker
severity: down
---
dddice experienced a major outage that affected rolling dice and user sessions. During the incident, users were periodically logged out from their accounts and rolls would not appear or take a long time to appear. This affected all dddice plugins and the main website.

The incident lasted approximately 12-hours.

## Background

dddice uses Redis as our main cache backend which stores user sessions, queued rolls, and queued assets for processing.

During a traffic spike our queue workers were overwhelmed which caused a backup in the queues, reaching >90% storage capacity. New rolls were not being queued and partial page caches could not be generated which resulted in some sections of the site being inaccessible and caused rolls to not roll.

## Problem

Our worker uses a “main” process that monitors “child” processes. This worker is powered by a single 2GB memory VM. Initial configuration of this worker assigned 1.5GB to the “main” process and 256MB to the “child” processes. The worker was also configured to manage a maximum of 10 “child” processes.

Incorrect assumptions about the configuration of the worker were made which resulted in the worker VM to consume all resources. This caused all queues to halt, including rolls.

When the worker was restarted on multiple occasions, the queue would begin processing which immediately began to overwhelm the Redis cache.

In short, redis was overwhelmed because of invalid configuration in our worker process.

## Solution

This incident resulted in two fixes.

1. Redis memory was updated from 100MB to 256MB to handle an increase in traffic.
2. Worker configuration was updated to consume less memory on the “main” process and consume more memory on the “child” processes. Child processes were scaled down from 10 to 3 in order to stay within VM resource limits. This configuration has been proven to be more effective after careful analysis of how our worker operates.

## Timeline

Below is a timeline of the incident and the solutions attempted to resolve it.

* 2023-01-19 6:48 PM - First out-of-memory (OOM) appears in error log
* 2023-01-19 6:57 PM - First user reported that they were frequently being logged out
* 2023-01-19 7:31 PM - Application memory increased in attempt to alleviate OOM errors
* 2023-01-19 7:36 PM - Redis cache cleared in attempt to free memory
* 2023-01-19 8:00 PM - Outage announced in #announcements channel in Discord
* 2023-01-19 10:21 PM - Release v0.9.9-252 started, fixes unrelated bug
* 2023-01-19 10:21 PM - Release v0.9.9-252 finished, restarted queue worker
* 2023-01-19 10:23 PM - Service observed as restored, with rolls from rooms and dice preview pages working as expected
* 2023-01-19 10:25 PM - Message posted in #announcements that update was made and incident was over
* 2023-01-19 10:29 PM - OOM errors started reoccurring, investigation continued
* 2023-01-19 11:11 PM - Status page of Redis hosting checked, urgent incident noted
* 2023-01-19 11:19 PM - Message posted in #announcements that outage was not over and we were still investigating an upstream provider (Redis)
* 2023-01-20 6:45 AM - Redis memory upgraded
* 2023-01-20 7:23 AM - Release v0.9.9-255 started, updates worker configuration
* 2023-01-20 7:34 AM - Release v0.9.9-255 finished
* 2023-01-20 7:36 AM - User reports that dice are again rolling for them
* 2023-01-20 7:40 AM - Internal communication made that errors stopped appearing in the error log and rolling was again working
* 2023-01-20 7:46 AM - Announcement made in #announcements that outage was over

## Future Solutions

We are actively monitoring the fix and have further thoughts on how to improve our response times, transparency to the community, and performance of the site.

* We plan to create scripts that automatically update this status page so that incidents can be seen by the community immediately.
* We plan to change how rolls are processed from asynchronous (queue) to synchronous (immediately)
  * This change requires some thought into our server resourcing which we are equipped to handle but want to ensure that proper steps are taken before making the change.