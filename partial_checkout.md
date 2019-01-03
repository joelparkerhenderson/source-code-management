# Partial check out

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
