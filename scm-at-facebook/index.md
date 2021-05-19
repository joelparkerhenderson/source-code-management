# SCM at Facebook

Facebook engineers historically see four repos: 

  * www
  * fbobjc
  * fbandroid
  * fbcode (C++, Java, Thrift services, etc)

As of 2017 Facebook uses one `hg` repo (www as a possible exception, I'm unsure). The "sparse checkout" machinery disguises it, but for engineers working cross platform (e.g. React Native) it's routine to make commits that span platforms.

At one point these repos were Git but for various reasons ended up being migrated to Mercurial some years ago.

FB uses GraphQL at a client level and Thrift at the service level.

So one pain point is that, for example, you can modify a GraphQL endpoint in one repo but its used by clients in others (ie mobile clients). There are lots of warnings about making backward-incompatible changes, some of them excessively pessimistic because deterministically showing something will break some mobile build in another repo is hard.
