---
layout: post
title: "Postmortem: Swift at Artsy"
date: 2017-01-22 12:18
author: orta
categories: [swift, eigen, eidolon, javascript, emission, reactnative]
series: React Native at Artsy
---

Swift was announced in June 2014, by August we had started using it in Artsy. By October, we had [Swift in production][eidolon-postmortem] channeling hundreds of thousands of dollars in auction bids.  

Since then we've built an [appleTV app][emergence] in Swift, integrated Swift-support into our key app Eigen and built non-trivial parts of that [application in Swift][live-a]. It is pretty obvious that Swift is the future of native development on Apple platforms. 

We first started experimenting with with React Native in February 2016, and by August 2016, we announced that [Artsy moved to React Native][artsy-rn] effectively meaning new code would be in JavaScript from here onwards.

We're regularly asked _why_ we moved, and it was touched on briefly in our announcement but I'd like to dig in to this and try to cover a lot of our decision process. So, if you're into understanding why a small team of iOS developers with decades of experience switched to JavaScript, read on. 

<!-- more -->

We were finding that our current patterns of building apps were not scaling as the team and app scope grew. Building anything new inside Eigen barely re-used existing code, and it was progressively taking longer and longer to build features. App and test build times were increasing, it would take 2 iOS engineers to build a feature in a similar time-frame as a single web engineer. 

By [March 2015][gave_up], we gave up trying to keep pace with the web.

Once we came to this conclusion, our discussion came to "what can we do to fix this?" Over the course of the 2015/2016 winter break we explored ideas on how we could write more re-usable code.    

### What are Artsy's apps?

Eigen specifically is an app where we taken JSON data from the server, and convert it into a user interface. It nearly always can be described as a function taking data and mapping it to a UI.

We have different apps with different trade-offs. [Eidolon][eidolon] (our Auctions Kiosk app) which contains a lot of Artsy-wide unique business logic which is handled with local state like card reader input, or unique user identification modes. [Emergence][emergence] is a trivial-ish tvOS app which has a few view controllers, and is mostly handled by Xcode's storyboards.

Eigen is where we worry, other apps are limited in their scope, but Eigen is basically the mobile representation of Artsy. We're never _not_ going to have something like Eigen. 

We eventually came to the conclusion that we needed to re-think our entire UIKit stack for Eigen. Strictly speaking, Objective-C was not a problem for us, our issues came from abstractions around the way we built apps.

Re-writing from scratch was not an option. That takes a lot of time and effort to remove technical debt. We also don't need a big redesign. However, a lot of companies used the Objective-C -> Swift transition as a time to re-write from scratch. We asked for the experiences from developers who had opted to do this, they said it was a great marketing tool for hiring - but was a lot of pain to actually work with day to day. They tend to talk abut technical debt, and clean slates - but not that Objective-C was painful and Swift solves major architectural problems. 

In the end, for Eigen, we came to the conclusion that we could try build a component-based architecture either from scratch ( one very similar to the route Spotify ([hub][hub]) or Hyperslo ([Spots][spots]) took ) or inspired by React ( like Bending Spoons ([Katana][katana]) ).

### Swift's upsides

Continuing to build native apps via native code had quite a bit running for it:

* **It was consistent with our existing code.** We wrote hundreds of thousands of lines of code in Objective-C and maybe around a hundred thousand of Swift. The majority of the team had 5+ years of Cocoa experience and no-one needs to essentially argue that _continuing_ with that has value.

* **Swift code can interact with Objective-C and can work on it's own.** We can write Swift libraries that can build on-top of our existing infrastructure to work at a higher level of abstraction. Building a component-based infrastructure via Swift could allow easy-reuse of existing code, while providing a language difference for "new app code" vs "infra." 

* **People are excited about Swift.** It's an interesting, growing language, and one of the few ones non-technical people ask about. "Oh you're an iOS developer, do you use Swift?" is something I've been asked a lot. The rest of the development team are have signed up multiple times for Swift workshops and want to know what Swift is, and what it's trade-offs are.

* **Swift improves on a lot of Objective-C.** Most of the patterns that are verbose in Objective-C can become extremely terse inside Swift. Potentially making it easier to read and understand.   

* **We would be using the official route.** Apple obviously _want_ you to be using Swift, they are putting a _lot_ of resources into the language. There are smart people working on the project, and it's becomes more stable and useful every year. There aren't any _Swift-only_ APIs yet, but obviously they'll be coming.

* **It's a [known-unknown][known-known] territory.** We have a lot of knowledge around building better tooling for iOS apps. From libraries like [Moya][moya], to foundational projects like [CocoaPods][cocoapods]. Coming up with, and executing dramatic tooling improvements is possible. Perhaps we had just overlooked a smarter abstraction elsewhere and it was worth expanding our search.

  This is worth continuing here, because if we end up building something which gains popularity we get the advantage of working with a lot of perspectives, and being able to gain from other people working on the same project. It's a pattern Basecamp discuss when they [talk about rails][rails] by beginning with a real project and abstracting outwards.

### Native Downsides

<!--It's hard to talk about some of the downsides to working natively without having something to contrast against. 

For example, without being able to fork any project in your dependency stack - it's really not something you think of as being feasible. Had someone asked "would you fork Foundation  with your changes" or said "Ah yeah, check out the Steipete fork of UIKit for the Popover rotation orientation bug fix" to me a year ago, I would have just laughed, as the idea would have never crossed my mind.-->

