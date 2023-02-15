---
section: issue
title: Database Outage
date: 2023-02-15T15:45:00.000Z
resolved: true
draft: false
informational: false
resolvedWhen: 2023-02-15T17:10:15.086Z
affected:
  - Database
severity: down
---
dddice experienced a major database outage that affected all services. During the incident, dddice was unusable.

The incident lasted approximately 1-hour 25-minutes.

## Background

dddice uses PlanetScale (MySQL/Vitess) as our primary database. Our database is used to store user information, rolls, dice, and practically all data found across the application. PlanetScale provides us an affordable, scalable, and reliable service so that our team can focus on improving dddice and not necessarily our underlying services.

## Problem

Up until now, dddice was using PlanetScale's free tier which limits the number of transactions we are able to make in a month.

At approximately 10:29am EST, we reached the free tier's limit and observed database transactions starting to fail.

To immediately solve the problem, we upgraded our database in PlanetScale which unknowingly stalled on their backend. This caused our entire database to fail, leaving it unreachable.

## Solution

This incident was raised with PlanetScale's support team who immediately began ivestigation. The investigation took about 1-hour 15-minutes before resolution.

## Timeline

Below is a timeline of the incident and the solutions attempted to resolve it.

* 2023-02-15 10:29 AM - First observed database transaction fails due to plan limit, dddice is unreachable
* 2023-02-15 10:33 AM - PlanetScale plan is upgraded
* 2023-02-15 10:34 AM - dddice returns online for a brief moment
* 2023-02-15 10:38 AM - First observed database is unreachable error, dddice is unreachable
* 2023-02-15 10:56 AM - Incident is filed with PlanetScale support for urgent investigation
* 2023-02-15 11:28 AM - Incident is acknowledged by PlanetScale team
* 2023-02-15 11:54 AM - Incident is updated by PlanetScale team stating the problem and resolution
* 2023-02-15 12:09 PM - Last observed database error, dddice returns online


## Future Solutions

We are actively monitoring the fix and have further thoughts on how to improve our response times, transparency to the community, and performance of the site.

We will be prioritizing "graceful degradation" across all our services that, should our database or any other service fail, our site should be able to handle that failure with some constraints. This means dddice will remain online but may have limited functionality while a service is experiencing downtime.

While we dislike incidents like this, we do trust they make dddice better and more resilient in the future.

Thank you for your support and patience.
