---
layout: post
title:      "Rails JS Project"
date:       2019-03-06 23:08:25 +0000
permalink:  rails_js_project
---


Well, again I've been working hard on this all day so the blog is the last step before I submit. So it won't be as exciting as my other posts may (or may not) have been.

I was happy to see that the Rails JS project was just to add JS dynamicism to the Rails project. So I opened up my Fares You Can Use Project and did the following:

* Did a bunch of serializing to make an API for Deals and Users, and later added posting APIs for user preference items.
* Deals Index page - call on the Deals API to render "All Deals", and the User API for "My Deals" to get all the current user's vacations, and the deals that matched those vacations. 
* Deals Show page - call on the Deals API to render a single deal. I wanted to add "next" and "previous" links but that was a real head-scratcher trying to figure out how I'd pass the info about the next and previous deals, since (1) the id's aren't necessarily consecutive and (2) we're not simply traversing the ids anyway, since the list of deals we're traversing is different depending on where we got to the show page from. So, I gave up on that for now.
* Preferences page: Dynamically add and delete home airports and preferences. 

I did a lot of Object Oriented JS for this one, which put me in my comfort zone having come from a Swift life. Deals, Vacations, and Airports all have classes, with methods that are primarily (perhaps exclusively) used for rendering bits of HTML, so that the JS methods that the buttons/links call on are very short, consisting usually of just an API call, iterating over the response, constructing Deal/Vacation/Airport objects, and calling a single method on the object to get the HTML. 
