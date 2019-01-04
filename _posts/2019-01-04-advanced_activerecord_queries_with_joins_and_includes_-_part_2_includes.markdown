---
layout: post
title:      "Advanced ActiveRecord Queries with #joins and #includes - Part 2: #includes"
date:       2019-01-04 17:01:08 +0000
permalink:  advanced_activerecord_queries_with_joins_and_includes_-_part_2_includes
---

## Captains and Boats and Classifications, oh my!

### Captain Spec

The `Captain` spec asks for methods like `Captain.sailors` which is all `Captain`s who have a `Boat` whose `Classification` is Sailboat. We are now one association deeper than we were with `Boat`. `Boat.joins(:classifications)` gave us `Boat`s with access to their `Classification`s and those `Classification`s' properties. But now we start at the `Captain`. Jeeeeezus.

Maybe we keep joining?

```
[25] pry(main)> Captain.joins(:boats).joins(:classifications)
=> #<Captain::ActiveRecord_Relation:0x3fc8e666a5c8>
```

Nope, empty. 

### The #includes method

Thanks again to AJ (or maybe it was another coach later in the day because this lab was so hard I needed two coaches a few hours apart) I looked at the ActiveRecord `includes` method. 

As a preamble to all of this, I want to put something in your head that is really important, and really easy to forget:
>The `#includes` method we're discussing here has ABSOLUTELY NOTHING to do with the `#include?` method we've used before. 

Each time you forget this, and you will (or at least I did), it will get ten times more difficult to try to understand `#includes` itself.

Preamble over. But keep remembering that. 

The Active Record docs for `includes` (which aren't even under a heading for `includes`) say this:

> Active Record lets you specify in advance all the associations that are going to be loaded. This is possible by specifying the `includes` method of the `Model.find` call. With `includes`, Active Record ensures that all of the specified associations are loaded using the minimum possible number of queries.

After all the work we did with `#joins`, that's actually not quite as impossible to understand as it was when I first read it yesterday. Although I still have no idea what they're talking about with the mention of the "`Model.find` call"

They mention, at the end, loading using the minimum number of queries. If you continue reading, they give examples of how using `includes` can greatly reduce the number of queries SQL is making to get the information you want. As I tried to understand why the lab solutions worked, I looked into all this SQL stuff and looked at what queries were executed doing their example without `includes` and with `includes`, and after running it about a half-dozen times I did indeed understand what was happening, and that's actually what I was initially planning on blogging about.

However, and it kind of makes me angry just to think about this right now, the examples they give in the docs show ONLY why `includes` improves performance. They do NOT focus on what `includes` allows you to actually *accomplish* that couldn't be accomplished *without* it.

Well, after all that rambling I did, here I am to save the day.

### The #includes method - actual information (not just a rant like above)

Looking at the section of the docs I quoted, we can see that `includes` "lets you specify in advance all the associations that are going to be loaded." Again, given the work we just did on `joins`, that's actually pretty clear. It lets us start with a `Captain` and wind up with information about its `Boat`s and its `Boat`s `Classification`s! 

```Captain.includes(boats: :classifications)```

Now, if you are not really sure why you don't get access to all those properties and associations in the first place, and why you need to ask for them special, it's probably because your thinking is still in the realm of a program where you define all your objects and their properties (which might be other objects with their own properties, etc) and everything is available and you can do `captian.boats.first.classifications.first` and you can do something like 
```
Captain.all.select do |cap|    # get all captains where...
  cap.boats.any? do |boat|  # any of their boats... 
    boat.classifications.any? do |cl| # have a classification...
      cl.name  == "Sailboat"   # called sailboat
    end
  end
end
```
THIS DOES IN FACT WORK WITH ACTIVERECORD!!! 

However, it doesn't pass the specs. The specs are written using `pluck` to get the names from the returned objects, and `pluck` only works on an ActiveRecord collection, **not** on an array, which is what the code above returns. This is the lesson's way to enforce that we are talking about server-side queries and not client-side searches. Or something like that. And I thought we were supposed to be learning about when to use Model class methods instead of instance methods. Don't get me started.


### The #includes method - ACTUAL information (not just ANOTHER rant like above)

SO, LET'S USE `includes` SHALL WE?!

```
captains_with_boats = Captain.includes(boats: :classifications) # gives us all Captains with boats, loaded with info about their boats and about the classifications.
captains_with_boats.count == Captain.count # true. it just so happens that every captain has a boat.

cappy = captains_with_boats.first
cappy.class # Captain
cappy.boats # this works to get the boats
```
So we can get boats this way. Let's compare this with `Captain.joins(:boats)` which also allows us to get the boats. 
```
cap_inc = Captain.includes(boats: :classifications)  # cap_inc is easier to type than captains_with_boats
cap_join = Captain.joins(:boats)

cap_inc.count # 6
cap_join.count # 10
```
Okay, so this `includes` "table" is like our grouped table yesterday using `joins`.
```
cap_join_by_cap = cap_join.group("captains.id") 
cap_join_by_cap.count # 6
cap_inc.count # 6 (as a reminder)
```
Are those the same thing, then?
```
Captain.includes(boats: :classifications) == Captain.joins(:boats).group("captains.id") 
# false
```
BUT!

```
[9] pry(main)> cap_join_by_cap.each.with_index{|c,i| puts c == cap_inc[i]}
true
true
true
true
true
true
```
In case you missed it, we just compared each corresponding item of each collection, and saw they were equal. 

So they appear to actually be the same thing. That's interesting, I guess. But we haven't addressed `classifications` in the statement `Captain.includes(boats: :classifications)`. If we remove it, we're just dealing with `boats`. Can we use this in our `Boat.with_three_classifications` test?

```
 Boat.includes(:classifications).having("COUNT(classifications.id)=3")
  Boat Load (0.3ms)  SELECT "boats".* FROM "boats" HAVING COUNT(classifications.id)=3
SQLite3::SQLException: a GROUP BY clause is required before HAVING: SELECT "boats".* FROM "boats" HAVING COUNT(classifications.id)=3
```

Oh right, that thing about `having` and `group by` and all that SQL stuff. Blech. `Boat.sailboats` doesn't work either, but I may just be doing something wrong. 

---

Moving on (or back) to the problem at hand. What we really want is `Classifications`. Well guess what:
```
cappy = Captain.includes(boats: :classifications).first
cappy.boats.first.classifications # this works to get the boats' classifications!
```

So now it's sort of easy. If you know the syntax to query on a result like this. Which I do...becase it was in the solution. 😇
```
Captain.includes(boats: :classifications).where(classifications: {name: "Sailboat"})
```

Ta-da! I'm tired of writing. I hope this all made sense and came together! Happy coding!
