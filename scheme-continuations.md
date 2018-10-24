# Are first class continuations a language-design dead end?

## Against call/cc

I think the majority opinion of programmers on first class continuations like `call-with-current-continuation` is just that they're "scary". The people who are more familiar with them might notice that they never actually use them in real programming.

I think it's worth mentioning early on that continuations and continuation passing style have been proven to be extremely useful tools in theory of programming languages. In this note I'm only focusing on the first class continuation operators as part of programming language design.

So at some point `call/cc` was added to the scheme language. Then people realized that they couldn't even safely open and close a file with a computation in between. A continuation could jump outside from the computation leaving it unclosed forever or cause it to be opened multiple times.

To address this problem `dynamic-wind` was created, a new form that has a start and finish thunk which will always be executed, even if continuations are involved. This seems to me a real "Now you have 2 problems moment". What was the great benefit provided by adding `call/cc` that mandated adding a new complex language feature? I don't think the inclusion of `call/cc` in the language really has any merit.

## Introducing multi-prompt delimited continuations

A much more sensible control operator is `shift`/`reset` which together delimit a continuation segment. These have been called "composable continuations" but I will argue that this is actually a misnomer. Oleg's webpage has the most thorough and well argued explanation of why delimited continuations are more natural and useful than `call/cc` so I won't repeat any of that stuff.

Delimited continuations may be used to implement various interesting language features like exception throwing and catching, generators that let you yield sequence elements, dynamic binding/special parameter variables, and many others including anything that can be captured as a monad. But you can only implement *one* of these features. If you implement two of these and try to use them together they will clash because the reset prompts get conflated. This is why I wouldn't call them composable continuations.

The solution to this is the multi-prompt delimited continuations. Instead of reset/shift we have reset-at/shift-at that take an extra prompt argument, allowing you to match them up. This lets us implement exception throwing and catching of multiple different kinds of exceptions in a way that interacts correctly with dynamic binding and other special language features that have been powered by continuations.

Many schemes do support these MPDC (multi-prompt delimited continuations) operators - they still need to implement dynamic-wind as a primitive feature though. Guile and Racket are two examples, guile implements its exceptions with these continuations as well as it's fibers green threading system. Racket also implements some of its features in terms of continuations (generators and exceptions for example).

So the MPDC operators can be used to implement several fundamental language constructs. They can also be used directly in some very specialized programming tasks like partial-evaluation, but their use is very rare. A question worth asking is, is it better to implement MPDC and then exceptions and so on using that. Or just design and implement a language with exceptions and all the features that you want without also adding continuation operators?

## Implementing multi-prompt delimited continuations

To address the question we need to think about how to implement these. There is a very clear and easy to understand paper on implementing a virtual machine for executing lambda calculus with single-prompt shift/reset. But I haven't found anything similar for the multi-prompt versions. Perhaps it is pretty simple to extend that work to multiple prompts but I don't really know how to do it myself after studying the related literature.

Ocaml shift/delimcc implements multiple-prompt delimited continuations in terms of the "scAPI" invented by Oleg which piggybacks on the exception frame mechanism of Ocaml. I believe this code has also been adapted to other systems like scala and javascript. There has also been the OchaCaml work, to provide direct implementations of single-prompt shift reset, but it has been shown that this plus mutable cells enables you to build back up to multiple-prompts.

A new alternative approach to supporting all these different kinds of language features in a way that cooperates with itself is to use extensible effects/algebraic effects and handlers approach. I have seen that it allows one to implement multi-prompt delimited continuations in a few lines of code. So it seems somewhat equivalent but I am not an expert in this domain.

## Conclusions

Overall I would say that we took a wrong turn long ago with `call/cc` (and even `shift`/`reset`): There is a good reason nobody is using these continuation operators - they aren't useful and they don't work together correctly. The motivations for multi-prompt continuations haven't really been disseminated widely despite the core of languages like racket and guile being based on them. So programming languages like python and nodejs which have been gradually adding features like yield haven't been able to benefit from the theory and unified platform that these features can all be placed upon. As for the control operators themselves: Do we really need them exposed to the programmer at all? I really only believe that their uses are to implement more human-understandable control operators - and I kind of find it hard to believe that there are any left that we don't already know about.
