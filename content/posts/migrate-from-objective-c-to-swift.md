+++ 
draft = false
date = 2017-05-27T07:00:00+07:00
title = "Migrate from Objective-C to Swift"
description = ""
slug = ""
authors = []
tags = ["iOS", "migration"]
categories = []
externalLink = ""
series = []
+++

## Disclaimer
My background is an engineer with less than two years of industrial mobile development experience and less than a year in iOS (with some experience doing the server-side as well as Android development on the side), so you will probably find this post:

- Beginner-friendly
- Containing flaw/nonoptimal/simply stupid/etc solutions. But hey, it works. At least Fabric agree with me (we started migration at the end of March).

![Crash rate](/assets/migrate-from-objective-c-to-swift/crash-rate.png)

Nevertheless, I would truly appreciate for any correction and suggestion to make this article a better guide for people who want to make a switch.
## Context
The reasons why we choose to migrate to Swift:
- Swift is easier to write, and is less error-prone.
- Swift is the new (nah, not that new in 2017) official programming language for iOS development => future-proof
- Swift is the language that are attracting more and more developers from the Objective-C developers, and from people who is new to iOS development

Before making the migration, here are some of our presumption:

[ ] The migration can be done incrementally. We does not have to done everything at once

[ ] The stability should not be degraded after switching

[ ] The speed of development should not compromised
## Solutions
Background:
We have 4 different targets when building our app as follows

ChoppDev: for development, where we will do testing locally on our own computer
ChoppStaging: for staging, where we will do testing internally in our team
Chopp: for production, which will be released through App Store
Chopp-Cal: for UI testing with Calabash

### Setup and basic
You can have a look at the detailed documentation from Apple. But I will summarize it specifically for our project

- Singularize the product module name for all targets (e.g: Chopp). This is the reason why I mentioned four targets previously, many tutorials only tell us to use product module name, but they don't consider the case of different targets
- Create a bridging header file to use the Objective C in Swift by:
    - Create a bridging header file (e.g: Chopp-Bridging-Header.h)
    - Add the path to bridging header file in Build Settings => Objective-C Bridging Header (e.g: Chopp/Chopp-Bridging-Header.h)
- To use Swift in Objective-C file, just use 'import "(Project Module Name)-Swift.h"' (e.g: Chopp-Swift.h) in Objective C file

### Beyond basic

There are some cases where we need to make extra effort to make these two compatible with each other:

### Constants and delegate
To use swift constants and delegate in existing Objective-C, we have to add @obj for the singleton class, and inherit NSObject. So it is like

`@objc class Constant:NSObject`
### Swift singleton
Similar to the previous one, we have to have @objc and NSObject. After that, instead of using a famous one line singleton
```
class TheOneAndOnlyKraken {
    static let sharedInstance = TheOneAndOnlyKraken()
}
```
we have to use this

```
@objc class CPRewardHelper: NSObject {
    class var swiftSharedInstance: Helper {
        struct Singleton {
            static let instance = Helper()
        }
        return Singleton.instance
    }
    class func sharedInstance() -> Helper {
        return Helper.swiftSharedInstance
    }
}
```
Tips: at first, to avoid making mistakes, and for ease of testing at first. We focus on the custom view and cell since these are less of a dependency for others

## Conclusion
Let go over the presumption once again

[x] The migration can be done incrementally. We does not have to done everything at once => YES, we have had approximately 1/5 of the code translated to Swift, and the speed keeps increasing. Except for the beginning of the migration, which takes us a few hours, everything went smoothly

[x] The stability should not be degraded after switching => YES, as you could see from our Fabric crash data. We still maintain the data at 100% for at least 1/3 of the time. On some bad days, it may go down to 99%. Actually, all of the crash is occured from either old Objective-C code or library code. However, we are not satisfied with the crash rate yet. Our goal is still trying to keep the crash rate as close to 100% as possible, and we do not compromise on 99%

[x] The speed of development should not compromised => YES, we still release new build every one or two weeks, with new feature every two week or four week. Our first priotity is still to bring better and better experience for the users

Overall, we are satisfied with our choice. The migration does not show any significant impact to app stability and performance, but it truly enhance the developers experience. With Swift, we could write code at the faster speed, with less errors (we treat Optional seriously, try to avoid ! at all cost)

NOTE: Since Kotlin is now an official programming language for Android development, we are also considering that too :)