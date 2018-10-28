---
layout: post
title:      "Update: rspec with puts and gets"
date:       2018-10-28 21:38:22 +0000
permalink:  update_rspec_with_puts_and_gets
---


I'm pretty sure I've got it figured out:

```
 it 'lets a user select what they want to filter by (subject, time, language)' do
      cli = CLI.new
      allow($stdout).to receive(:puts)
      expect($stdout).to receive(:puts).with(%(Enter the number of the filter you'd like to change.))
      
      allow(cli).to receive(:gets).and_return('1') 
      expect(cli.ask_for_filter_number).to equal(:subjects)     
      allow(cli).to receive(:gets).and_return('4')      
      expect(cli.ask_for_filter_number).to equal(:date_added)     
 end    
```

With the beauty of rspec's readability this is actually kind of self-explanatory.

Taken slightly out of order, the actual execution takes place in `expect(cli.ask_for_filter_number).to equal(:subjects)` which runs `cli.ask_for_filter_number`.
`expect($stdout).to receive(:puts).with(%(Enter the number of the filter you'd like to change.))` tells the test to expect `puts` to be called with the specified output.

`allow(cli).to receive(:gets).and_return('1')` "pretends" to type "1" in response to gets and then
`expect(cli.ask_for_filter_number).to equal(:subjects)` tells the test to expect that the call to `cli.ask_for_filter_number` – a function that calls `gets` – will return `:subjects`.

Pretty simple actually. Just gotta remember it. Thank god for copy and paste.

Note: I think an easy confusion point is the word "return" in `allow(cli).to receive(:gets).and_return('4')`. It's not talking about what your function will return, it's specifying what the test enters into `gets`.

Okay, now you!


