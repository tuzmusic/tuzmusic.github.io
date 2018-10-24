---
layout: post
title:      "How to test #gets in rspec - Once and For All!!!"
date:       2018-10-24 20:23:17 +0000
permalink:  how_to_test_gets_in_rspec_-_once_and_for_all
---


I hate the internet.

Googling is the worst.

You get too much information and get exhausted and give up before you find the actual, say, recipe you're looking for (after two pages of rambling about how much the author's kids love this dish or why this vegetable in the dish is so good or good for you) or perhaps the instructions on how to change an A/C filter (after three paragraphs about "How did people stay cool before air conditioning?"

### And oh my god I'm doing it right now.

Sorry. 

I was trying to write a spec for a method that takes user input using `gets`. Googling was difficult. The official rspec documentation is...well, I'll be polite and choose to not say anything more on that subject. Various Stack Overflow questions didn't seem to address what I'm actually looking for. Also, `expect(this).to receive(:that).and_return("something")` can actually be used in a number of other ways.

As a lazy (efficient!) coder, rather than learning and understanding all of that, I finally managed to find the answer by looking back at the specs Flatiron provides in the Tic Tac Toe w/AI project.

## So here's the actual answer. TL;DR;HTI
#### (Too long. Didn't read. Hate the internet.)


    it 'gets the user\'s input' do
      obj = Your_Class.new # the instantiation isn't "important" here. 
      allow($stdout).to receive(:puts)  # Optional. This is just to hide your puts statements from the test output.
      allow(obj).to receive(:gets).and_return('the desired input') # THE SETUP
      expect(obj.method_that_calls_gets_and_returns_something).to eq('desired return value') # THE EXPECTATION
    end
		
The `allow(obj)...` line sets up two important things. 
1. The call to `gets` inside the method will be "received" by the `obj` (Honestly, I'm not 100% sure what this means)
2. The argument passed to `and_return` is what the test pretends to type when `gets` is called.

Then the last line (`expect...`) sets up the expectation normally as usual.

I hope this made any sense to you and is helpful for this bewildering topic. My brain is fried. Time for some coffee.

Happy coding!
