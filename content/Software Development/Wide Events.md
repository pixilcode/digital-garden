---
title: Wide Events
tags:
  - link
  - tips
  - topic-software-development
  - topic-logging
date: 2026-07-06
---
**Link: [Logging Sucks - Your Logs Are Lying To You](https://loggingsucks.com/)**

The way logging is usually done is not very useful. There are a bunch of messages in a non-standardized form, and trying to grep through all of them can be a nightmare.

However, there is a better way to do it: _wide events_. _Wide events_ are logs that are structured data (like JSON) that have all the important details of a given request/response. Rather than having logs split up throughout the response process, log one wide event at the end.

_See also: [Using Canonical Log Lines for Online Visibility](https://brandur.org/canonical-log-lines)_

