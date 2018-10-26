---
layout: post
title:      "Audiobook Now: My CLI Portfolio Project"
date:       2018-10-26 15:48:33 +0000
permalink:  audiobook_now_my_cli_portfolio_project
---


I'm a freelance musician, so I have a lot of different gigs. Many of them are one-off, or few rehearsals for a show, and I get by through whatever my commute might be by listening to podcasts ([MBMBaM](http://www.maximumfun.org/shows/my-brother-my-brother-and-me) is my current favorite). But when I have a regular gig, 5 days a week through rush hour traffic, or two gigs with my band in one weekend that are each 2 hours away (UGH that's next weekend) I need something that will hold my attention for longer with more continuity. 

### Like an audiobook!

And isn't it great how you can download audiobooks from the library for free from your computer (or phone or tablet)? Yes! It is! 

But the problem on a commute (or on a road trip) is that I have a set amount of time that I want to fill. I'd prefer not to wind up with several hours where I have to, god forbid, choose what else to listen to. And I certainly don't want to miss the end of the book because my trip is over. So I want to find a book that's the right length.

But....you can't. Overdrive, the system that libraries use for their e-catalog, will show you the length of a book once you click into that book's page, but as far as I can tell, you can't search or sort by length. So all you can do is do whatever sorting or searching or narrowing (by subject, for instance), and then click through each book that looks interesting and hope it's not twice, or half, as long as the time you're trying to fill.

### Until now! (well, sort of)

For my CLI Portfolio Project I decided to build a program that would sort audiobooks by length. 

Now, it doesn't actually do that yet (it will soon, though!). The requirements of the Flatiron project are to display a list of things (books in my case) and allow the user to go one level deep, selecting an item on the list to get more information about it.

## The project

Using the New York Public Library site, the first thing I did was save the ["Available Audiobooks"](https://nypl.overdrive.com/collection/26060) page. It's nice that the library has a dedicated page for audiobooks, and for available audiobooks. I saved this page to my `fixtures` directory (if alarm bells are tinkling in your head, you're right. just wait), and then clicked on a book and saved the book's page as well.

I created a `Scraper` class and wrote two class methods: `.scrape_book_list(url)` and `.scrape_book_page`. Both methods use `open-uri` and `nokogiri` to get the HTML and then find the appropriate tags for the info. Overdrive has most of the "lower-level" info (release date, duration, etc) tagged with something called `aria-label` in tags like `aria-label="Release date"`. I don't know what aria is, but that's what I used to find most of the info I needed.

The `Scraper` class methods create hashes that I then fed into `Book.create_from_hash(hash)` to create objects. `Book.initialize` takes a `title` and an `author` and an `available` Boolean property, which in the current implementation defaults to `true`. After all, the point is to search only available books, but I wanted to make this class flexible for other use cases. If a book is available it gets added to `@@available`, which is accessed by two redundant methods: `Book.available` and `Book.all` (the latter of which is technically "wrong" in a world where books might not be available, but is fine for now).

My `audiobook-now` executable (which I haven't been able to get to execute without calling `ruby audiobook-now`, I'm working with my section lead on figuring that out. I do have the shebang in there!) simply calls `CLI#run`. That method (in my `CLI` class) displays a welcome message, calls a `#get_books_from(url)` method that creates a list from `Scraper.scrape_book_list(url)`, maps the urls from the list, and then uses the urls to create `Book` objects using `Scraper.scrape_book_page(url)` and then `Book.create_from_hash(hash)`. `#run` then asks the user to select a book, shows info for that book, and asks if they want to see another book.

So you may have noticed something funny (there's a few things, but I'll only tackle the easiest right now). I'm scraping the book page before asking the user to pick a book! Yes, this is inconvenient, but if the whole point of the program is to find a book that's *the right length*, you need to see the lengths of the books right away. And once the program starts sorting books by length before even showing you a list, it will *definitely* need the lengths right away.

The big downside to this is that it takes a non-negligible amount of time to visit and scrape a book page (I guess it's just the loading of the page that takes time). And that can only be done once you scrape the list of books to get their urls, although that takes much less time. Once I loaded the program, it takes maybe 10-20 seconds for the list of books to load, which to an internet user feels like a lifetime. At first I displayed a message "Loading books. Sorry, this may take several seconds." But then I came up with something a *little* more fun: 

     book_urls.each{ |url| 
       hash = Scraper.scrape_book_page(url)
       Book.create_from_hash(hash)
       puts "#{Book.all.count}. #{Book.all.last.listing}"      
    }
		
`Book.create_from_hash(hash)` still takes a second or so (probably more like 0.6 seconds or something like that), but each book then gets displayed as soon as it's loaded/created. The user sees the list being assembled. Kinda fun, but more important, there's less time where nothing is happening.

So that's the program, right?!

## WRONG.

I wrote it all. It worked great. ***With the fixtures file***. I was using the fixture file for the list of books, although each book did load the actual webpage. However, when I switched the url to the live url, no books loaded.

Hm... I had been afraid, at the very start of the process, about all the book info on each book page not loading. When you go to the page for a book, you have to click "details" to get to the area where it displays the book's duration. I was afraid this wouldn't be loaded immediately. Scraping and searching showed me I was wrong. Phew. 

But I hadn't thought about the list of books itself. By the time I hit "Save" in Chrome (to create my fixture file) the page had loaded, completely. Javascript had populated the main content frame of the page. However, when Nokogiri grabs the page, it just grabs the initial code, before JS gets its hands on it. 

Try it yourself. Open irb and type

    require 'open-uri'
		require 'nokogiri'
		html = open("https://nypl.overdrive.com/collection/26060")
		doc = Nokogiri::HTML(html)
		File.open('book_scrape.html', 'w') { |file| file.write(doc) }
		open book_scrape.html
		
You'll see that the top and left frames (they're probably not called that 20-year-old term anymore!) are there, but the main content is empty.

This was the state of my app when I got on my call with my section lead, Kevin. He suggested `poltergeist` which uses `capybara` to get JS-generated html. Replacing my Nokogiri code in `Scraper.scrape_book_page(url)` with equivalent Poltergeist code – an easy task – fixed the whole process!

## Done!

For now. 

I tried making the program into a gem. Long story short, there are definitely some gaps in my understanding about how to do that. I'll get to it. Eventually.

But! I have those faraway gigs coming up next weekend! So the next step is to be able to actually sort by length. Upcoming steps include:

1. CLI and model code to select filters and sorting, mainly by subject (also by language), *before* the search happens. Since we take the time to visit and scrape *every* book page once we actually search, we want to narrow our actual search as much as possible before we search.
2. CLI and model for sorting and filtering by audiobook length!
3. Ultimately give (and hopefully execute) the url for the book you want to borrow.
4. Select and aggregate different libraries.


Stay tuned for the actual developed gem and get ready to have an easier time picking your next road trip audiobook!
		

