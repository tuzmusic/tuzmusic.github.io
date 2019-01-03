---
layout: post
title:      "Advanced ActiveRecord Queries with #joins and #includes - Part 1: #joins"
date:       2019-01-03 23:05:53 +0000
permalink:  advanced_activerecord_queries_with_joins_and_includes_-_part_1_joins
---

## Captains and Boats and Classifications, oh my!

I just finished the Model Class Methods Lab. IT WAS THE WORST. It took many hours and TWO Learn coach screen-shares, and ultimately I used the solution branch. So you might say I cheated.

But, I didn't submit the lesson until I actually understood what all the code was doing. All the `captain_spec` tests hinge on using the ActiveRecord `#include` method, which is EXTREMELY difficult to understand, and the fact that it appears to resemble the familiar `#include?` method but does something COMPLETELY DIFFERENT doesn't help.

Now that I've finished this insane lab, I'm going to treat myself to lunch. And then I'll come back and explain `#include` so maybe this lab will only take one hour for you instead of four.

## TL;DR
>`FirstThing.joins(:other_things)` basically represents the `first_thing_other_things` join table. It returns `FirstThing` objects with access to their `other_things` collection, and that collection's objects, and those objects' properties. To find a `FirstThing` with an `OtherThing` named "Omelette", you'd ask for: `FirstThing.joins(:other_things).where("other_things.name=?","Omelette")`
>
>To get any aggregate properties of the `FirstThing.other_things` collection, group the join collection above by a column in the `first_things` table: `FirstThing.joins(:other_things).group("first_things.id")` 

---

Okay, back from lunch. Let's get into this lab.

As a refresher, we're talking about boats and their captains, and the relevant aspects of the models are:
* `Boat`: `belongs_to` a `Captain`
* `Captain`: `has_many` `Boat`s
* `Boat`:  `has_many` `Classification`s
* `Classification`: `has_many` `Boat`s. 

So the tables are `boats`, `captains`, `classifications` and `boat_classifications`. We're just going to pay attention to the hard tests.

### Boat Specs
`Boat.sailboats` is supposed to return all boats that are sailboats. That sounds like a simple relationship to us humans, but to a computer the full relationship chain must be traversed, and that looks something like this:  
1. All `Boat` objects...
2. whose `classifications` collection contains...
3. an object...
4. whose `name` property is Sailboat. 

Off the top of your head it might seem like `Boat.where(classification.name: "Sailboat")` will do the trick, but a `Boat` has many `Classifications`, so `classification.name` isn't even a thing.

