---
layout: post
title:      "Start your Flatiron projects quicker!"
date:       2018-12-16 19:01:50 +0000
permalink:  start_your_flatiron_projects_quicker
---


## Automate Yoself!

Ever since I switched from the Learn IDE to my local IDE, VSCode, I've been slowly getting tired of the repetitive sequence required for starting each lab:
1. Click the github button on the lesson page
2. Click fork (wait)
3. Copy the ssh url
4. Hop into terminal, paste the url and clone the repo (`git clone git@github.com:repo.url`)
5. `cd` into the new directory
6. Install dependencies with `bundle install` 
7. Type `code .` to open the folder in VSCode.

That's a lot! So I decided to automate it. 

Terminal is just a console, similar to `irb` or your `pry` console or `rails console`, using the BASH language instead of ruby. So you can write programs (scripts) to execute all your steps and just run the program and you're done!

Bash has its own idiosyncracies and commands to learn. The good news is you already know most of what you need, and the simplest script consists simply of the lines of code you already type in.

## Bash Aliases

You can create a bash script (extension `.sh` for "shell script") and then run it with `bash script-name.sh`. Or to make things way easier you can modify your system Bash files (add to, but don't change anything that's already there!), which live in the home/root `~` folder. In `~/.bash_profile` you can define one-line aliases like this one:

```alias path='PS1="\W$ "'```

Which reduces the prompt to the name of the current directory followed by a dollar sign when you simply type `path` into the terminal. You can substitute any character (or emoji!) you'd like for that dollar sign. I've noticed that one of the Flatiron lecturers uses a heart emoji. Pretty cool.

Once you edit and save your `~/.bash_profile` file, you'll need to either reload your terminal window, or, more easily, enter `source ~/.bash_profile` which will start using the newly saved file. Then you can type `path`, or whatever new alias you've created. Your life is now 12% easier. You're welcome.

## Bash Scripts and Functions
When you're ready to get a little more complicated and write multi-line scripts, open up `~/.bashrc` to define a function. Use the syntax:

```function_name() {
    # code
}```

A few idiosyncracies to note: Unlike some languages, you must have the brackets on their own lines as above,  you can't write `function_name() { # code }`, although that would be silly anyway because you could just write an alias.

Also, your functions can take arguments, but they're not declared in the function declaration. Any arguments you include when you type the command into terminal (or in another bash script, which, remember, is the same thing!) will be passed like so:
For `git clone git@github.com:repo.url`, `git` is the function, `clone` is the first argument and is usable in the code as `$1`, and the url is the second argument, accessible in the code as `$2`. Further arguments would of course be `$3`, `$4`, and so on.

Again, remember that you don't wite `function_name($1, $2)` etc.

Another important bash thing is that spaces are IMPORTANT. They separate different commands. This means that when you assign a variable, for instance, you can't write `this = "that"` but instead you must write `this="that"`. Variables are then accessed with `$variable_name` syntax, so to get to the `this` variable assigned above, you'd use `$this`

## Let's fork!

You know what, that's enough explanation. Here's the script to clone and fork a repo from its http url. 

Here's the contents of a file you should save somewhere as `learn-fork.sh`

```
#!/bin/env bash

# assign descriptive names for the arguments
folder=$1 
url=$2

# construct the ssh url
replace="https://github.com/"
with="git@github.com:"
git_ssh=${url/$replace/$with}.git

# navigate to the folder from which you called the script
cd $folder
# clone the repo
git clone $git_ssh
# navigate to the repo folder (the code in quotes gets the most recently modified folder)
cd  "$(\ls -1dt ./*/ | head -n 1)" 
# fork the repo
hub fork
# install the dependencies
bundle install
```

Then, in your `~/.bashrc` file, paste these two functions:
```
learn-fork()  { 
  # replace this path with the path to your script above
  bash /Users/TuzsNewMacBook/Development/code/Bash/learn-fork/learn-fork.sh $(pwd) $1
  cd ..
}

learn-fork-open()  { 
  # replace this path with the path to your script above
  bash /Users/TuzsNewMacBook/Development/code/Bash/learn-fork/learn-fork.sh $(pwd) $1
	# replace 'code' with the command for your preferred editor, like 'sublime' or 'atom'.
  code .
}
```

**I KNOW YOU'RE EXCITED, BUT BEFORE YOU USE ANY OF THESE YOU NEED TO INSTALL THE `hub` PACKAGE FROM HOMEBREW. SEE BELOW.**

You will then execute either of these functions from anywhere in terminal by typing `learn-fork http://www.github.com/repo-url` or  `learn-fork-open http://www.github.com/repo-url`
 
Notice that they both call the script we wrote in the earlier step, and pass it two arguments. The first argument, `$(pwd)`, is the current directory. Calling the `learn-fork.sh` file will switch our current directory to the one where you saved the script, so we pass in the current directory so that the script can run the `cd $folder` line to put the repo in the correct place. We also pass in `$1` which is the repo's url that you included when you called the function.

Use `learn-fork-open` when you want to work on the project right away. Use `learn-fork` to save the forked-repo and continue from the folder you started from. 

## Important Prep Steps

1. Install the github command line tools by typing `brew install hub`. This uses the Homebrew installer. If you don't have that already, Google how to get it. Warning that it will take a while to update Homebrew, which is annoying.
2. Go to your Flatiron lesson page and right-click on the work "Fork" in "Fork this Lesson" and copy the link. 
3. Paste the link as your argument after `learn-fork` or `learn-fork-open`.
4. The first time you run the `hub` tool, you'll have to enter your github login info.

## The Takeaway
I hope this has helped make your forking life easier and inspired you to write your own scripts!

Another thing I hope you'll take away from this post is that you shouldn't be afraid to dive in when inspiration strikes! There's a **LOT** I don't know about the Bash language. I barely know any of it. I Googled questions to figure out how to write almost every line of this code (I simply copied and pasted the line about finding the most recently modified directory). I still don't understand how to use `if` statements, which would be required to start making anything more complex (like any kind of error checking at all in this script). 

But I wrote this small and very useful script! It's fun to create things!

Happy coding!
