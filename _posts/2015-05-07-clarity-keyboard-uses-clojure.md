---
layout: post
title: "What Makes Clarity Keyboard Tick?<br/> Clojure!"
description: ""
comments: true
tags: [clojure, clarity, programming, swiftkey, android, keyboard, ime, java, lein-droid, async]
authors: [ben, adam]

---

<img style="width:250px;float:right;shape-outside:circle();" src="https://blog.swiftkey.com/content/uploads/2015/05/ClarityLogo.png" alt="Clarity logo"/>

[[Reposted from the SwiftKey Tech Blog]](http://swiftkey.com/en/blog/what-makes-clarity-keyboard-tick-clojure/)

*Co-written with [Adam Clements](https://github.com/adamclements/).*<br /><br />

Last week here at [SwiftKey](http://swiftkey.com), we released [Clarity Keyboard Beta](https://play.google.com/store/apps/details?id=com.swiftkey.clarity.keyboard&referrer=utm_source%3Dadamblog%26utm_medium%3Dblog%26utm_content%3Dprogrammingpost) - an experimental keyboard that we'll be using to test out some new keyboard concepts over the coming few weeks and months. It's been built from the ground up and uses a host of bleeding-edge technologies as well as an interesting and unusual (for mobile) choice of programming language: Clojure.

This gave us an unprecedented degree of freedom and agility. When you can safely redefine whole chunks of your program on the fly without even restarting, it changes the way you think about designing your product and gives you the freedom to genuinely **play** with new features.

## What is Clojure?

<img style="width:250px;float:right;shape-outside:circle();" src="https://blog.swiftkey.com/content/uploads/2015/05/ClojureLogo.png" alt="Clojure logo"/>

[Clojure](http://clojure.org/) is a functional programming language that, like Java, targets the JVM.

For the uninitiated a language is 'functional' because it encourages the use of 'pure' functions (no side-effects - i.e. no state) and immutable data. Contrast this with an imperative language like Java in which the use of member variables (state) that can be updated many times during a program's execution are the norm. One of the biggest benefits of this approach is that multi-threaded programming is much easier and safer if you don't have to worry about mutable state (i.e. variables being updated by multiple threads, potentially simultaneously and/or unpredictably). It also makes chunks of code easier to reason about without having to understand all of the context in which it is used. 

I won't list the many other benefits of using Clojure here - you can read a more extensive [rationale](http://clojure.org/rationale) on the Clojure website. Not everything is rosy though, there are some big drawbacks to using Clojure too. The main one being that it has historically had a very long initial load time. The time to load a Clojure application is often counted in seconds! As a result of this, Clojure is most often used to build server infrastructure where a long initial load time is irrelevant for the long running process. It has rarely been used to build consumer mobile applications.



## Clojure on Android

As you might expect, the issue of the long load time was the biggest barrier to using Clojure to build a mobile application. How long would you wait for an application to load on your mobile phone before thinking it's crashed (especially when in some cases it had... sorry people on 0.3.2 with a Samsung phone)? However with the help of the key contributors to some of the open-source libraries we're using we've reached a point where using Clojure to build a mobile application in production is absolutely feasible. Clarity Keyboard's initial loading time is around 1.5-2 seconds and there are still a number of improvements we plan to make to get this down well below a second.

This load time was made possible by [Alexander Yakushev's](https://github.com/alexander-yakushev) fork of Clojure called [Skummet](http://clojure-android.info/skummet/), with standard upstream Clojure the load time is still around 10 seconds, which is obviously unacceptable. Skummet takes away some of Clojure's dynamicism in return for some extra speed. This means we accept the long load time when developing and get REPL development, then when we're ready for launch hit the Skummet button for a free speed boost. Long term we'd really like to see some lean compilation options added to the official version of Clojure so that we don't have to rely on an unofficial fork. Hopefully now we've shown that Clojure on Android is not only possible, but desirable, this will become more of a priority for upcoming versions.

### Development

We're using a fairly typical Clojure development setup here - nothing Android-specific and pretty standard stuff for Clojure developers: Emacs + cider.

One of the most powerful tools this setups offers us though - which for us was a revelation when we first tried this on Android a year or so ago - was the Clojure REPL **on Android**!

### The Clojure REPL

REPL stands for Read, Evaluate, Print, Loop - an environment in which you can tweak your program on the fly without re-compiling and re-installing (a process that can take several minutes). The Clojure REPL has been especially valuable for us while working on Clarity Keyboard, in particular when making subtle changes to the user experience, and has made a big difference to our development speed.

We no longer need to re-compile and re-install our application to test our changes, we simply re-evaluate our changes on the device and a moment later the keyboard is running our new code. In a lot of cases we can make these changes without even refreshing the keyboard on screen: A user testing our debug build might spot something wrong with the keyboard layout and we can tweak it before they've even finished their sentence.

### Build tools

Unfortunately, as the use of Clojure on Android is a relatively recent innovation all of the Android build tools assume that you are using standard Java, in a standard IDE, and that you won't wander too far from the beaten path.

Luckily for us [Daniel Solano Gómez](https://github.com/sattvik), [Zach Oakes](https://github.com/oakes) and [Alexander Yakushev](https://github.com/alexander-yakushev) among others have done a lot of the hard work in improving Clojure support for Android as part of the [Clojure-Android](http://clojure-android.info) group on GitHub. This includes lein-droid, a plug-in for the leiningen build system which deals with turning Clojure code into an APK, packaging resources etc. and a special build of Clojure which can actually generate Android Dalvik byte code at runtime, allowing for the dynamic behaviour mentioned earlier.

Having said this, when looking to make a production-quality app, there were some features which simply hadn't been built yet and so tool support was missing for things like version code generation, bundling of native libraries, Proguard optimisation and Multi-dex (which allows you to have more than 65,000 method references). However, lein-droid is open source and its contributors have been really supportive and responsive to our proposed changes so we've been able to propose the missing features and have them merged into upstream quickly and they are now part of the official version.

## Design decisions influenced by Clojure

### Event Driven
In Clarity, everything that happens in the system is an "event" expressed as pure data, namely a Clojure map. No modules can do or react to anything if it isn't broadcast as data in an event.

{% highlight clojure %}
{:type :start-input :editor {:type :plain :action :send}}
{:type :touch :touches [[0.12 0.43 12332]] :action :down}
{:type :touch :touches [[0.12 0.43 12332] [0.13 0.43 12333]] :action :up}
{% endhighlight %}

The fact that everything is data means that we can really easily see everything that's happening in the system when debugging - simply print out the event stream as things pass through it. It also means that if we want to test a system in isolation, it's trivial to spin up only that system and simply inject the events that it expects. All application configuration is simply done by appending a `:persist :true`  key to **any** event which simply indicates that the settings module should store that particular event and replay it on application startup.

This is all made possible by persistent immutable data structures in Clojure. You can happily send the events off to as many receivers as you want and know that the operation of one cannot change the copy that another sees. It also means that we can broadcast for example every time a new sample is added to a touch trace, **without** creating an entirely new list, it will share structure with the previously emitted touch event but without changing it for any modules that are still processing it.

In practice this is really powerful and leads to excellent modularity.

### Modular

Because all subsystems only care about events in/out, and don't care at all about where those events come from or where they go to, it's absolutely trivial to replace whole subsystems or add new ones, without breaking what's already there. It also means practically no conflicts when merging feature branches, because each new feature can be implemented in isolation. This has the added benefit that if you're not sure about a feature, you can simply keep the old feature code but not activate it (and it will get stripped out in the build step). By making sure everything considers the fact that there may be multiple receivers for their events, it's easy and encouraged to keep everything nice and generic, so you don't end up with all your modules depending on each other in a spaghetti code mess.

<img src="https://blog.swiftkey.com/content/uploads/2015/05/pseudo-architecture-clarity-copy.png" alt="Clarity architecture diagram"/>

The fact that everything is nice and modular makes things very very easy to test. You can simply fire events in one side of a module and check that the appropriate events come out the other side. Or spin up a couple of modules and fire events in to check that they interact with one another as expected. This combined with [test.check](https://github.com/clojure/test.check) to generate all sorts of unforeseen properties of the incoming events and check that various properties hold true have made testing a breeze. We plan to post again in the future about our testing setup.

### Totally Asynchronous

Given that we have immutable, persistent data and many largely independent modules, it makes sense to make everything asynchronous by default. [core.async](https://github.com/clojure/core.async) was an excellent fit for us as it allowed us to set up our central stream of events totally asynchronously but in such a way that we can guarantee ordering of events (i.e. if a `:change-layout` event happens followed by a `:touch` event they are guaranteed to arrive that way round, even if some other module is being really slow processing `:change-layout` events).

While it's really powerful, and beats using threading primitives hands down, it is still asynchronous code, and asynchronous code is always hard to reason about and even harder to test. In practice, our unit tests tend not to spin up modules using core.async, but instead feed lists of events through the module using [transducers](http://clojure.org/transducers) where possible, and get a simple list out the other side.

## Conclusion

~~We'll be publishing more detailed posts about our experience building mobile applications in Clojure on Android; technologies we've used such as [test.check](https://github.com/clojure/test.check), [Clojure on Android](http://clojure-android.info/) and [core.async](https://github.com/clojure/core.async); REPL-driven development; Clojure itself and the Clarity project over the next few weeks and months. Let us know in the comments if there's anything in particular you'd like us to cover in more depth.~~