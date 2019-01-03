# Source code management - notes and ideas

Notes on source code management such as Git, repository architecture such as monorepo vs. multirepo, topic flows such as trunk-based-development, and more. Work in progress. Improvement ideas welcome.

Contents:

* [](#)


Files:

* [Security, auditing, compliance](security_auditing_compliance.md)
* [Bazel/Blaze for dependencies](bazel_blaze_for_dependencies.md)
* [SCM at Facebook](scm_at_facebook.md)
* [SCM at Google](scm_at_google.md)

See also:

* [Trunk based development](https://trunkbaseddevelopment.com/)
* [Why Google stores billions of lines of code in a single repository (2016) (acm.org)](https://dl.acm.org/citation.cfm?id=2854146)
* [Scaling Mercurial at Facebook](https://code.facebook.com/posts/218678814984400/scaling-mercurial-at-facebook/)
* [Monorepos: Please don’t! by Matt Klein](https://medium.com/@mattklein123/monorepos-please-dont-e9a279be011b)


## Tradeoffs of monorepo vs. multirepo

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


## Monorepo vs. multirepo


### What is monorepo? What is multirepo?

Monorepo is a nickname that means "using one repository for the source code management version control system".

  * A monorepo architecture means using one repository, rather than multiple repositories.

  * For example, a monorepo can use one repo that contains a directory for a web app project, a directory for a mobile app project, and a directory for a server app project. 

  * Monorepo is a.k.a. one-repo or uni-repo.
  
Multirepo is a nickname that means "using multiple repostories for the source code management version control system". 

  * A multirepo architecture means using multiple repositories, rather than one repository.

  * For example, a multirepo can use a repo for a web app project, a repo for a mobile app project, and a repo for a server app project. 

  * Multirepo is a.k.a. many-repo or poly-repo.  


### What are key similarities and differences between monorepo and multirepo?

Key similarities between monorepo and multirepo:

  * Both architectures ultimately track the same source code files, and do it by using source code management (SCM) version control systems (VCS) such as git or mercurial. 
  
  * Both architectures are proven successful for projects of all sizes.

  * Both architectures are simple to implement using any typical SCM VCS, up to a scaling limit.

The key differences between monorepo and multirepo in terms of structure:

  * A monorepo manages projects in one repository, together, holistically. In typical practice, a monorepo repo contains multiple projects, programming languages, packaging processes, and the like. 

  * A multirepo manages projects in multiple repositories, separately, independently. In typical practice, a multirepo repo contains one project, programming language, packaging process, release roadmap, etc.

The key differences between monorepo and multirepo in terms of pros and cons:

  * Pro: monorepo proponents like the ability to work in all the projects simultaneously, all within the monorepo. A monorepo emphasizes that code changes affect all the projects, and can be tracked together, tested together, and released together.

  * Pro: multirepo proponents like the ability to work in each project one at a time, each in its own repo. A multirepo emphasizes that code changes affect only one project, and can be tested independently, and released independently.
  
  * Con: monorepo scaling necessitates specialized tooling. For example, it is currently not practical to use vanilla git with very large repos, or very large files, without any extra tooling. For monorepo scaling, teams invest in writing custom tooling and providing custom training.

  * Con: multirepo scaling necessitates specialized coordination. For example, it is currently not practical to use vanilla git with many projects across many repos, where a team wants to coordinate code changes, testing, packaging, and releasing. For multirepo scaling, teams invest in writing coordination scripts and careful cross-version compatibility.



### Monorepo scaling

Monorepo scaling becomes a problem when a typical developer can't work well with the code by using typical tools such as vanilla git.

  * Monorepo scaling eventually becomes impractical in terms of space: when a monorepo grows to have more data than fits on a developer's laptop, then the developer cannot fetch the monorepo, and it may be impractical to obtain to more storage space.

  * Monorepo scaling eventually becomes impractical in terms of time: when a monorepo grows, then a complete file transfer takes more time, and in practice, there are other operations that also take more time, such as git pruning, git repacking.
    
  * A monorepo may grow so large contain so many projects that it takes too much mental effort to work across projects, such as for searching, editing, and isolating changes.

Monorepo scaling can be improved by:

  * Some type of virtual file system (VFS) that allows a portion of the code to be present locally. This might be accomplished via a proprietary VCS like Perforce which natively operates this way, or via Google’s “G3” internal tooling, or via Microsoft’s GVFS.

  * Sophisticated source code indexing/searching/discovery capabilities as a service. This is because a typical developer is not going to have all the source code locallly, in a searchable state, using vanilla tooling. 

Monorepo scaling seems to become an issue, in practice, at approximately these kinds of metrics:

  * 100+ developers writing code full time.

  * 100+ projects in progress at the same time.

  * 100+ packagings during the same time period, such as a daily release.


### Opinions

I have two main concerns when I see monorepos being used.

  * First, like in other areas, I see companies that want to "google scale" and blindly copy the idea of monorepos but without the requisite tooling teams or cloud computing background / infrastructure that makes this possible.

  * Second, I worry about the coupling between unrelated products. While I admit part of this probably comes from my more libertarian world view but I have seen something as basic as a server upgrade schedule that is tailored for one product severely hurt the development of another product, to the point of almost halting development for months. I can't imagine needing a new feature or a big fix from a dependency but to be stuck because the whole company isn't ready to upgrade.

  * I've read of at least one less serious case of this from google with JUnit: "In 2007, Google tried to upgrade their JUnit from 3.8.x to 4.x and struggled as there was a subtle backward incompatibility in a small percentage of their usages of it. The change-set became very large, and struggled to keep up with the rate developers were adding tests.""

We don't use a monorepo, and it's hell. 

  * I may make a change to X, but some other developer doesn't know it, and then one day when he makes a change to X and tries to pull those changes into Y, now he has to change the bits of Y which are affected by my changes as well as his.

I would love to have a monorepo, although I'd advocate for branch-based rather than trunk-based development. 

  * Yes, merges can be their own kind of hell, but they impose that cost on the one making breaking changes, rather than everyone else.

If tech's biggest names use a monorepo, should we do the same?

  * Some of tech’s biggest names use a monorepo, including Google, Facebook, Twitter, and others. Surely if these companies all use a monorepo, the benefits must be tremendous, and we should all do the same, right? Wrong! 
  
  * Why? Because, at scale, a monorepo must solve every problem that a polyrepo must solve, with the downside of encouraging tight coupling, and the additional herculean effort of tackling VCS scalability. 
  
  * Thus, in the medium to long term, a monorepo provides zero organizational benefits, while inevitably leaving some of an organization’s best engineers with a wicked case of PTSD (manifested via drooling and incoherent mumbling about git performance internals).




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
 
  * Google monorepo: yes, and this is the default way to work

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

