---
layout: post
title:      "Custom Flight Deals - My Rails Portfolio Project"
date:       2019-02-07 16:52:03 +0000
permalink:  custom_flight_deals_-_my_rails_portfolio_project
---


Okay, listen. I just finished my Rails site after 3 weeks of very hard work. So I'm going to make this blog post short.

I built a website to go with my wife's flight email list where she finds great airline deals. 

An admin can create and edit deals. Deals have a headline, description, links to the actual flight search where the deals can be found, start and end dates, and origin and destination airports. When a deal is created, the app looks at its destination airports to determine the deal's region, and assigns the deal's region property.

A user can visit their Preferences page and enter their home airports, their upcoming vacations dates, and the regions they're interested in flying to.

Then the user can visit the Deals page, or a nested Region Deals page (e.g., "Europe Deals") and see either all the deals in that category (or simple all deals at the main index), or they can choose to see only those deals that match their preferences, i.e., deals that are

1. From their home airport
2. During their vacations
3. To their chosen destination regions.

Some of the MANY challenges I encountered were:
* Importing Airports from a gem (converting from JSON to AR objects), and using a Continents gem to assign regions to each airport
* Scope methods to filter deals by origin airport, destination region, vacation span.
* Putting multiple forms on the preferences page
* Other things I was thinking of 2 minutes ago but have since forgotten.

There's quite a bit of work to be done before the site goes live, but I'm excited at the prospect of using one my Flatiron projects in the real world when it's ready.
