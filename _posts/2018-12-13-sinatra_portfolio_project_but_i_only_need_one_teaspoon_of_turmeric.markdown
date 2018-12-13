---
layout: post
title:      "Sinatra Portfolio Project: "But I only need one teaspoon of turmeric!""
date:       2018-12-13 15:56:10 +0000
permalink:  sinatra_portfolio_project_but_i_only_need_one_teaspoon_of_turmeric
---


My Sinatra Portfolio Project comes from yet another brilliant idea from my wife, with her friend Lauren this time. 

When you're cooking, you sometimes need some mildly uncommon ingredient you don't have, an ingredient you know you won't use often. Let's use turmeric for example. (I put the accent on the first syllable, *tur*meric, or even *too*meric. Honestly not *actually* sure if this is correct, but I like it more than too*mer*ic). Your recipe will call for one teaspoon of turmeric, but you go to the store and they want you to pay $5 for about 10 times more than you need.

(find me an appropriate gif for this moment of frustration and I'll put it here. 50 fake bonus points)

Wouldn't it be great if you could just knock on your neighbor's door and just borrow a teaspoon of turmeric? The proverbial "cup of sugar" of connected times of old? Well we live in the 21st century so god forbid we knock on our neighbor's door and ask them for something face-to-face. Instead we will use the internet to see who has some turmeric and arrange to borrow it.

![](http://www.picslyrics.net/images/301/160747/thumb.jpg?1462698040)

As the [McElroy brothers](https://www.maximumfun.org/shows/my-brother-my-brother-and-me) would say, "tmtmtmtmtmtm"

My Sinatra app has a `User` model and an `Ingredient` model. A user has many ingredients and an ingredient belongs to a user. I use ActiveRecord for the database. In the app's initial form, an ingredient just has a name. 

My controller structure is a little different then the standard, but I'm hoping my reviewer will be okay with it. My `/` route is my welcome and login page, with a link to a separate `get '/signup'` route with its `signup.erb` page. 

Once logged in, the main home page is addressed via `get '/index'`, which currently shows 3 lists: your ingredients (with links to add and edit your ingredients), all users, and all ingredients. Not necessarily the most useful collections of information. I think ultimately what I'd want, on a basic homepage, is a list of your ingredients, and a form to search for an ingredient you need.

The `get 'ingredients/new'` route opens the Add Ingredients page. Since you want to know what ingredients you already have, so as to not add them again, I've got a table on the Add Ingredients page, where the first column is ten slots for new ingredients, and the second column is your current ingredients. The `get 'ingredients/edit'` route shows all your current ingredients in text fields for editing, and each ingredient has a checkbox for deleting.

The multi-item create/edit route was different from anything we'd done before in lessons, but still pretty simple. Just wrap your creation in a loop.
```
params[:ingredients].each do |ingredient|
      next if ingredient[:name].empty?
      ingredient = Ingredient.create(ingredient)
      ingredient.user = user
      ingredient.save
    end
```
You'll notice the first line of the loop where if one of the ingredient slots is empty it just skips to the next one. I wanted to allow the user to save many ingredients at once, but not have to save all ten, so I didn't do any "no blank object" kind of validation here. Even if you leave all the fields blank, it just brings you back to the home page without saving any ingredients. I did use "no blanks allowed" validation on the edit page, where if you leave an existing ingredient blank, you get an error: `"You cannot leave an existing ingredient blank. To delete ingredients, use the checkboxes."`

A note about these error messages: Previous lessons talk about using "flash" messages. Flash messages are great, and they're simple. So simple, in fact, that I don't really understand why there are gems for them. As long as you're using sessions, all you have to do is assign a value to `session[:flash]` in your controller, and in your erb file (or your layout.erb file), put this code where you want the flash message to appear:
```
<% if session[:flash] %>
  <%= session[:flash] %>
  <% session.delete(:flash) %>
<% end %>
```

To round out the app, a user's show page lists that user's ingredients, and an ingredient's show page lists all the users with that ingredient. The ingredient show page is accessed with a slug, and the `get '/ingredients/#{:slug}'` This route first takes the slug and uses it to find an ingredient in the database. If one isn't found, it shows an error message and redirects to the home page. If it is found, we find the users with that ingredient, like so:
```
      @users_with = User.all.select do |user|
        ings = user.ingredients.map {|i| i.name.downcase}
        ings.include?(params[:slug].downcase)
      end
```

### Next steps:
The remaining requirement for the project is data validation. After I publish this post I'll be adding username/email uniqueness validation as well as email address validation. The project also requires that your data in the database is validated. I'm just going to make sure that a user can't input an ingredient they already have. Not very interesting, but then again I don't see anything else to validate. I don't see any reason to restrict a user from entering numbers or punctuation if they feel like adding "p@rs1ey" instead of "parsley." Although that would be a dumb thing for them to do because then no one can find it! 

Currently an ingredient belongs to a single user, in a many(ingredients)-to-one(user) relationship. Multiple users might have an ingredient called "parsley" but each of those parsleys are individual objects. A better way would be a many-to-many relationship where multiple users can have the same parsley object in their ingredients array. Listing users with an ingredient would then be as simple as a call to `ingredient.users`.

Furthermore, a user might specify how much of an ingredient they have, maybe with a few options such as 
```
amounts = [ {index: 0, description: 'a few teaspoons'},
              {index: 1, description: 'about half of a small bottle'},
              {index: 2, description: 'most of a small bottle'},
              {index: 3, description: 'more than a small bottle'}, ]
```
Then the user would have many `ingredient_relationship`s (or something like that) and each `ingredient_relationship` would belong to an `ingredient`, which would have many `ingredient_relationship`s. 

The search function, mentioned above, is interesting. The form could send `post /search` which would try to find an ingredient in the database. However, it could also simply send `get /ingredients/#{search_term}` which would call up an ingredient show page if one exists, and if that ingredient isn't in the database, it would just give a message saying so.
