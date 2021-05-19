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


### Google capabilties

By [malkia](https://news.ycombinator.com/user?id=malkia)

I worked at google for 2-3 years, under google3 depot.

You can actually compile, and run in dev/staging certain things, inspect code, click on function, and see callsites, even "debug" from the browser. Debugging, is pretty much, if your binary have stepped through a bookmark, then it'll tell you, and you may print out locals from the scope - like vars, etc. - hard to explain in short.

Then "checking out", to put in p4/svn - is not really like that - but you can find pretty much videos, docs explaining it. It's more like - you create a "workspace", "client" (piper has history from p4, so some terms are similar to perforce), and in this client - you "view" the entire depot, and changes you've made are "overlayed". Then you can submit these in a CL (much like perforce).

There is also git (git5) and hg mode, but I've never used them. I've used the "CitC" one (client in the cloud), and was great as I was able to edit, and later edit, even build,... even deploy all from the browser (though prefer real IDE there - like Eclipse/IntelliJ/CLion).

AFAIK, only few hundreth files are not visible to employees, and certain folks (contractors) may be limited there too.


### Google does monorepo and polyrepo

By [malkia](https://news.ycombinator.com/user?id=malkia)

Google's significant new project, fuchsia, is set-up as multi-git repo. For fuchsia, they use a tool called "jiri" to update the repos, previously (and maybe still in use) is the "gclient" sync tool way from depot_tools.
  
  * jiri: https://fuchsia.googlesource.com/jiri/
  
  * gclient: https://chromium.googlesource.com/chromium/tools/depot_tools.git/+/master/gclient
  
  * depot_tools: https://chromium.googlesource.com/chromium/tools/depot_tools.git

  * This reflects a bit to the build system of choice, GN (used in the above), previously gyp, feels similar on the surface (script) to Bazel, but has some significant differences (gn has some more imperative parts, and it's a ninja-build generator, while bazel, like pants/bucks/please.build is a build system on it's own).

  * Bazel is getting there to support monorepos (through WORKSPACEs), but there are some hard problems there.