Okay, so your next thought might be `Boat.where(classifications.map{|c| c.name}.include? "Sailboat")`. That looks really good to me, but it doesn't work:
```
NameError: undefined local variable or method `classifications' for main:Object
```

I'm going to be honest with you, I don't know _precisely_ what this means, but it obviously doesn't work. The moral of this part of the story is that even though you can query `Boat.first.classifications` and get a happy collection of `Classifications`, you can't get to the properties of an object's relationships from `where`. ("You can't get there from `where`.") 

So while you can pass the `self.dinghy` spec with `Boat.where("length<?", 20)`, that's simply querying a property's value, whereas something like `Boat.where(classifications.first.name: "Sailboat")` or even `Boat.where('classifications.first.name = ?',  "Sailboat")` isn't allowed.

### The #joins method
Here's where it gets hard, because this is a case in Ruby/Rails/ActiveRecord where the syntax doesn't flow linguistically and intuitively in the way it often does. It just doesn't read like a sentence. And the docs aren't very much help in this case:

> 12.1.2.1 Joining a Single Association
> 
>     Category.joins(:articles)
> 
> This produces:
> 
>     SELECT categories.* FROM categories
> 
>       INNER JOIN articles ON articles.category_id = categories.id
> 
> Or, in English: "return a Category object for all categories with articles". Category.joins(:articles)

That last line is sort of helpful. "All categories with articles." Actually as I type it again, maybe it's not that helpful. Maybe I've read that line too many times. The moral is that `Category.joins(:articles)` basically gives you a fleshed out version of the `category_articles` tables, based on the categories.

I checked this to make sure I was understanding it correctly: in the lab, I queried `Boat.joins(:classifications).count` I get the same number of rows as are in the `boat_classifications` table when I inspect it using DB Browser. 

Where the `boat_classifications` table just has a bunch of `id` columns, `Boat.joins(:classifications)` generates the following SQL:

```SELECT "boats".* FROM "boats" INNER JOIN "boat_classifications" ON "boat_classifications"."boat_id" = "boats"."id" INNER JOIN "classifications" ON "classifications"."id" = "boat_classifications"."classification_id"```

And if you put this into a DB viewer and change the columns you're asking for...

```SELECT "boats"."name", "classifications"."name" FROM "boats" INNER JOIN "boat_classifications" ON "boat_classifications"."boat_id" = "boats"."id" INNER JOIN "classifications" ON "classifications"."id" = "boat_classifications"."classification_id"```

You get a table where each row displays the boat name and classification name for each relationship. If a boat has 3 classifications, there will be 3 rows with that boat's name.

**NOW** you can query information about the `classification` because you have access to those objects. It *seems* like you have access to each boat's classifications, as if each row was a boat and one of its columns had all its classifications, and you were querying that classifications column. But in fact you just have rows of relationships, and you can actually say: 

```
Boat.joins(:classifications).where("classifications.name=?","Sailboat")
```

And it will find each row where the classification name column is "Sailboat", and give you the name of the boat in that row.

---

Deep breath. Because it's actually not exactly that simple, in terms of actually comprehending what's going on. Let's look at that last line of the docs, replacing its example of `categories` and `articles` with ours:

>"return a Boat object for all boats with classifications". Boat.joins(:classifications)

The objects in the collection returned by `Boat.joins(:classifications)` actually appear to be strange multi-dimensional creatures. Let's make our display more manageable by getting just the first two boats in this join table creature:

```
boats = Boat.joins(:classifications).limit(2)
```

If you've just typed this into your rails console (which you should; and of course your database should be previously seeded with `rake db:seed`) ignore the output of this for a second. Take a look at `boats.first`:
```
[40] pry(main)> boats.first
=> #<Boat:0x00007f8d0ad9e3f8
 id: 1,
 name: "H 28",
 length: 27,
 captain_id: 1,
 created_at: Thu, 03 Jan 2019 14:59:25 UTC +00:00,
 updated_at: Thu, 03 Jan 2019 14:59:25 UTC +00:00>
 ```
 (by the way, you see a `pry` prompt in my Rails because I'm using the `pry-rails` gem, which uses Pry for your Rails console, which formats your output about 1000 times more nicely than the normal Rails console. I highly recommend it. Best thing ever.)

 Looks like a `Boat`. Since we've gotten to it from `Boat.joins(:classifications)`, we can actually get its classifications:
 ```
 [41] pry(main)> boats.first.classifications
  Classification Load (0.2ms)  SELECT "classifications".* FROM "classifications" INNER JOIN "boat_classifications" ON "classifications"."id" = "boat_classifications"."classification_id" WHERE "boat_classifications"."boat_id" = ?  [["boat_id", 1]]
=> [#<Classification:0x00007f8d0d019ce8
  id: 1,
  name: "Ketch",
  created_at: Thu, 03 Jan 2019 14:59:25 UTC +00:00,
  updated_at: Thu, 03 Jan 2019 14:59:25 UTC +00:00>,
 #<Classification:0x00007f8d0d019568
  id: 2,
  name: "Sailboat",
  created_at: Thu, 03 Jan 2019 14:59:25 UTC +00:00,
  updated_at: Thu, 03 Jan 2019 14:59:25 UTC +00:00>]
