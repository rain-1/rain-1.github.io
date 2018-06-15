# What UNIX shell could have been

The shell (bash or whatever) is an excellent tool that saves people a huge amount of work. Being able to easily script complex jobs together is one of the best things. It does have some weaknesses though that I feel could be improved though.

The two biggest weaknesses in shell, in my opinion, are the quoting and escaping mess and secondly that all the objects are strings. I've talked about the quotation stuff before so I wont cover that here. My idea for improving it would be to make separate (dynamic) data types for strings, paths and command lines flags.

This could be implemented in a simple (but ugly) way by encoding each object into strings with a tag saying what type they are:

* `"sfoo"` for a string
* `"fhelp"` for a flag
* `"p/dev/random"` for a path

Maybe there's a more aesthetic way to do this, open to suggestions.

This change can't be done just by writing a new shell though. Every UNIX tool that we have (`ls`, `cat`, `grep`, `jq`, ...) would need to conform to this protocol. It would probably steamroll over 'dd' (which has an ideosyncratic argument style).

# Advantages and Drawbacks

What would be good about this idea? Command line tools would throw an error if given the wrong input, instead of what they currently do: attempt to continue with a misinterpreted string. This could be considered an improvement if you're interested in your scripts correctness. Issues like the recent GPG security problem CVE-2018-12020 would have been avoided.

But the drawback might be that some things take a bit more programming to achieve. For example you would have to explicitly convert a string to a path. You would have to build up paths and apply regex and things to them in a different way than you do it on strings. Maybe the language could be designed to make these things easier.

# The current mess

It's worth mentioning the current "solution". In the current system if something starts with a - it's a flag. This leaves the problem of filenames that start with -. They are very rare so it isn't a big problem: We only encounter it occasionally. Some might never encounter it.

Anyway the answer is that tools can provide the -- flag to say that the next argument (or all preceeding) are not flags.

You can read about that [here](https://www.mariocampos.io/blog/more-unix-tools-hints/)

We also have incredibly good documentation about working around the difficulties of coping with paths and spaces and stuff in shell:

* https://www.dwheeler.com/essays/fixing-unix-linux-filenames.html
* https://mywiki.wooledge.org/BashFAQ

Now these are excellent resources that I value, but I believe that it is incidental complexity. And a better designed shell would result in much less documentation and edge cases to worry about.

Some shells have added array data types [rc](http://doc.cat-v.org/plan_9/4th_edition/papers/rc), but these only work internally. Across process boundaries everything is squeezed through the string object.

# Summary

I believe we make all of our lives easier by improving the tools we use. Shell has a couple big problems that can actually be solved. There have been experiments to solve the quotation issues [execline](https://skarnet.org/software/execline/) [s](https://github.com/rain-1/s). And I am proposing the idea of a very basic dynamic type system to solve the 'in-band signalling' issues with command line args.

