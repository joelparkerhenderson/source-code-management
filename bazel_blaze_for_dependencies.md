# Bazel/Blaze for dependencies

Bazel solves the CI/CD dependencies for mono-repos.

  * All build targets (buildable units) explicitly define their direct dependencies. 

  * If you modify something, you can then run all tests for all units that transitively depend on your change.

  * Note: At Google, Bazel is known internally as Blaze.

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