```

But if you look at the output from `boats` (which I told you to ignore before), you'll see that both objects returned from `Boat.joins(:classifications).limit(2)` appear to be the same `Boat`. They have the same properties. But! Their memory addresses (that big weird number that starts with 0x0000) are different. They are different objects. They are each a relationship between a `Boat` and a `Classification`.

Now this is where the firmament of my reality starts to break apart a bit, because I decided to check out ` boats.first.class`, expecting it to be some kind of `ActiveRecord_Relationship` or something. But it's just a fucking `Boat`. Excuse my language. 

So, I don't 100% understand it, but basically I'd say it like this: `Boats.joins(:classifications)` returns `Boat` objects that *have access to* the `Classification`s they're related to. The collection that returns has one row (object) for each *relationship*, but, again, that row is "filled" with a `Boat`. You might say it as `Boats.joins(:classifications)` gives you `Boats` *as they relate to* `Classifications`. Or you might not say it that way. Who knows.

So this is what you do when you want to wind up with a `Boat` by way of something about `Classifications`. If you wanted it the other way around you would ask for `Classification.joins(:boats)` and wind up with the same relationships but from the other perpsective: rows "filled" with `Classification` objects that have access to the `Boat`s they're related to.

---

Oh my god, so you finished that ONE SPEC. Don't celebrate just yet, because the last `Boat` spec is to find boats that have 3 classifications. 

Again, you might think it doesn't sound too hard. Try it out, something like...

```
[52] pry(main)> Boat.joins(:classifications).where(classifications.count==3)
NameError: undefined local variable or method `classifications' for main:Object

[53] pry(main)> Boat.joins(:classifications).where('classifications.count=3')
  Boat Load (0.8ms)  SELECT "boats".* FROM "boats" INNER JOIN "boat_classifications" ON "boat_classifications"."boat_id" = "boats"."id" INNER JOIN "classifications" ON "classifications"."id" = "boat_classifications"."classification_id" WHERE (classifications.count=3)
SQLite3::SQLException: no such column: classifications.count
```

IS NOTHING EASY IN THIS LIFE?!?!?

Okay, the second error makes a little more sense, because at least you UNDERSTAND that there is indeed no such column as `classifications.count`. So how the hell do we figure this out? 

Well remember before (all those paragraphs ago) where we noticed that our `Boat.joins(:classifications)` table had individual rows for each relationship, but each row looked like the same `Boat` object? Actually, to quote myself:

>It *seems* like you have access to each boat's classifications, as if each row was a boat and one of its columns had all its classifications, and you were querying that classifications column.

That's actually what we want to do here! I'm going to make the disclaimer now that most of the steps that follow from here were guided pretty strongly by the technical coach who got on the line with me, AJ (thanks AJ!). 

To get that "one row per boat" table we need to take our joins table and group it by boat:

```Boat.joins(:classifications).group("boats.id")```

Will give us just that table. It should have one row for each boat, so let's check `Boat.count`, and we see that (in our lab's seeded database) it's 12, and (because there happen to be no boats without any classifications), let's check `Boat.joins(:classifications).group("boats.id").count`:

```=> {1=>2, 2=>3, 3=>2, 4=>3, 5=>1, 6=>2, 7=>2, 8=>2, 9=>2, 10=>3, 11=>2, 12=>2}```

WHAT IS THAT?! It took me a few times to understand this hash, and it's still crazy to me that `.count` doesn't give the number of "rows in this table". But, it doesn't. I don't really know how they invaded my mind, but that has actually represents exactly what we're looking for. There are indeed 12 rows in this table (as we were hoping), but the hash is giving us one key-value pair per row, where somehow the value is the number of classifications!

I don't get it. But, there it is.

Unfortunately, we're not going to use that hash directly. Since this came, again, from a strong suggestion from AJ (rather than me completely figuring it out) I will just tell you that we can get boats with 3 classifications like this:

```
Boat.joins(:classifications).group("boats.id").having("COUNT(classifications.id)=3")
```

I'll break that down just a bit:

```
boats_and_classifications         = Boat.joins(:classifications)
boats_by_classification           = boats_and_classifications.group("boats.id")
boats_with_three_classifications  = boats_by_classification.having("COUNT(classifications.id)=3")
```

It actually starts to make sense, except maybe for that SQL at the end, if, like me, you're not all that fluent in SQL.

---

If I live to tomorrow, I'll be back to talk about `#includes` which is what you need to solve the `Captain` spec. It's even more linguisitcally confusing than `joins`, but I think the work we did here will have us in good shape to understand it. 

See you then, and happy coding!
