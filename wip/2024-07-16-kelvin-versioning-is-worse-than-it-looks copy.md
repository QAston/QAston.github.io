---
layout: post
title:  "Kelvin versioning is worse than it looks - Part 2"
date:   2024-06-16 20:07:56 +0000
categories: urbit kelvin versioning semver
---



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





