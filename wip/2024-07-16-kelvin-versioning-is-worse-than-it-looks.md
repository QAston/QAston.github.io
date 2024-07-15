---
layout: post
title:  "Kelvin versioning is worse than it looks"
date:   2024-06-16 20:07:56 +0000
categories: urbit kelvin versioning semver
---

Even though software versioning and its aspects are strictly about programmer development experience, this blogpost will try to aim for an explanation that can also be understood by a general audience, as Urbit advocates include the versioning scheme as a differentiator in their pitch for the new non-programmer users. Due to the nature of the topic, some programmer jargon is required to understand the problems being addressed by versioning schemes, as well as the current solutions and their characteristics.

Isn't it annoying that windows 10 keeps asking you to upgrade to windows 11? Why does Youtube keep redesigning their UI and annoying users, wouldn't it be great if they just stopped changing the layout? They're making it worse every time. Why does my phone need to be periodically updated?

Kelvin versioning solves none of these user problems - as they aren't caused by the numbering scheme used to distinguish the versions. There's nothing in Windows 10 that required a major update, internally it's still using the same build based version counter that it always has, with named major updates in which major OS features and deprecations are introduced.  

Youtube doesn't version their UI in any way, it does version the API and website internally for the software development purposes, but there's no need for versioning the website really - you just get what youtube decides to serve you personally and that is it.

What causes all of this "software churn" then? Some of it is caused by neccessary changes, like support for new hardware or new software solutions. When your system is updated bugs are fixed, user and developer features are added, including making it easier to develop applications or enabling using of new hardware.

Most of the software churn however is caused by the companies selling you the final product. Windows wanted a big update for marketing purposes. Youtube refreshes its ui to monetize you better and to appear more "modern" to users and potential shareholders. They also want to be able to add new features to keep the users coming in and drop the features which turned out too expensive. Youtube needs to be just good enough to not be abandoned for competing platforms and continued changes are needed to maintain that.

Unsurprisingly Urbit does the same thing, right now they're working on a completely new user interface to attract new users just like youtube does. Kelvin versioning didn't prevent them from doing this, in fact no versioning scheme can do this. So, what do versioning schemes actually do?

## What do versioning schemes actually do?

Versioning schemes are what software developers use to pin down the expected behavior of the software they're writing their program with. 

## Complaints about semver

Rich hickey made a popular talk which is brought up whenever semver or versioning schemes are brought up in programming cicrles. I generally agree with the points made by Hickey, but some of the points he makes are aimed at the wrong cause.

Hickey wrongly blames semver for the issues with packaging in java ecosystems. Java has a limitation that there's no way to deploy 2 different versions of classes of a package at the same time. This is a java-specific issue, rust allows this just fine. In java, the popular workaround is to include the major semver version in the package name. So, not an issue with semver, just an issue with how java library tools work.

Hickey is right to complain about packages that break compatibility for no reason. However, this isn't caused by semver, it's just a decision that library creators make. Luckily for us, most of the library authors tend to agree with Hickey and do not make breaking changes. Instead, they either do what Hickey recommends and make additive changes to their libraries, or they batch up all of the changes they make for a breaking change and only make the breaking change when the backlog of the things they want to break gets big. Both strategies are perfectly supported by semver, in Hickey's strategy you perpetually stay in 1.x.x version (this is what most languages do anyway), in the batch up strategy you increase the major version number very rarely, once every 2-3 years, and then the users can continue to use both of the artifacts so nothing is actually broken.

## Semver resolvers and status quo 

```ai-slop
A semver resolver is a crucial component of a modern software platform. It is responsible for managing dependencies and ensuring compatibility between different versions of software libraries.

When a developer specifies the dependencies for their project, the semver resolver analyzes the version ranges specified in the project's configuration file. It then resolves these ranges to specific versions of the libraries that satisfy the requirements.

The resolver takes into account the semantic versioning rules defined by the SemVer specification. It considers the major, minor, and patch versions of the libraries, as well as any pre-release or build metadata tags.

By resolving dependencies, the semver resolver helps prevent compatibility issues and ensures that the project uses compatible versions of the required libraries. It also helps in maintaining stability and reproducibility of the software by providing a consistent set of dependencies.

In addition to resolving dependencies, the semver resolver may also handle conflict resolution when multiple libraries have overlapping or conflicting requirements. It intelligently selects the best combination of versions that satisfy all dependencies, taking into account any constraints or preferences specified by the developer.

Overall, a semver resolver plays a vital role in managing dependencies and ensuring the smooth functioning of a modern software platform.
```





