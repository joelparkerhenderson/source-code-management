# SCM at Google

Google has separate repos:

  * Android
  * Chrome
  * ChromeOS
  * Google3

Each repo has its own build system: Gradle, gyp/ninja, Portage, Blaze. 

When people talk about Google's monolithic repo they're talking about Google3.

  * It's important to stress that the Google mono-repo uses Perforce, not git. 

  * Google uses git/gerrit for Android.

Google3 here consists of several parts:

  * The source code itself, which is essentially Perforce. This includes code in C++, Java, Python, Javascript, Objective-C, Go and a handful of other minor languages.

  * SrcFS. This allows you to check out only part of the repo and depend on the rest via read-only links to what you need from the rest.

  * Blaze. Much like Bazel. This is the system that defines how to build various artifacts. All dependencies are explicit, meaning you can create a true dependency graph for any piece of code. This is super-important because of...

  * Forge. Caching of built artifacts. The hit-rate on this is very good and it consumes a huge amount of resources given the number of artifacts produced. Forge turns build times for some binaries from hours (even days) into minutes or even seconds.

  * ObjFS. SrcFS is for source files. ObjFS is for built artifacts.

  * Google has shown the monolithic model of source code management can scale to a repository of one billion files, 35  million commits, and tens of thousands of developers.

The workflow is pretty good:

  * You can check out directories if you want to modify them, and just use the read only version if you don't. 

  * You can still step through the read only code with a debugger however.

Google uses protobufs for platform independence. 

Engineers who are working on Google3 tend to compile very little to nothing locally.

  * Google has spent a vast amount of effort making it so the same build and caching systems can handle C++ server code as well as Objective-C iOS app code. 

  * There are definite issues with Google3, like the dependency graph getting so large that even reading it in and figuring out what to build is a significant performance cost and optimization issue.

Engineers working on Android, Chrome, and ChromeOS, tend to compile a lot of things locally, and thus get far beefier workstations.

Presubmit checks, code review processes, and rollbacks, keep things sane.

A monorepo this size would simply not scale on git, at least not without huge amounts of hacks (and to be fair, Google built an entire infrastructure on top of Perforce to make their monorepo work).

This used to be done manually via "gcheckout" but that's long since been replaced. Users now don't do anything but create quick throwaway clients that have the entire repo in view.

Until very recently there was a versioning system for core libraries so those wouldn't typically be at HEAD (minimizing global breakage). Even that has been eliminated now and it's truly just the presubmit checks and code review process that keeps things sane.

Plenty of opensource projects are in google3 (that frequently gets merged into external).

  * Many OSS projects live in google3: e.g. gRPC, protobufs, etc.

  * Projects use Copybara to import/export OSS code. https://github.com/google/copybara

