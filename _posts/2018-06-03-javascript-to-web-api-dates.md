---
layout: single
title: "Dates: From JavaScript to Web Api"
date:  2018-06-01 00:00:00 -0500
categories: javascript
comments: true
tags: javascript json web-api date
published: true
---

Dates can be tricky things. Let's consider the case of getting a date from a Javascript front end all the way to SQL Server.

In Javascript we have something that looks a lot like this.

```
let f = new Date()
Fri Jun 01 2018 16:15:03 GMT-0400(Eastern Daylight Time)
```

Scratch that. Lets clean that up and get rid of those distracting hours, minutes and seconds.

```
let f = new Date()
f.setHours(0, 0, 0)
Fri Jun 01 2018 00:00:00 GMT-0400 (Eastern Daylight Time)
```

When we serilaize this as JSON to send to a web service, this gets converted.

```
JSON.stringify(f)
"2018-06-01T04:00:00.000Z"
```

The output here is the same as `f.toISOString()` where the ISO in question is <a href="https://www.iso.org/iso-8601-date-and-time-format.html" target="_new" alt="booooring">ISO 8601</a>. 

```
f.toISOString()
"2018-06-01T04:00:00.000Z"
```

The timezone here is denoted by the "Z" for "Zulu" (which [is not](https://english.stackexchange.com/a/54152) as problematic as it sounds, my woke friends), and is pretty much the same as GMT, give or take the occasional second. Also neat:  GMT stands for Greenwich Mean Time which is sometimes the time at the Royal Observatory in Greenwich, London, except during daylight savings when they switch to British Summer Time (BST).

When this date is posted to Web Api we can have the model binder assign it to (among other things) either a `DateTime` or `DateTimeOffset` field. When we bind to a `DateTime` we get the following

```
? request.DueDate2

//DateTime offset
{6/1/2018 4:00:00 AM}
  Date: {6/11/2018 12:00:00 AM}
  Day: 1
  DayOfWeek: Saturday
  DayOfYear: 174
  Hour: 4
  Kind: Utc
  Millisecond: 0
  Minute: 0
  Month: 6
  Second: 0
  Ticks: 63665312000000000
  TimeOfDay: {04:00:00}
  Year: 2018
```

When bound to a `DateTimeOffSet` we get this: 

```

? request.DueDate
//DateTime offset
{6/1/2018 4:00:00 AM +00:00}
  Date: {6/1/2018 12:00:00 AM}
  DateTime: {6/1/2018 4:00:00 AM}
  Day: 1
  DayOfWeek: Saturday
  DayOfYear: 174
  Hour: 4
  LocalDateTime: {6/1/2018 12:00:00 AM}
  Millisecond: 0
  Minute: 0
  Month: 6
  Offset: {00:00:00}
  Second: 0
  Ticks: 63665312000000000
  TimeOfDay: {04:00:00}
  UtcDateTime: {6/1/2018 4:00:00 AM}
  UtcTicks: 63665312000000000
  Year: 2018
```

And when we save to SQL Server with matching column types of `DateTime` and `DateTimeOffset` we see the following values, respectively.

```
2018-06-01 04:00:00.000
2018-06-01 04:00:00.0000000 +00:00	
```

Those look pretty much alike, but beauty of the DateTimeOffSet is that the offset gives us a point of reference.

Let's say on June 1st at 9 AM you ran a job to process all entries where the date was greater that the DateTime column of 0400. No matter where you are in the world, be it Disneyland or Narnia, 9 AM is more than 4 AM and this row will be processed. However, sometimes the context of 4AM matters.  `6/1/2018 4:00:00 AM +00:00` gives us this context. 9 AM in Washington, DC is well after after this time, but 9 AM in Laos, Cambodia is a few hours prior to this (there is an excellent and much better explaination [here](https://stackoverflow.com/a/14268167/190592))






