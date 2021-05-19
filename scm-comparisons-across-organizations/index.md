# SCM comparisons across organizations


## Facebook vs. Google

By [cletus](https://news.ycombinator.com/user?id=cletus)

Related:

* [SCM at Facebook](scm_at_facebook.md)
* [SCM at Google](scm_at_google.md)

At Facebook, you can modify a GraphQL endpoint in one repo, but its used by clients in others (ie mobile clients). There are lots of warnings about making backward-incompatible changes, some of them excessively pessimistic because deterministically showing something will break some mobile build in another repo is hard.

At Google, Google3 has less of these problems because the code is in the same repo. On top of that, Google has spent a vast amount of effort making it so the same build and caching systems can handle C++ server code as well as Objective-C iOS app code. Basically if you're working on Google3 you basically compile very little to nothing locally.

Engineers on Android, Chrome and ChromeOS however compile a lot of things locally and thus get far beefier workstations.

At FB the mobile build system doesn't seem to be as advanced in that there is a far higher proportion of local building.

Just to stress, the above is just my personal experience and I hope it's taken as intended: general observations rather than complaints and definitely not arguing that one is objectively better than the other. There are simply tradeoffs.


## How long does a commit with tests and checks take?

Related:

* [SCM at Facebook](scm_at_facebook.md)
* [SCM at Google](scm_at_google.md)

Google:

  * Unit tests finish relatively fast.

  * Integration tests that bring up server tend to be slow (30-40 mins best case almost for these for projects im working on). 

  * Mobile tests that run on emulators are the worst.

  * The cost of testing gets amortized two ways: you can run immediate unit tests manually on command line as the fastest signal. Then when you send for code review, presubmit runs. During code reviews you may choose to run them as you go. Eventually when you submit they run again. If there has not been any changes to your commit/cl and there is an already passing submit, it will just skip and submit.

Twitter: 

  * Generally submit queue takes 10-20 minutes, longer if you're changing a core library.

  * The larger factor is what kind of test has to run. Feature tests can take a while on CI if you're spinning up lots of embedded services (dependent services, MySQL, storage layers, etc).

