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
3. [Mocks](#mocks)

## Introduction

**Daniel Shaw**: Welcome to NodeUp 28, a testing deep dive with myself, dshaw, Nuno Job, Domenic Denicola
and Adam Christian. We're gonna talk about testing today and hopefully get into some of the testing approaches
in Node and Nuno is gonna tell you all about [specify](https://github.com/dscape/specify) and why you should do that. 
And we're gonna ignore him.

**Nuno Job**: Welcome, guys. Today we're gonna talk a lot about testing as Daniel just said. If you follow Node core
you know that most of Node core, if not everything, is tested as...each functionality, each test is a single js file.
The reason for that is that it isolates the test. It's super hard to have control over side-effects if it's not 
like that in Node.js right now. I can't speak for the guys at core, but that's why I think they use single js files for
that. In this show we're gonna talk a lot about the frameworks and the decisions that they make to enable you to have
seperate files that instead of encapsulating a single test, encapsulate a series of tests.
And we're also gonna talk about why [domains](http://nodejs.org/api/domain.html) are important. 
But we're not gonna go into a lot of detail. And what that means for all of you in the future. If you don't know, 
[domains](http://nodejs.org/api/domain.html) is a feature that got introduced in node.js in 0.7 and they basically allow 
you to put context in your node programs. Before, when you call a callback, node lost track of where it was. It didn't 
have a stack trace and it couldn't know the context where that was fired. So, if something went wrong, like an uncaught 
exception, all you could do is catch an uncaught exception, but you couldn't really know which user that uncaught exception was from. 
So, [domains](http://nodejs.org/api/domain.html) fix that and that's gonna be very important going forward for  testing, 
because that's something we  really need in testing if you want accurate counts and you wanna know why your program broke. 
So, I know that there's some people here, Domenic for instance is a contributor to [chai.js](http://chaijs.com/). 
Adam, who is from Saucelabs wrote a lot of testing code. He wrote jellyfish and a lot of other frameworks. I'm actually 
curious what all of you guys think about uncaught exceptions and how you have handled it in your code so far.

## Uncaught Exceptions

**Domenic Denicola**: Yeah, I guess, I'll start. Domenic here. It's definitely a big problem. Usually my exceptions get
caught. So, that's good. I mean, I try to write really small, focused unit tests. So, usually you know exactly how it's
gonna break, but if something is going wrong, I have definitely spent many minutes going like "where is this uncaught 
exception coming from?", you know? If I have written a test that integrates multiple levels itself, it's like four levels
deep and I dont' have any context. So, without [domains](http://nodejs.org/api/domain.html), which none of the frameworks 
I used have, I can definitely see the issue.

**Adam Christian**: So, we have a pretty interesting situation, which is that people sort of tend toward mixing unit tests
with functional tests, because they're used to sort of the way there unit test runner works. And so, when you get to a
large codebase that has lots and lots of tests this becomes an absolute desaster. Especially, when it's part of your
build process. So, somebody has to spend hours digging through something somebody else did trying to figure out where
this exception came from. So, that's why the people that are the most successful are the ones that break these down
to the smallest little encapsulated modules. That's what excites me, obviously, about [domains](http://nodejs.org/api/domain.html). 
And the node community is moving, I think, more in that direction, which is an exciting and definitely healthy thing.

## Mocks

**Daniel Shaw**: So, Adam, to have successful unit tests, do you have to have mocks?

**Adam Christian**: I think, absolutely. For me, with my last couple of projects, the most frustrating part is that there
wasn't a collection of awesome mocks for common stuff. I wanted to mock...that I could be sitting on an airplane working
on an integration with twitter. I want to just "npm twitter-mock", run the server and have that sitting there like it's
talking to twitter services, so that I can build those apps. And it needs to be reliable, because everytime they ship 
something, somebody needs to be paying attention and update that, so that everybody else's tests aren't broken. So, I would say
that mocks are one of the very key pieces to successful testing going forward.

**Domenic Denicola**: Yeah, I just really want to second that. Without mocks it's not a unit tests, it's an integrations test
and you get a whole different set of problems. I also wanna say it's been very frustrating not having good just basic things.
I think, there are some good FS mocks by now, but HTTP mocks I've had a lot of trouble finding. We end up just creating a stream
and saying like, oh, this is our response and this is our request and we'll just write to them. It's a lot of manual code, just
because I wanna create unit tests and not integration tests. It's so frustrating. At times, I'm like I should just fire up
an HTTP server and test that, because I'll write so much less testing code.

**Daniel Shaw**: So, let's give people some points of reference. So, you know, you say it would be really nice to have a
nice twitter mock. What other enviroments have that in place and what could we look to as inspiration?

**Adam Christian**: Well, I have seen a handful of pretty interesting mock stuff in the Ruby community. I'm not a Ruby guy,
but I see people tweeting and blogging about that stuff. I don't know how deep the rabbit hole goes, but whoever starts on
a project that is a collection of commonly used APIs to mock. You know, you got your Twitters, and your Facebooks and your
Githubs, and your LinkedIns, blablabla... You know there is a handful of stuff that most people are using, especially for
auth. And if there were some really nice drop-in mocks of that, I think, that would get a lot of traffic and contributions.
That would be pretty cool to see.

(*skype ringtone, trying to get substack on the line ;)*)

**Domenic Denicola**: Yeah, I actually wrote up something a while ago that I still haven't implemented. I'll put it in the chat.
Just for my idea how a mock HTTP library would work. You just say: here is a mock server and here is a mock request and it should
give you this error or it should respond with a response that has this error or should respond with a string or it should respond
with some headers. Like, you could write these in one line and get a nice little mock request or response object, but I can't
do that write now.

**Nuno Job**: So, one thing that's...if you think about [Nodejitsu](http://nodejitsu.com), one thing that sets us apart from other
PaaS is that we can run on multiple infrastructures. So we have clients on Amazon, clients on Rackspace...we run our public cloud
on Joyent and it's blazing fast. And we have people running on their own servers. So, that means, we have a common API. You
can think of it like a translation which we call package cloud (?), that is kind of like libfog (?) in Ruby. It has the same
set of tests, but it needs to run agains Rackspace, it needs to run against Amazon, it needs to run against Joyent. I mean,
I'm talking about Solaris, Windows. Things that are completely different. We use [Nock](https://github.com/flatiron/nock) to do
that. It's super simple. Before, I can honestly say that it was a nightmare. If you think about it, spinning up server on
Rackspace from one 1min to 5min and sometimes in our tests we need to spin up 10. So, the tests used to run in something like
two or three hours and now they run within like 15 seconds with [Nock](https://github.com/flatiron/nock). It's super simple to do. It's just a JSON file, that says
it's a GET, you respond this, even if it's chunks. Whatever it is. It's really super simple. I feel like this problem is solved.
I just don't feel like, we, nodejitsu and Pedro [Texeira] who did [Nock](https://github.com/flatiron/nock) have advertised it enough. 
But there's a [nodetuts](http://nodetuts.com/) about it.

**Adam Christian**: The problem I'm talking about is not whether we have the technology to do it. It's people kind of sitting
down and getting together adn using the awesome community we have to build some of this stuff. Because for me, you can totally
build tests that ping these outside services, right? But what we're looking for is speed. People don't want their unit tests to
take 20 minutes, because they are waiting on requests. So, I don't see how anybody could really test their app without decent
mocks for all the different services that they are interacting with.

**Daniel Shaw**: Well, maybe as kind of product of this Podcast we can come up with a community whishlist and can see if we can
make this happen. That would be fun.

**Adam Christian**: That would be awesome. Totally. I'm happy to speardhead that as well.

**Daniel Shaw**: Let's put that in the show notes and make that happen.

**Domenic Denicola**: Actually, Nuno, do you wanna talk about [Nock](https://github.com/flatiron/nock)? Because I'd never heard 
about it and I'm looking at the readme and it looks amazing.

**Nuno Job**: So, [Nock](https://github.com/flatiron/nock) basically overrides the HTTP request and it does very little, but it 
allows you to...when someone does a native request in Node, [Nock](https://github.com/flatiron/nock) will intercept it and it keeps 
stack of routes that you can hit. Like GET /route and it returns a response for that. So, it's kind of DSL-ish, so you put like dot get 
this, respond with this and then in your application you do your normal requests. If you put Nock on, I normally set an environement 
variable and if I write  NOCK=on npm test it will just run the tests with [Nock](https://github.com/flatiron/nock). If I don't put that, 
it will do the integration tests. It's seamless. So for instance in [Nano](https://github.com/dscape/nano), which is a CouchDB driver 
I have, when I wanna do a new version, I do npm test and that's with mocks. When I'm doing from a contributor and I'm not sure about the 
code I usually run npm test without the mocks to make sure that things are running. So it's pretty seamless, it's very easy. It just have 
like the fixtures and for each .js test file I have the corresponding specification. That's it. It just runs and it intercepts. It's like 
one line of code I put there with this little helper. And that pretty much does the trick for me, does the trick for Nodejitsu when dealing 
with Rackspace, Amazon and Joyent. This is really complicated, I mean these are not the kind of...the Twitter API is like cheesecake compared 
to these APIs. These APIs take time, they are not as consistent as the Twitter API. They are spinning up VMs, right. I feel like 
[Nock](https://github.com/flatiron/nock)  is a really good solution for that.

**11:25 Someone**: I have a question about that, Nuno. So like if you have a giant set of tests, is there a nice way to just define 
run subsets of that. Because usually if you're going...if you're doing stuff in a build you just wanna run a chunk of those, but if
you're looking at going live you want to run them all. That's a problem that a lot of people have, they get this build that just 
gets bigger and bigger. Devs only want to run a small chunk. I'm curious what you think.

**Nuno Job**: So, in [specify](https://github.com/dscape/specify) you can specify which files you wanna run and you can specify which 
tests inside the files you wanna run because you give them names and you you can pass the names in the command line. Only those tests will 
run. I haven't added wildcards which would be like colon something and then it would run those wildcard tests. As for [Nock](https://github.com/flatiron/nock), you need to add Nock to each js file, on the top something like ```var nock = nock('http://twitter.com')``` and it then it
grabs the specification from the nock object, which means since it's per file, as long as you run the file, you will have the mocks or not.
Controlled by your environement variable. If you don't run the file, you don't run the test. It's really is simple to make sure that you're
selecting just those tests from those files and you have control over if they have mocks or not. I mean, in Nodejitsu we need this all the time.
So, this is quite essential. We can't afford to run all the tests all the time. - Hey, welcome substack.

**James Halliday**: Yooo, so, I actually...I missed a few minutes there, but I did catch the start of it. You guys are talking about mocks. I
want to caution people against going down the mocking route, because it's very easy testing against your mocks instead of reality. And, like, you
don't want to have to write tests against your mocks to make sure they're compatible with reality. A lot of weird bugs happen when your mocks
get out of sync with how reality actually works. What I would advise people do is instead of writing so many mocks, you should probably just 
seperate out your code into the code that does more side effect related I/O and then the code that does the more functional stuff. You really
want to minimize that layer that's actually doing the mocking, I think, as much as possible. That really needs to be said.

**Daniel Shaw**: Substack, how do you unit test something like [seaport](https://github.com/substack/seaport), which is all about I/O?

**James Halliday**: I don't, I just run the real I/O. I/O is cheap in Node, I/O is really cheap in Node. You can spin up server and bring
them down like it's nothing. In tens of milliseconds all of that happens.


<!-- Continue at 14:30! -->

