# Repository strategies

Work in progress.

See also:

* [Trunk based development](https://trunkbaseddevelopment.com/)
* [Why Google stores billions of lines of code in a single repository (2016) (acm.org)](https://dl.acm.org/citation.cfm?id=2854146)
* [Scaling Mercurial at Facebook](https://code.facebook.com/posts/218678814984400/scaling-mercurial-at-facebook/)


## Tradeoffs

Benefits:

  * unified versioning

  * extensive code sharing

  * simplified dependency management

  * atomic changes

  * large-scale refactoring

  * collaboration across teams

  * flexible code ownership

  * code visibility

Drawbacks:

  * having to create and scale tools for development and execution

  * having to maintain code health

  * potential for codebase complexity (such as unnecessary dependencies)

Opinions:

  * For a huge non-open codebase there are some pretty large downsides to a fully distributed VCS in exchange for relatively few benefits. 


## Mono-repo vs. Multi-repo

To me the defining aspect of a monorepo is storing apparently unrelated code in the same repo, which is highly non-intuitive.


### Opinions

I have two main concerns when I see monorepos being used.

  * First, like in other areas, I see companies that want to "google scale" and blindly copy the idea of monorepos but without the requisite tooling teams or cloud computing background / infrastructure that makes this possible.

  * Second, I worry about the coupling between unrelated products. While I admit part of this probably comes from my more libertarian world view but I have seen something as basic as a server upgrade schedule that is tailored for one product severely hurt the development of another product, to the point of almost halting development for months. I can't imagine needing a new feature or a big fix from a dependency but to be stuck because the whole company isn't ready to upgrade.

  * I've read of at least one less serious case of this from google with JUnit: "In 2007, Google tried to upgrade their JUnit from 3.8.x to 4.x and struggled as there was a subtle backward incompatibility in a small percentage of their usages of it. The change-set became very large, and struggled to keep up with the rate developers were adding tests.""

We don't use a monorepo, and it's hell. 

  * I may make a change to X, but some other developer doesn't know it, and then one day when he makes a change to X and tries to pull those changes into Y, now he has to change the bits of Y which are affected by my changes as well as his.

I would love to have a monorepo, although I'd advocate for branch-based rather than trunk-based development. 

  * Yes, merges can be their own kind of hell, but they impose that cost on the one making breaking changes, rather than everyone else.


## Git vs. Subversion vs. CVS vs. Perforce etc.

Many open source projects have migrated to git. 

A lot of companies don't use git.

People switched to Git because Subversion merging continues to suck, and branching and tagging are implemented on the most inane way possible.

Why have many projects switch from CVS, Subversion, etc. to Git?

  * Git is fundamentally built on more powerful ideas than what came before it.

  * Git makes it easier to branch, merge, tag, and do distributed source code.

  * GitHub makes it easier/cheaper to host open source projects, fork, and comment.

  * Standardizing on one VCS makes it easier for more people to contribute.


## Partial check out

Can you check out an individual directory without having to check out the entire repo?

  * Git: no

  * Svn: yes, and this is the default way to work
 
  * Google mono-repo: yes, and this is the default way to work

For example in `svn`, checking out only some subdirectory instead of entire repo is pretty much the default way how you should use it.

This is a common feature in most non-DCVS. 

This is helpful for projects where you want to skip downloading some directories that massive, such as a game project with an art directory that is 500 GB.


Notes:

* While technically true due to some features of tooling, that is really only masking off part of the repo under a READ-ONLY directory.

  * Builds can (and usually do) depend on things that aren't part of your local checkout.

  * I'd say CitC is a much more accurate representation of the way Piper and blaze "expect" things to work.


## Git submodules

IIRC the Git people seemed to reject the idea of large code bases. Or, rather, their solution was to use Git submodules. There was (and maybe is?) parts of the Git codebase that didn't scale because they were O(n). Apologies if I'm misspeaking here but I peripherally followed these discussions on HN and elsewhere years ago as someone from the outside looking in so I'm no authority on this.

The problem of course is that Git submodules don't give you the benefits of a single repo and I've honestly not heard anyone say anything good about Git submodules.


## Google

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

	
### Google CI/CD

Bazel solves this. 

  * All build targets (buildable units) explicitly define their direct dependencies. 

  * If you modify something, you can then run all tests for all units that transitively depend on your change.

  * Note: Bazel is known internally as Blaze.

Bazel lets you do reverse dependency querying with the query language.

  * I.e. "Bazel, give me the list of targets that depend on this target that I've just modified".

        $ bazel query 'rdeps(//foo/my:target, //...)'

  * For surface level changes the target list is often quite small. For changes to core libraries, well, you run a lot of tests.

  * Of course, the query '//...' in the monorepo will take a long time or not work, because the target universe of "//..." is far too large. This is where other systems come in.

Google CI/CD works really well.

  * One benefit is all tests affected by your target is run as you presubmit. 

  * The benefit is everything is at head so things like library bugs, security bugs are all handled naturally as part of the new releases. 

  * This usually happens twice a week for most server binaries.

  * Changes that break lots of projects are rolled back quite fast. Usually.

See https://docs.bazel.build/versions/master/query.html#rdeps


## Facebook

Facebook engineers historically see four repos: 

  * www
  * fbobjc
  * fbandroid
  * fbcode (C++, Java, Thrift services, etc)

As of 2017 Facebook uses one `hg` repo (www as a possible exception, I'm unsure). The "sparse checkout" machinery disguises it, but for engineers working cross platform (e.g. React Native) it's routine to make commits that span platforms.

At one point these repos were Git but for various reasons ended up being migrated to Mercurial some years ago.

FB uses GraphQL at a client level and Thrift at the service level.

So one pain point is that, for example, you can modify a GraphQL endpoint in one repo but its used by clients in others (ie mobile clients). There are lots of warnings about making backward-incompatible changes, some of them excessively pessimistic because deterministically showing something will break some mobile build in another repo is hard.


## Facebook vs. Google

By [cletus](https://news.ycombinator.com/user?id=cletus) (ed: some of this content is repeated above/below for clarity)

At Facebook, you can modify a GraphQL endpoint in one repo, but its used by clients in others (ie mobile clients). There are lots of warnings about making backward-incompatible changes, some of them excessively pessimistic because deterministically showing something will break some mobile build in another repo is hard.

At Google, Google3 has less of these problems because the code is in the same repo. On top of that, Google has spent a vast amount of effort making it so the same build and caching systems can handle C++ server code as well as Objective-C iOS app code. Basically if you're working on Google3 you basically compile very little to nothing locally.

Engineers on Android, Chrome and ChromeOS however compile a lot of things locally and thus get far beefier workstations.

At FB the mobile build system doesn't seem to be as advanced in that there is a far higher proportion of local building.

Just to stress, the above is just my personal experience and I hope it's taken as intended: general observations rather than complaints and definitely not arguing that one is objectively better than the other. There are simply tradeoffs.

	

## How long does a commit with tests and checks take?

Google:

  * Unit tests finish relatively fast.

  * Integration tests that bring up server tend to be slow (30-40 mins best case almost for these for projects im working on). 

  * Mobile tests that run on emulators are the worst.

  * The cost of testing gets amortized two ways: you can run immediate unit tests manually on command line as the fastest signal. Then when you send for code review, presubmit runs. During code reviews you may choose to run them as you go. Eventually when you submit they run again. If there has not been any changes to your commit/cl and there is an already passing submit, it will just skip and submit.

Twitter: 

  * Generally submit queue takes 10-20 minutes, longer if you're changing a core library.

  * The larger factor is what kind of test has to run. Feature tests can take a while on CI if you're spinning up lots of embedded services (dependent services, MySQL, storage layers, etc).

