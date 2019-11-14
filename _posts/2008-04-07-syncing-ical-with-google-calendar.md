---
id: 508
title: Syncing iCal with Google Calendar
date: 2008-04-07T21:35:40+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=508
permalink: /2008/04/07/syncing-ical-with-google-calendar/
categories:
  - Uncategorized
---
**EDIT:** This exercise isn&#8217;t necessary any more because Google Calendar now supports CalDAV. Check [this](http://www.google.com/support/calendar/bin/answer.py?hl=en&answer=99358) out for more info.

A major caveat with Google Calendar is the absence of any CalDAV support, meaning you can distribute calendars through subscription to a feed. This is fine but still doesn&#8217;t let you edit events in a client app like Mozilla Sunbird or iCal. However it&#8217;s possible to manipulate your calendars through the GCalendar API; the [Google GData talk](http://www.youtube.com/watch?v=ADos_xW4_J0 "GData talk") on YouTube is particularly worth watching.

Anyway, I digress; after exhausting my SpanningSync trial and not being particularly impressed with the way it blindly syncs both sources at a predefined interval I decided to hack out my own ruby script which I could daemonize later.

The approach I took was to instantiate a WEBrick server to act as a go-between so you could &#8216;publish&#8217; the calendar to localhost. As soon as a PUT comes through from a client with the iCal user agent, the request body is compared to the previous file (if it exists).

I then use the icalendar and googlecalendar gems to parse the body if a change has occurred, and push the changes to Google. The only problem here is that there doesn&#8217;t seem to be any sensible way of performing a diff on the local and remote data (and even if there were it&#8217;d be pretty bandwidth-intensive) so icalendar just iterates through every event in the old file, deletes it remotely, then adds the new one. Rinse and repeat. Or is that bleach and repeat?

I will update this soon, but this is the fruit of an hours&#8217; labour:

<http://gist.github.com/40274>