The biggest two issues come from differences in opinions in how software should be built. 

* **Types.** Types are useful. Overly strict typing systems make to hard to build _quick_ (not easy) to change codebases.

  Strictly typed language work _really_ well for [building systems][systems], or completely atomic apps - the sort Apple have to build on a day to day basis. When I say an atomic app, I mean one where the majority of the inputs and outputs exist within the domain of the application. Think of apps with their own filetypes, that can control inputs and outputs really easily.

  Even in Objective-C, a looser-typed language where you were not discouraged from using meta--programming, handling JSON required _a tonne_ of boilerplate, and inelegant code when working with an API. Considering how bread-and-butter working with an API is for most 3rd party developers it should come as no surprise that the most popular CocoaPods are about handling JSON parsing, and making network requests.  

  Problems which Apple, generally speaking, don't have. They use iCloud, or CloudKit, or whatever. The Apple opinion was was neatly summed up on the official Swift blog on how to handle JSON parsing [exhibits the problem well][swift_blog].

  > Swift’s built-in language features make it easy to safely extract and work with JSON data decoded with Foundation APIs — without the need for an external library or framework.

  They do, but it's not great code to write nor maintain.

* **Slow.** Native development when put next to web development is slow. Application development requires full compilation cycles, and full state restart of the application that you're working on. A trivial string change in Eigen takes [25 seconds][eigen_25] to show up. When I tell some developers that time, they laugh and say I have it good.

  This becomes extremely painful once you start [getting used][injection_twitter] to technologies like Injection for Xcode, which is what ruined my appetite for building apps the traditional way. We were starting to come up with all sorts of techniques to allow separation of any part of the codebase into a new app so you can iterate just there. 
  
  I've heard developers say they use using Playgrounds to work around some of these problems, and the KickStarter app has probably the closest I've seen to an [actual implmentation of this][kickstart_play].

  The Swift compiler is slow. Yes, it will improve. However, it's root issue comes from Swift being a more complicated language to compile, and it doing more work. On the side of doing more work, the awesome type inference systems can make it feel arbitrary about what will take longer to compile or not. We eventually [automated having our CI warn us][danger-eigen] whether the code we were adding was slow. 


### React Native

You may want to read our announcement of switching to [React Native][artsy-rn] in anticipation of this. However the big three reasons are:

* Better developer experience.
* Same conceptual levels as the rest of the team.
* Ownership of the whole stack.

However, the key part of this post is how does this compare to native development? Also, have these arguments stood up to the test of time a year later? 

#### Better Developer Experience

* Better abstractions for building JSON driven apps
 - https://rauchg.com/2015/pure-ui

* Falling back to native code is no problem at all

* Tests operate outside of the iOS sim


#### Same Tools, Different Dev


* Reduce the barrier to entry for rest of team
  - more external contributions from web engineers since we moved

* Will only be usable for Apple products
 - Why use Swift when there's Kotlin?
 - Swift on a Server might be usable in a few years, not sure anyone would push on server  
 
* Encourages mobile developers to make API changes, same concepts
  - Web and API is JS, same tools, same workflow

#### Owning the stack

* Conceptual idea of customizing your language to the project
  - We pick and choose language features we want

* "Forking" / Contributing back (can't use our own fork of Swift)
  - You can expect a reply to an issue, you don't to a radar
  - Relay
  - VS Code
  - React-Native

* Open but hard to be accessible
 - You need to be a compiler engineer to improve Swift
 - Can't fork Foundation, Cocoa, UIKit
 - Small pool of active contributors to OSS

* Tooling immaturity, and redundant re-implementations
 - Community manually re-create a bunch of apple tools, why?
 - Community had to re-write every useful library "For Swift" again, making it instable
 - Community changed to be "Swift XX" as opposed to "Cocoa XX", swift purism vs mature pragmaticism
 - https://twitter.com/orta/status/649214813168640000


[eidolon-postmortem]: http://artsy.github.io/blog/2014/11/13/eidolon-retrospective/
[emergence]: https://github.com/artsy/emergence
[live-a]: http://artsy.github.io/blog/2016/08/09/the-tech-behind-live-auction-integration/
[artsy-rn]: o/blog/2016/08/15/React-Native-at-Artsy/
[what-is-artsy-app]: /blog/2016/08/24/On-Emission/#Why.we.were.in.a.good.position.to.do.this
[eidolon]: https://github.com/artsy/eidolon
[spots]: https://cocoapods.org/pods/Spots
[hub]: https://cocoapods.org/pods/HubFramework
[katana]: https://cocoapods.org/pods/Katana
[gave_up]: https://github.com/artsy/mobile/issues/22
[known-known]: https://en.wikipedia.org/wiki/There_are_known_knowns
[moya]: https://github.com/moya/moya
[cocoapods]: https://cocoapods.org
[rails]: https://signalvnoise.com/posts/660-ask-37signals-the-genesis-and-benefits-of-rails
[systems]: http://mjtsai.com/blog/2014/10/14/hypothetical-objective-c-3-0/#comment-2177091
[swift_blog]: https://developer.apple.com/swift/blog/?id=37
[eigen_25]: https://twitter.com/orta/status/778242899821621249
[injection_twitter]: https://twitter.com/orta/status/705890397810257921
[kickstarter_play]: https://github.com/kickstarter/ios-oss/tree/master/Kickstarter-iOS.playground/Pages
[danger-eigen]: https://github.com/artsy/eigen/pull/1465