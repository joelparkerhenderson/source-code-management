# Source code management

Notes on mono-repos, trunk-based-development, etc.

Work in progress.

* [Security and auditing, such as for SOX, PCI, HIPAA, etc.](security.md)
* [Bazel/Blaze for dependencies](bazel_blaze_for_dependencies.md)
* [SCM at Facebook](scm_at_facebook.md)
* [SCM at Google](scm_at_google.md)

See also:

* [Trunk based development](https://trunkbaseddevelopment.com/)
* [Why Google stores billions of lines of code in a single repository (2016) (acm.org)](https://dl.acm.org/citation.cfm?id=2854146)
* [Scaling Mercurial at Facebook](https://code.facebook.com/posts/218678814984400/scaling-mercurial-at-facebook/)


## Tradeoffs of mono-repo vs. many-repo

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


## Facebook vs. Google

By [cletus](https://news.ycombinator.com/user?id=cletus) (ed: some of this content is repeated above/below for clarity)

Related:

* [SCM at Facebook](scm_at_facebook.md)
* [SCM at Google](scm_at_google.md)

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

