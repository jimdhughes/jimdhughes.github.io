+++
author = "James D Hughes"
categories = ["techsplanations"]
date = 2020-04-13T21:11:48Z
description = ""
draft = true
slug = "sharepoint-throttling"
tags = ["techsplanations"]
title = "Sharepoint Throttling"

+++


My new job has me explaining very technical things to very non-technical people.

Every so often I have an explanation that, in my opinion at least, is so simple and drives the point home so well that I just have to sit back and smile.  I decided that I am going to document these experiences and share it for the nerd world to see! If I get indexed and you happen to be able to find some solutions to an issue you may be experiencing then that will be a happy bonus.

# The Scenario

T'was a bleak midsummer evening.  I had just taken my new role when our public website had an outage during an event.  The outage was intermittent and lasted ~20 minutes.  During this time, our public website, which is back-ended by SharePoint, became overwhelmed with requests and started dropping requests.  This led to a lot of questions from a lot of people on top of our digging in to why it had happened.

The odd part was - our farm servers were bored the entire time.  Our monitoring solutions showed that our RAM and CPU stayed stagnant and under-utilized.  Our SQL Server cluster showed the same.  Our Systems Analysts were able to update the throttling thresholds and load test our pre-production systems to ensure that we wouldn't fall over.  At the end of the scenarios, we easily updated our traffic limits by 10x. Which is more than we would expect to see even if there were an event to occur.

# The Explanation

There were _several_ times where I needed to explain whath appened to a variety of stakeholders of varying technical knowledge. My original explanation went something like this:

---

Our SharePoint configuration was set to only allow a queue of 5 000 requests at a time.  After our queue hits 5 000 requests SharePoint starts dropping requests to protect the hardware.  The problem we had was that we could handle significantly more than 5 000 incoming requests so we upped it to 500 000 which we have tested in our pre-production environment successfully.

---

This went OK for some people.  Either they understood the idea of our bucket filling up and spilling over - or it sounded like I knew what I was talking about and they went along with it.  I'll never know. There was one very senior individual that wanted an explanation and I tried to explain it with my aforementioned explanation to which she responded along the general lines of:

> "I'm not technical at all. Can you explain it differently?"

After a few seconds of brain racking (which felt like an hour) I had an idea.

---

Let's say that we were going to a concert.  We're going up to the door and people are being allowed in.  Once we get to the door the bouncer says "Nope, you're not allowed in. We're at capacity" and sends you home (This is the worlds rudest bouncer). When one person leaves the concert, the bouncer lets one more person in but no more than 5 000 people is allowed in the venue at any given time.

Now, this is fine and dandy.  The bouncer is turning people away so we don't go over capacity and possibly have a disaster with the fire department and get fined.  The problem was that the bouncer was working an event that's being held in a 500 000 person stadium.  Essentially we fired this bouncer and hired one with a little more knowledge of the capacity of our venue.

---

> "That makes way more sense. Thank you! So we can fit 500 000 people in our site?"

I was pretty happy with this explanation and it got the point across.  As one of those tech folks who only works and talks with tech folks, I often forget that ideas of queues and throttling falls on deaf ears for the normal people of the world.

Hopefully this helps you think of new ways to explain your technological issues **** without making people go cross-eyed :). Most of us are here to help our businesses function.  First we need to get people in the same book - then we can get them on the same page.  That's how we can accomplish things together.





