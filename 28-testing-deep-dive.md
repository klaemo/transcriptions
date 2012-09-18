[28: A Testing Deep Dive Show](http://nodeup.com/twentyeight)
===

Panel:

* Daniel Shaw
* Nuno Job
* James Halliday
* Domenic Denicola
* Adam Christian

Table Of Contents:

1. [Introduction](#introduction)
2. [Uncaught Exceptions](#uncaught-exceptions)

## Introduction

**Daniel Shaw**: Welcome to NodeUp 28, a testing deep dive with myself, dshaw, Nuno Job, Domenic Denicola
and Adam Christian. We're gonna talk about testing today and hopefully get into some of the testing approaches
in Node and Nuno is gonna tell you all about specify and why you should do that. And we're gonna ignore him.

**Nuno Job**: Welcome, guys. Today we're gonna talk a lot about testing as Daniel just said. If you follow Node core
you know that most of Node core, if not everything, is tested as...each functionality, each test is a single js file.
The reason for that is that is isolates the test. It's super hard to have control over side-effects if it's not 
like that in Node.js right now. I can't speak for the guys at core, but that's why I think they use single js files for
that. In this show we're gonna talk a lot about the frameworks and the decisions that they make to enable you to have
seperate files that instead of encapsulating a single test a series of tests.
And we're also gonna talk about why [domains](http://nodejs.org/api/domain.html) are important. 
But we're not gonna go into a lot of detail. And what that means for all of you in the future. If don't know, 
domains is a feature that got introduced in node.js in 0.7 and they basically allow to put context in your 
node programms. Before, when you call a callback, node lost trackof where it was. It didn't have a stack trace and 
it couldn't know the context where that was fired. So, if something went wrong, like an uncaught exception, all 
you could do was catch it, but you couldn't really know which user that uncaught exception was from. 
So, domains fix that and that's gonna be very important going forward for  testing, because that's something we 
really need in testing if you want accurate counts and you wanna know why your program broke. 
So, I know that there's some people here, Domenic for instance is a contributor to [chai.js](http://chaijs.com/). 
Adam, who is from saucelabs wrote a lot of testing code. Jellyfish and a lot of other frameworks. I'm actually curious
what all of you guys think about uncaught exceptions and how you have handled it in your code so far.

## Uncaught Exceptions

**Domenic Denicola**: Yeah, I guess, I'll start. Domenic here. It's definitely a big problem. Usually my exceptions get
caught. So, that's good. I mean, I try to write really small, focused unit tests. So, usually you know exactly how it's
gonna break, but if something is going wrong, I have definitely spent many minutes going like "where is this uncaught 
exception coming from", you know? If I have written a test that integrates multiple levels itself, it's like four levels
deep and I dont' have any context. So, without domains, which none of the frameworks I used have, I can definitely see
the issue.

**Adam Christian**: So, we have a pretty interesing situation, which is that people sort of tend toward mixing unit tests
with functional tests, because they're used to sort of the way there unit test runner works. And so, when you get to a
large codebase that has lots and lots of tests this becomes an absolute desaster. Especially, when it's part of your
build process. So, somebody has to spend hours digging through something somebody else did trying to figure out where
this exception came from. So, that's why the people that are the most successful are the ones that break these down
to the smallest little encapsulated modules. That's what excites me about domains. And the node community is moving, 
I think, more in that direction, which is an exciting and definitely healthy thing.

**Daniel Shaw**: So, Adam, to have successful unit tests, do you have to have mocks?

**Adam Christian**: I think, absolutely. For me, with my last couple of projects, the most frustrating part is that there
wasn't a collection of awesome mocks for common stuff. I wanted to mock...that I could be sitting on an airplane working
on an integration with twitter. I want to just "npm twitter-mock", run the server and have that sitting there like it's
talking to twitter services, so that I can build those apps. And it needs to be reliable, because everytime they ship 
something, somebody needs to pay attention and update that, so that everybody else's tests aren't broken. So, I would say
mocks are one of the very key pieces to successful testing going forward.

**Domenic Denicola**: Yeah, I just really want to second that. Without mocks it's not a unit tests, it's an integrations test
and you get a whole different set of problems. I also wanna say it's been very frustrating not having good just basic things.
I think, there are some good FS mocks by now, but HTTP mocks I've had a lot of trouble finding. We end up just creating a stream
and saying like, oh, this is our response and this is our request and we'll just write to them. It's a lot of manual code, just
because I wanna create unit tests and not integration tests. It's so frustrating. At times, I'm like I should just fire up
an HTTP server and test that, because I'll write so much less testing code.

**Daniel Shaw**: So, let's give people some points of reference. So, you know, you say it would be really nice to have a
nice twitter mock. What other enviroments have that in place and what could we look to as inspiration?

<!-- Continue at 6:20! -->

