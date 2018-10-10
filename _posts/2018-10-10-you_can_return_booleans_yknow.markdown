---
layout: post
title:      "You can return Booleans, y'know!"
date:       2018-10-10 12:13:58 -0400
permalink:  you_can_return_booleans_yknow
---


As I move through the ruby labs, I'm frequently seeing things like:
```
if some_method_returns?(true)
 true
else
 false
end
```

or 

```
if x < y
 true
else
 false
end
```

Does this seem a little verbose to you? A little redundant? Good eye, you're right!

As I learned to code, something it took me a while to realize (because it often wasn't taught explicitly) was that even though we usually see things like `x<y` or `something == something_else` in the context of conditional statements (like `if`), they are simply values. 

`if` is simply saying `if true`and all you're doing is replacing `true` it something that will either *equal*  `true`, or won't.

Did I say something about verbose and redundant? Oops. A taste of my own medicine I suppose.

Here's all that I'm getting at. If you have a method that ends like this: 
```
if x < y
 return true
else
 return false
end
```
you can simply write  `return x<y` instead! 
	
To get a little more difficult, what about a Boolean method, like implementing your own version of `#all?` (that's the lesson that made me think of writing this, by the way!).

Flatiron's lesson gives the solution like this (spoiler alert!):

```
def my_all?(collection)
 i = 0
 block_return_values = []
 while i < collection.length
  block_return_values << yield(collection[i])
  i = i + 1
 end
 
# Here's the important part!
 if block_return_values.include?(false)
  false
 else
  true
 end
end
```

This is sort of the opposite of what we said before: the `if` part returns `false` while the `else` part returns `true`. So we can't just say  `return  block_return_values.include?(false)`.

Do we have to say ` `return block_return_values.include?(false) == false`? (remember, `block_return_values.include?(false) == false` is just a statement that evaluates to a boolean)

Well, we could, and that certainly saves a lot of space. But we could make it even shorter, if only we had something that was a shortcut for `x == false`. Like, the *opposite *of `x == true`...

<iframe src="https://giphy.com/embed/OlWMp4ZxTseBy" width="379" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/avatar-forehead-sokka-OlWMp4ZxTseBy">via GIPHY</a></p>

Oh yeah, we do have that! `!x` is the same as `x == false` (the opposite of `x == true`).

So we can say  `return !block_return_values.include?(false)`. Or if you want to be *slightly* clearer (i.e., more readable) you could wrap the value we're falsifying in parentheses:  `return !(block_return_values.include?(false))`

Phew, and I thought we were trying to be less verbose! I guess that's for my code, not for my thoughts.

Come back next time for more tips and hacks!

# > TL;DR
You can always replace
```
if true_statement == true 
 return true
else
 return false
end
```
with  `return true_statement`.
If you want 
```
if true_statement == true 
 return false
else
 return true
end
```
you can replace it with `return !true_statement`.

