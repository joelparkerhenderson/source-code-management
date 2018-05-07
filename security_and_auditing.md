# Security and auditing

Security for source code management is especially important for areas such as industry auditing, goverment regulation, sensitive codebases, and high-security applications, and legal needs for [SOX](https://en.wikipedia.org/wiki/Sarbanes%E2%80%93Oxley_Act), [PCI](https://en.wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard), [HIPAA](https://en.wikipedia.org/wiki/Health_Insurance_Portability_and_Accountability_Act), [FERPA](https://en.wikipedia.org/wiki/Family_Educational_Rights_and_Privacy_Act), [GDPR](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation), etc.

* [](*)


## Best practices


Key concepts:

* Establish a clear policy and also a clear process. 

* Use auditing, automation, encryption, monitoring, etc.

* Ensure all areas are covered by multiple responsible people.



### Least Privilege

All repositories are restricted by default, only users who need access have access, and read/write permissions are set depending on their role. Just because they are an employee, doesn't necessarily mean they need access to the product's source code.


### Branch Protection

The master branch and all release branches are restricted: no one can push directly to them. 

Disable all ways to change history, such as force push.


### Sign commits 

Use auditing & second factor trust. Can be a bit complicated/annoying to setup for a whole org though.


### Automate builds

Use continuous integration (CI) automation that automatically triggers on relevant events, such as when a developer creates a pull request.

Run test suites & static analysis to validate code. Only code successfully passing both can be merged.

Comment: Jenkins works great when used with an automation user for accessing protected resources.

Comment: If you want to use Jenkins and want to separate out production builds/deploys from dev builds/deploys, then you have a couple options, and oth of these solutions can be paired with separate credentials for each environment: 1) Separate Jenkins servers per environment, 2) Single Jenkins master (use the folder plugin for restricting user access per environment, or use dedicated build slaves for each environment e.g. prod jobs run only on prod slave. 


### Automate versioning

Automated versioning can be a bit complicated. 

We use something like CapsuleCD for versioning our libraries (pip/chef cookbooks/etc) automatically, but our core applications are automatically versioned using SNAPSHOT/SHA versions and relative tags (ie. git describe --long). Version files are embedded inside the artifacts and the version string is also embedded in the filename.

Comment: We use [Artifactory](https://jfrog.com/artifactory/) as our central artifact storage, though I've heard good things about [Nexus](https://www.sonatype.com/nexus-repository-sonatype) as well.


### Automate deployments

Production deployments are done in a completely automated fashion. 

Continuous Deployment or Continuous Delivery is your friend. CD helps for auditing, and means there's less chance for a developer/operations member to fat finger something.


### DevSecOps

Automated security and vulnerability scanning is done against every PR. 

This includes source code analysis as well as dependency vulnerability analysis.

The awesome-devsecops page has a good list of various tools that you can integrate with your existing test suites.



### Force code reviews

In addition to doing code reviews, if you use Github/Enterprise Github, a `CODEOWNERS` file can be used to ensure that relevant developers are notified whenever certain areas of the codebase are touched.


### Create immutable release artifacts

All builds/releases/etc that generate deployable artifacts are versioned and uploaded to a centralized artifact storage system. 

These versioned artifacts are concrete and cannot be change/updated once created. To update your artifact a new version must be created.


### Use annotated tags 

Use annotated tags rather than lightweight tags if you use Git. 

Annotated tags create a separate commit in the repo history, and can include user information. Also can be signed, and should never be moved.


### Rotate user credentials

All user credentials used to access systems are required to be rotated on a schedule, including automation users & ssh keys.


### Protect secrets 

Production Secrets are not accessible to developers. They are created/rotated by Ops, and developers can point to a dev vault/secret storage system during development/testing. Only automation has access to production secrets (which are also rotated).


### One-way logging/monitoring

In addition to enabling monitoring & logging for your system, make sure that your logs and metrics are immediately exported into a append-only system. Basically a bad actor should never be able to compromise your system and be able to delete any trace of their actions.


### Build system security 

If you're following Devops best practices, you'll probably be running a system like Concourse or Jenkins. You'll want to ensure that you keep those systems up-to-date with 0-day security patches and completely lock down access to the actual machine. A compromised Build system means all your code security is moot.


### Binary Reproducible Builds 

This one isn't necessarily a best practice that makes sense for all organizations, however if you're in a heavily audited industry(finance/meditech/etc), binary reproducible builds can be useful. Google's [Bazel](https://bazel.build/) might be worth looking into.


### Audit

Have owners for each project, such as each GitHub project, BitBucket project, etc.

Ensure there are at least two project admins for each project.

Inform participants where to find the admins, in order to get a repository created, or a permission update, etc.


### Use encryption

Use encryption on transport for all the things 

* Use nfs v4 for BitBucket Data Center with kerberos auth, where nfs is the only supported protocol 

* Use TLS 1.2 on db connections to/from BitBucket, postgres is my favorite for this

* BitBucket has Personal Tokens for automation as of 5.5+. This goes a long way towards build user passwords not getting passed around

* Users should use a password protected private SSH key - using keychain and jenkins credential stores to store the password for faceless automation users

* A lot of content handling rules (like H-to-the-I-to-the-P-to-the-AA) require encryption on disk (where possible) - a rule I have never been a fan of, as it seems pretty freaking ridiculous that someone would walk into a server room and lift the hard drive, but if you are running Server (not Data Center), you can usually rely on LUKS

* Do not make automation users with passwords shared between 8 different build systems administrators on projects... this is bad automation, but if one must, separate projects used by people from the robots

* Make sure to put SSL/TLS on the Elasticsearch connection (using the buckler plugin) for BitBucket Data Center

* As long as you are on a newer version of BitBucket Server, I think the embedded for standalone binds to localhost only, but make sure your index is not exposed, or you are excluding anything that should not go over the wire


### Regulator controls

Regulatory controls are a bit tricky to tie directly back to actual best practices - they're generally written with language such as "there must be a traceable log of all changes done to this system". That of course can be accomplished by a plain 'ol ticketing system, or by a git log on a repo, provided there is documentation on our end explaining how our process implements that required control.


### Regulatory controls vs. DevOps

You may have to take a slightly anti-devops stance if your services are sensitive. In my work, one cast-iron rule is that there is no overlap between the people writing the code and the people deploying the code.

This essentially means Ops/Systems have no write access to code repositories or ability to build code, and developers have no access to the systems or data in production. It sounds like some sort of historical horror story in the light of modern practices, but it may unfortunately be necessary to gain regulatory approval (which you don't mention; are you subject to any?). It also mandates the existence of development and staging environments which are fit for purpose.

There is also separation of privileges, which means people who grant access are never the same ones who use that access. This is easily done in large corporate entities with central IT departments who can have that delegated to them, but it may be a challenge in a smaller organization.

Use auditing so if people try to abuse the system, then there is a record of it and someone (or some system) is analyzing it to alert for unexpected behaviour.


### DevOps Handbook

The DevOps Handbook has some good stuff on these topics, in addition to what others here have said.

https://smile.amazon.com/DevOps-Handbook-World-Class-Reliability-Organizations/dp/1942788002/ref=sr_1_1


## Implementations


### By zieziegabor

https://www.reddit.com/user/zieziegabor

We basically do what the Linux kernel does, with Signed-Off by: <>, except we require those to also be GPG signed(along with every regular commit also gpg signed). Jenkins(really a Makefile ran by jenkins) verifies at least 2 developer GPG signatures are used for every commit into trunk/MASTER. We white-list the GPG keys allowed to be used for signatures.

Every commit user gets their own branch repo. They do whatever they want there, but we force every commit to be GPG signed. When they are ready for it to be merged, they poke another person with commit access to review and commit to the main branch repo. This also is GPG signed.

This gives our developers the ability to use whatever tool(s) they want to do code-reviews, and we are not forcing anyone to use any particular tool (mostly we us kiln's code review tool, however). You just have to GPG sign your commits, and 2 diff. committers have to gpg sign commits to get it into the built product.

In HG we do this by using both GPG modules (the built in one for regular commits and the 3rd party one for sign offs).


## Wordbook


### Tools

* [Artifactory by JFrog](https://jfrog.com/artifactory/): Enterprise Universal Artifact Manager
* [Awesome DevSecOps](https://github.com/devsecops/awesome-devsecops)
* [Bazel by Google](https://bazel.build/): Build and test software of any size, quickly and reliably.
* [CapsuleCD](https://github.com/AnalogJ/capsulecd): a generic Continuous Delivery pipeline for versioned artifacts and libraries written in any language.
* [Nexus Repository by Sonatype](https://www.sonatype.com/nexus-repository-sonatype): Expert flow control for binaries, build artifacts, and release candidates.

### Security laws

* [FERPA: Family Educational Rights and Privacy Act](https://en.wikipedia.org/wiki/Family_Educational_Rights_and_Privacy_Act)
* [GDPR: General Data Protection Regulation](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation)
* [HIPAA: Health Insurance Portability and Accountability Act](https://en.wikipedia.org/wiki/Health_Insurance_Portability_and_Accountability_Act)
* [PCI: Payment Card Industry Data Security Standard](https://en.wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)
* [SOX: Sarbanes Oxley Act](https://en.wikipedia.org/wiki/Sarbanes%E2%80%93Oxley_Act)


## Related

Related links:

* [GitHub special files and paths](https://github.com/joelparkerhenderson/github_special_files_and_paths)

* [When it comes to managing repositories for highly sensitive/regulated code bases, what are some best practices that can be implemented with regards to code reviews, approvals, and deployment to production?](https://www.reddit.com/r/devops/comments/8gmf5m/when_it_comes_to_managing_repositories_for_highly/)

* Special thanks to https://www.reddit.com/user/analogj

* [Defense Information Systems Agency (DISA)](https://disa.mil/)
  * [Information Assurance Support Environment (IASE)](https://iase.disa.mil/)
    * [Security Technical Implementation Guides (STIGs)](https://iase.disa.mil/stigs)
      * [DISA IASE STIGs Master List](https://iase.disa.mil/stigs/Pages/a-z.aspx)



