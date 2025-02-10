---
title: Threats & mitigations
description: A comprehensive technical analysis of supply chain threats and their corresponding mitigations in SLSA.
---

What follows is a comprehensive technical analysis of supply chain threats and
their corresponding mitigations in SLSA. For an introduction to the
supply chain threats that SLSA is aiming to protect against, see [Supply chain threats].

The examples on this page are meant to:

-   Explain the reasons for each of the SLSA [requirements](requirements.md).
-   Increase confidence that the SLSA requirements are sufficient to achieve the
    desired [level](levels.md) of integrity protection.
-   Help implementers better understand what they are protecting against so that
    they can better design and implement controls.

**TODO:** Update the ordering to match the diagram. We're currently in the
middle of refactoring, with a jumble of new and old.

**TODO:** Expand this threat model to also cover "unknowns". Not sure if that is
a "threat" or a "risk". Example: If libFoo is compromised, how do you know if
you are compromised? At a first level, if you don't even know whether you
include libFoo or not, that's a big risk. But even then, it might be that you
don't use libFoo in a way that makes your product vulnerable. We should capture
that somehow. This isn't specific to dependencies - it applies to the entire
diagram.
([discussion](https://github.com/slsa-framework/slsa/pull/1046/files/ebf34a8f9e874b219f152bad62673eae0b3ba2c3#r1585440922))

**TODO:** Revisit and possibly update any item that says "out of scope", since
we want to really expand SLSA's scope to include all of these threats. A more
nuanced answer would be that the current version does not define a specific
mitigation, but it's in scope for the project overall. We can also list specific
mitigations even if they're not listed as a level requirement.

<article class="threats">

## Overview

![Supply Chain Threats](images/supply-chain-threats.svg)

This threat model covers the *software supply chain*, meaning the process by
which software is produced and consumed. We describe and cluster threats based
on where in the software development pipeline those threats occur, labeled (A)
through (I). This is useful because priorities and mitigations mostly cluster
along those same lines. Keep in mind that dependencies are
[highly recursive](#dependency-threats), so each dependency has its own threats
(A) through (I), and the same for *their* dependencies, and so on. For a more
detailed explanation of the supply chain model, see
[Terminology](terminology.md).

Importantly, producers and consumers face *aggregate* risk across all of the
software they produce and consume, respectively. Many organizations produce
and/or consume thousands of software packages, both first- and third-party, and
it is not practical to rely on every individual team in the organization to do
the right thing. For this reason, SLSA prioritizes mitigations that can be
broadly adopted in an automated fashion, minimizing the chance of mistakes.

## Source threats

A source integrity threat is a potential for an adversary to introduce a change
to the source code that does not reflect the intent of the software producer.
This includes the threat of an authorized individual introducing an unauthorized
change---in other words, an insider threat.

SLSA v1.0 does not address source threats, but we anticipate doing so in a
[future version](current-activities#source-track). In the meantime, the
threats and potential mitigations listed here show how SLSA v1.0 can fit into a
broader supply chain security program.

### (A) Producer

The producer of the software intentionally produces code that harms the
consumer, or the producer otherwise uses practices that are not deserving of the
consumer's trust.

Threats in this category likely *cannot* be mitigated through controls placed
during the authoring/reviewing process, in contrast with (B).

**TODO:** The difference between (A) and (B) is still a bit fuzzy, which would
be nice to resolve. For example, compromised developer credentials - is that (A)
or (B)?

<details><summary>Software producer intentionally submits bad code</summary>

*Threat:* Software producer intentionally submits "bad" code, following all
proper processes.

*Mitigation:* **TODO**

*Example:* A popular extension author sells the rights to a new owner, who then
modifies the code to secretly mine cryptocurrency at the users' expense. SLSA
does not protect against this, though if the extension were open source, regular
auditing may discourage this from happening.

</details>

**TODO:** More producer threats? Perhaps the attack to xz where a malicious
contributor gained enhanced privileges through social engineering?

### (B) Authoring & reviewing

An adversary introduces a change through the official source control management
interface without any special administrator privileges.

Threats in this category *can* be mitigated by code review or some other
controls during the authoring/reviewing process, at least in theory. Contrast
this with (A), where such controls are likely ineffective.

#### (B1) Submit change without review

<details><summary>Directly submit without review</summary>

*Threat:* Submit bad code to the source repository without another person
reviewing.

*Mitigation:* Source repository requires two-person approval for all changes.

*Example:* Adversary directly pushes a change to a GitHub repo's `main` branch.
Solution: Configure GitHub's "branch protection" feature to require pull request
reviews on the `main` branch.

</details>
<details><summary>Review own change through a sock puppet account</summary>

*Threat:* Propose a change using one account and then approve it using another
account.

*Mitigation:* Source repository requires approval from two different, trusted
persons. If the proposer is trusted, only one approval is needed; otherwise two
approvals are needed. The software producer maps accounts to trusted persons.

*Example:* Adversary creates a pull request using a secondary account and then
approves and merges the pull request using their primary account. Solution:
Configure branch protection to require two approvals and ensure that all
repository contributors and owners map to unique persons.

</details>
<details><summary>Use a robot account to submit change</summary>

*Threat:* Exploit a robot account that has the ability to submit changes without
two-person review.

*Mitigation:* All changes require two-person review, even changes authored by
robots.

*Example:* A file within the source repository is automatically generated by a
robot, which is allowed to submit without review. Adversary compromises the
robot and submits a malicious change without review. Solution: Require human
review for these changes.

> TODO([#196](https://github.com/slsa-framework/slsa/issues/196)) This solution
> may not be practical. Should there be an exception for locked down robot
> accounts?

</details>
<details><summary>Abuse review exceptions</summary>

*Threat:* Exploit a review exception to submit a bad change without review.

*Mitigation:* All changes require two-person review without exception.

*Example:* Source repository requires two-person review on all changes except
for "documentation changes," defined as only touching files ending with `.md` or
`.html`. Adversary submits a malicious executable named `evil.md` without review
using this exception, and then builds a malicious package containing this
executable. This would pass the policy because the source repository is correct,
and the source repository does require two-person review. Solution: Do not allow
such exceptions.

> TODO This solution may not be practical in all circumstances. Are there any
> valid exceptions? If so, how do we ensure they cannot be exploited?

</details>

#### (B2) Evade code review requirements

<details><summary>Modify code after review</summary>

*Threat:* Modify the code after it has been reviewed but before submission.

*Mitigation:* Source control platform invalidates approvals whenever the
proposed change is modified.

*Example:* Source repository requires two-person review on all changes.
Adversary sends a "good" pull request to a peer, who approves it. Adversary then
modifies it to contain "bad" code before submitting. Solution: Configure branch
protection to dismiss stale approvals when new changes are pushed.

> Note: This is not currently a SLSA requirement because the productivity hit is
> considered too great to outweigh the security benefit. The cost of code review
> is already too high for most projects, given current code review tooling, so
> making code review even costlier would not further our goals. However, this
> should be considered for future SLSA revisions once the state-of-the-art for
> code review has improved and the cost can be minimized.

</details>
<details><summary>Submit a change that is unreviewable</summary>

*Threat:* Send a change that is meaningless for a human to review that looks
benign but is actually malicious.

*Mitigation:* Code review system ensures that all reviews are informed and
meaningful.

*Example:* A proposed change updates a file, but the reviewer is only presented
with a diff of the cryptographic hash, not of the file contents. Thus, the
reviewer does not have enough context to provide a meaningful review. Solution:
the code review system should present the reviewer with a content diff or some
other information to make an informed decision.

</details>
<details><summary>Copy a reviewed change to another context</summary>

*Threat:* Get a change reviewed in one context and then transfer it to a
different context.

*Mitigation:* Approvals are context-specific.

*Example:* MyPackage's source repository requires two-person review. Adversary
forks the repo, submits a change in the fork with review from a colluding
colleague (who is not trusted by MyPackage), then merges the change back into
the upstream repo. Solution: The merge should still require review, even though
the fork was reviewed.

</details>
<details><summary>Compromise another account</summary>

*Threat:* Compromise one or more trusted accounts and use those to submit and
review own changes.

*Mitigation:* Source control platform verifies two-factor authentication, which
increases the difficulty of compromising accounts.

*Example:* Trusted person uses a weak password on GitHub. Adversary guesses the
weak password, logs in, and pushes changes to a GitHub repo. Solution: Configure
GitHub organization to requires 2FA for all trusted persons. This would increase
the difficulty of using the compromised password to log in to GitHub.

</details>
<details><summary>Hide bad change behind good one</summary>

*Threat:* Request review for a series of two commits, X and Y, where X is bad
and Y is good. Reviewer thinks they are approving only the final Y state whereas
they are also implicitly approving X.

*Mitigation:* Only the version that is actually reviewed is the one that is
approved. Any intermediate revisions don't count as being reviewed.

*Example:* Adversary sends a pull request containing malicious commit X and
benign commit Y that undoes X. In the pull request UI, reviewer only reviews and
approves "changes from all commits", which is a delta from HEAD to Y; they don't
see X. Adversary then builds from the malicious revision X. Solution: Policy
does not accept this because the version X is not considered reviewed.

> TODO This is implicit but not clearly spelled out in the requirements. We
> should consider clarifying if there is confusion or incorrect implementations.

</details>

#### (B3) Render code review ineffective

<details><summary>Collude with another trusted person</summary>

*Threat:* Two trusted persons collude to author and approve a bad change.

*Mitigation:* **Outside the scope of SLSA.** We use "two trusted persons" as a
proxy for "intent of the software producer".

</details>
<details><summary>Trick reviewer into approving bad code</summary>

*Threat:* Construct a change that looks benign but is actually malicious, a.k.a.
"bugdoor."

*Mitigation:* **Outside the scope of SLSA.**

</details>
<details><summary>Reviewer blindly approves changes</summary>

*Threat:* Reviewer approves changes without actually reviewing, a.k.a. "rubber
stamping."

*Mitigation:* **Outside the scope of SLSA.**

</details>

### (C) Source code management

An adversary introduces a change to the source control repository through an
administrative interface, or through a compromise of the underlying
infrastructure.

<details><summary>Project owner bypasses or disables controls</summary>

*Threat:* Trusted person with "admin" privileges in a repository submits "bad"
code bypassing existing controls.

*Mitigation:* All persons are subject to same controls, whether or not they have
administrator privileges. Disabling the controls requires two-person review (and
maybe notifies other trusted persons?)

*Example 1:* GitHub project owner pushes a change without review, even though
GitHub branch protection is enabled. Solution: Enable the "Include
Administrators" option for the branch protection.

*Example 2:* GitHub project owner disables "Include Administrators", pushes a
change without review, then re-enables "Include Administrators". This currently
has no solution on GitHub.

> TODO This is implicit but not clearly spelled out in the requirements. We
> should consider clarifying since most if not all existing platforms do not
> properly address this threat.

</details>
<details><summary>Platform admin abuses privileges</summary>

*Threat:* Platform administrator abuses their privileges to bypass controls or
to push a malicious version of the software.

*Mitigation:* TODO

*Example 1:* GitHostingService employee uses an internal tool to push changes to
the MyPackage source repo.

*Example 2:* GitHostingService employee uses an internal tool to push a
malicious version of the server to serve malicious versions of MyPackage sources
to a specific CI/CD client but the regular version to everyone else, in order to
hide tracks.

*Example 3:* GitHostingService employee uses an internal tool to push a
malicious version of the server that includes a backdoor allowing specific users
to bypass branch protections. Adversary then uses this backdoor to submit a
change to MyPackage without review.

</details>
<details><summary>Exploit vulnerability in SCM</summary>

*Threat:* Exploit a vulnerability in the implementation of the source code
management system to bypass controls.

*Mitigation:* **Outside the scope of SLSA.**

</details>

### (D) External build parameters

**TODO:** Move under "Build threats".

An adversary builds from a version of the source code that does not match the
official source control repository, or changes the build parameters to inject
behavior that was not intended by the official source.

The mitigation here is to compare the provenance against expectations for the
package, which depends on SLSA Build L1 for provenance. (Threats against the
provenance itself are covered by (E) and (F).)

<details><summary>Build from unofficial fork of code <span>(expectations)</span></summary>

*Threat:* Build using the expected CI/CD process but from an unofficial fork of
the code that may contain unauthorized changes.

*Mitigation:* Verifier requires the provenance's source location to match an
expected value.

*Example:* MyPackage is supposed to be built from GitHub repo `good/my-package`.
Instead, it is built from `evilfork/my-package`. Solution: Verifier rejects
because the source location does not match.

</details>
<details><summary>Build from unofficial branch or tag <span>(expectations)</span></summary>

*Threat:* Build using the expected CI/CD process and source location, but
checking out an "experimental" branch or similar that may contain code not
intended for release.

*Mitigation:* Verifier requires that the provenance's source branch/tag matches
an expected value, or that the source revision is reachable from an expected
branch.

*Example:* MyPackage's releases are tagged from the `main` branch, which has
branch protections. Adversary builds from the unprotected `experimental` branch
containing unofficial changes. Solution: Verifier rejects because the source
revision is not reachable from `main`.

</details>
<details><summary>Build from unofficial build steps <span>(expectations)</span></summary>

*Threat:* Build the package using the proper CI/CD platform but with unofficial
build steps.

*Mitigation:* Verifier requires that the provenance's build configuration source
matches an expected value.

*Example:* MyPackage is expected to be built by Google Cloud Build using the
build steps defined in the source's `cloudbuild.yaml` file. Adversary builds
with Google Cloud Build, but using custom build steps provided over RPC.
Solution: Verifier rejects because the build steps did not come from the
expected source.

</details>
<details><summary>Build from unofficial parameters <span>(expectations)</span></summary>

*Threat:* Build using the expected CI/CD process, source location, and
branch/tag, but using a parameter that injects unofficial behavior.

*Mitigation:* Verifier requires that the provenance's external parameters all
match expected values.

*Example 1:* MyPackage is supposed to be built from the `release.yml` workflow.
Adversary builds from the `debug.yml` workflow. Solution: Verifier rejects
because the workflow parameter does not match the expected value.

*Example 2:* MyPackage's GitHub Actions Workflow uses `github.event.inputs` to
allow users to specify custom compiler flags per invocation. Adversary sets a
compiler flag that overrides a macro to inject malicious behavior into the
output binary. Solution: Verifier rejects because the `inputs` parameter was not
expected.

</details>
<details><summary>Build from modified version of code modified after checkout <span>(expectations)</span></summary>

*Threat:* Build from a version of the code that includes modifications after
checkout.

*Mitigation:* Build platform pulls directly from the source repository and
accurately records the source location in provenance.

*Example:* Adversary fetches from MyPackage's source repo, makes a local commit,
then requests a build from that local commit. Builder records the fact that it
did not pull from the official source repo. Solution: Verifier rejects because
the source repo does not match the expected value.

</details>

## Dependency threats

**TODO:** Move after Usage Threats.

A dependency threat is a potential for an adversary to introduce unintended
behavior in one artifact by compromising some other artifact that the former
depends on at build time. (Runtime dependencies are excluded from the model, as
[noted below](#runtime-dep).)

Unlike other threat categories, dependency threats develop recursively through
the supply chain and can only be exploited indirectly. For example, if
application *A* includes library *B* as part of its build process, then a build
or source threat to *B* is also a dependency threat to *A*. Furthermore, if
library *B* uses build tool *C*, then a source or build threat to *C* is also a
dependency threat to both *A* and *B*.

This version of SLSA does not explicitly address dependency threats, but we
expect that a future version will. In the meantime, you can [apply SLSA
recursively] to your dependencies in order to reduce the risk of dependency
threats.

-   **TODO:** Should we distinguish 1P vs 3P boundaries in the diagram, or
    otherwise visualize 1P/3P?
-   **TODO:** Expand to cover typosquatting, dependency confusion, and other
    "dependency" threats.
-   **TODO:** The word "compromised" is a bit too restrictive. If the publisher
    intends to do harm, either because they tricked you into using a dependency
    (typosquatting or dependency confusion), or because they were good and now
    do something bad, that's not really "compromised" per se.
-   **TODO:** Should we expand this to cover "transitive SLSA verification"?
-   **TODO:** Update the Terminology page to show "build time" vs "runtime",
    since the latter term results in confusion. Also consider the term "deploy
    time" as an alternative.

[apply SLSA recursively]: verifying-artifacts.md#step-3-optional-check-dependencies-recursively

### Build dependency

An adversary compromises the target artifact through one of its build
dependencies. Any artifact that is present in the build environment and has the
ability to influence the output is considered a build dependency.

<details id="included-dep"><summary>Include a vulnerable dependency (library, base image, bundled file, etc.)</summary>

*Threat:* Statically link, bundle, or otherwise include an artifact that is
compromised or has some vulnerability, causing the output artifact to have the
same vulnerability.

*Example:* The C++ program MyPackage statically links libDep at build time. A
contributor accidentally introduces a security vulnerability into libDep. The
next time MyPackage is built, it picks up and includes the vulnerable version of
libDep, resulting in MyPackage also having the security vulnerability.

*Mitigation:* **TODO**

</details>
<details id="build-tool"><summary>Use a compromised build tool (compiler, utility, interpreter, OS package, etc.)</summary>

*Threat:* Use a compromised tool or other software artifact during the build
process, which alters the build process and injects unintended behavior into the
output artifact.

*Example:* MyPackage is a tarball containing an ELF executable, created by
running `/usr/bin/tar` during its build process. An adversary compromises the
`tar` OS package such that `/usr/bin/tar` injects a backdoor into every ELF
executable it writes. The next time MyPackage is built, the build picks up the
vulnerable `tar` package, which injects the backdoor into the resulting
MyPackage artifact.

*Mitigation:* **TODO**

</details>

Reminder: dependencies that look like [runtime dependencies](#runtime-dep)
actually become build dependencies if they get loaded at build time.

<details id="runtime-dep-at-build-time"><summary>Use a compromised runtime dependency during the build (for tests, dynamic linking, etc.)</summary>

*Threat:* During the build process, use a compromised runtime dependency (such
as during testing or dynamic linking), which alters the build process and
injects unwanted behavior into the output.

**NOTE:** This is technically the same case as [Use a compromised build
tool](#build-tool). We call it out to remind the reader that
[runtime dependencies](#runtime-dep) can become build dependencies if they are
loaded during the build.

*Example:* MyPackage has a runtime dependency on package Dep, meaning that Dep
is not included in MyPackage but required to be installed on the user's machine
at the time MyPackage is run. However, Dep is also loaded during the build
process of MyPackage as part of a test. An adversary compromises Dep such that,
when run during a build, it injects a backdoor into the output artifact. The
next time MyPackage is built, it picks up and loads Dep during the build
process. The malicious code then injects the backdoor into the new MyPackage
artifact.

*Mitigation:* In addition to all the mitigations for build tools, you can often
avoid runtime dependencies becoming build dependencies by isolating tests to a
separate environment that does not have write access to the output artifact.

</details>

### Related threats

The following threats are related to "dependencies" but are not modeled as
"dependency threats".

<details id="runtime-dep"><summary>Use a compromised dependency at runtime <span>(modeled separately)</span></summary>

*Threat:* Load a compromised artifact at runtime, thereby compromising the user
or environment where the software ran.

*Example:* MyPackage lists package Dep as a runtime dependency. Adversary
publishes a compromised version of Dep that runs malicious code on the user's
machine when Dep is loaded at runtime. An end user installs MyPackage, which in
turn installs the compromised version of Dep. When the user runs MyPackage, it
loads and executes the malicious code from Dep.

*Mitigation:* N/A - This threat is out of scope of SLSA. SLSA's threat model
does not explicitly model runtime dependencies. Instead, each runtime dependency
is considered a distinct artifact with its own threats.

</details>

## Build threats

A build integrity threat is a potential for an adversary to introduce behavior
to an artifact without changing its source code, or to build from a
source, dependency, and/or process that is not intended by the software
producer.

The SLSA Build track mitigates these threats when the consumer
[verifies artifacts](verifying-artifacts.md) against expectations, confirming
that the artifact they received was built in the expected manner.

### (E) Build process

An adversary introduces an unauthorized change to a build output through
tampering of the build process; or introduces false information into the
provenance.

These threats are directly addressed by the SLSA Build track.

<details><summary>Forge values of the provenance (other than output digest) <span>(Build L2+)</span></summary>

*Threat:* Generate false provenance and get the trusted control plane to sign
it.

*Mitigation:* At Build L2+, the trusted control plane [generates][authentic] all
information that goes in the provenance, except (optionally) the output artifact
hash. At Build L3+, this is [hardened][unforgeable] to prevent compromise even
by determined adversaries.

*Example 1 (Build L2):* Provenance is generated on the build worker, which the
adversary has control over. Adversary uses a malicious process to get the build
platform to claim that it was built from source repo `good/my-package` when it
was really built from `evil/my-package`. Solution: Builder generates and signs
the provenance in the trusted control plane; the worker reports the output
artifacts but otherwise has no influence over the provenance.

*Example 2 (Build L3):* Provenance is generated in the trusted control plane,
but workers can break out of the container to access the signing material.
Solution: Builder is hardened to provide strong isolation against tenant
projects.

</details>
<details id="forged-digest"><summary>Forge output digest of the provenance <span>(n/a)</span></summary>

*Threat:* The tenant-controlled build process sets output artifact digest
(`subject` in SLSA Provenance) without the trusted control plane verifying that
such an artifact was actually produced.

*Mitigation:* None; this is not a problem. Any build claiming to produce a given
artifact could have actually produced it by copying it verbatim from input to
output.[^preimage] (Reminder: Provenance is only a claim that a particular
artifact was *built*, not that it was *published* to a particular registry.)

*Example:* A legitimate MyPackage artifact has digest `abcdef` and is built
from source repo `good/my-package`. A malicious build from source repo
`evil/my-package` claims that it built artifact `abcdef` when it did not.
Solution: Verifier rejects because the source location does not match; the
forged digest is irrelevant.

[^preimage]: Technically this requires the artifact to be known to the
    adversary. If they only know the digest but not the actual contents, they
    cannot actually build the artifact without a [preimage attack] on the digest
    algorithm. However, even still there are no known concerns where this is a
    problem.

[preimage attack]: https://en.wikipedia.org/wiki/Preimage_attack

</details>
<details><summary>Compromise project owner <span>(Build L2+)</span></summary>

*Threat:* An adversary gains owner permissions for the artifact's build project.

*Mitigation:* The build project owner must not have the ability to influence the
build process or provenance generation.

*Example:* MyPackage is built on Awesome Builder under the project "mypackage".
Adversary is an administrator of the "mypackage" project. Awesome Builder allows
administrators to debug build machines via SSH. An adversary uses this feature
to alter a build in progress.

</details>
<details><summary>Compromise other build <span>(Build L3)</span></summary>

*Threat:* Perform a malicious build that alters the behavior of a benign
build running in parallel or subsequent environments.

*Mitigation:* Builds are [isolated] from one another, with no way for one to
affect the other or persist changes.

*Example 1:* A build platform runs all builds for project MyPackage on
the same machine as the same Linux user. An adversary starts a malicious build
that listens for another build and swaps out source files, then starts a benign
build. The benign build uses the malicious build's source files, but its
provenance says it used benign source files. Solution: The build platform
changes architecture to isolate each build in a separate VM or similar.

*Example 2:* A build platform uses the same machine for subsequent
builds. An adversary first runs a build that replaces the `make` binary with a
malicious version, then subsequently runs an otherwise benign build. Solution:
The builder changes architecture to start each build with a clean machine image.

</details>
<details><summary>Steal cryptographic secrets <span>(Build L3)</span></summary>

*Threat:* Use or exfiltrate the provenance signing key or some other
cryptographic secret that should only be available to the build platform.

*Mitigation:* Builds are [isolated] from the trusted build platform control
plane, and only the control plane has [access][unforgeable] to cryptographic
secrets.

*Example:* Provenance is signed on the build worker, which the adversary has
control over. Adversary uses a malicious process that generates false provenance
and signs it using the provenance signing key. Solution: Builder generates and
signs provenance in the trusted control plane; the worker has no access to the
key.

</details>
<details><summary>Poison the build cache <span>(Build L3)</span></summary>

*Threat:* Add a malicious artifact to a build cache that is later picked up by a
benign build process.

*Mitigation:* Build caches must be [isolate][isolated] between builds to prevent
such cache poisoning attacks.

*Example:* Build platform uses a build cache across builds, keyed by the hash of
the source file. Adversary runs a malicious build that creates a "poisoned"
cache entry with a falsified key, meaning that the value wasn't really produced
from that source. A subsequent build then picks up that poisoned cache entry.

</details>
<details><summary>Compromise build platform admin <span>(verification)</span></summary>

*Threat:* An adversary gains admin permissions for the artifact's build platform.

*Mitigation:* The build platform must have controls in place to prevent and
detect abusive behavior from administrators (e.g. two-person approvals, audit
logging).

*Example:* MyPackage is built on Awesome Builder. Awesome Builder allows
engineers on-call to SSH into build machines to debug production issues. An
adversary uses this access to modify a build in progress. Solution: Consumers
do not accept provenance from the build platform unless they trust sufficient
controls are in place to prevent abusing admin privileges.

</details>

### (F) Artifact publication

An adversary uploads a package artifact that does not reflect the intent of the
package's official source control repository.

This is the most direct threat because it is the easiest to pull off. If there
are no mitigations for this threat, then (D) and (E) are often indistinguishable
from this threat.

**TODO:** We need to define "official source control repository". Its meaning is
not obvious. The gist is that each package theoretically has some "official" or
"canonical" repository from which it "should" be built, and the attack here is
that you either build from a different source repository or otherwise do
something that doesn't reflect that source repository. But we need to nail down
this concept.

<details><summary>Build with untrusted CI/CD <span>(expectations)</span></summary>

*Threat:* Build using an unofficial CI/CD pipeline that does not build in the
correct way.

*Mitigation:* Verifier requires provenance showing that the builder matched an
expected value.

*Example:* MyPackage is expected to be built on Google Cloud Build, which is
trusted up to Build L3. Adversary builds on SomeOtherBuildPlatform, which is only
trusted up to Build L2, and then exploits SomeOtherBuildPlatform to inject
malicious behavior. Solution: Verifier rejects because builder is not as
expected.

</details>
<details><summary>Upload package without provenance <span>(Build L1)</span></summary>

*Threat:* Upload a package without provenance.

*Mitigation:* Verifier requires provenance before accepting the package.

*Example:* Adversary uploads a malicious version of MyPackage to the package
repository without provenance. Solution: Verifier rejects because provenance is
missing.

</details>
<details><summary>Tamper with artifact after CI/CD <span>(Build L1)</span></summary>

*Threat:* Take a benign version of the package, modify it in some way, then
re-upload it using the original provenance.

*Mitigation:* Verifier checks that the provenance's `subject` matches the hash
of the package.

*Example:* Adversary performs a proper build, modifies the artifact, then
uploads the modified version of the package to the repository along with the
provenance. Solution: Verifier rejects because the hash of the artifact does not
match the `subject` found within the provenance.

</details>
<details><summary>Tamper with provenance <span>(Build L2)</span></summary>

*Threat:* Perform a build that would not meet expectations, then modify the
provenance to make the expectations checks pass.

*Mitigation:* Verifier only accepts provenance with a valid [cryptographic
signature][authentic] or equivalent proving that the provenance came from an
acceptable builder.

*Example:* MyPackage is expected to be built by GitHub Actions from the
`good/my-package` repo. Adversary builds with GitHub Actions from the
`evil/my-package` repo and then modifies the provenance so that the source looks
like it came from `good/my-package`. Solution: Verifier rejects because the
cryptographic signature is no longer valid.

</details>

### (G) Distribution channel

An adversary modifies the package on the package registry using an
administrative interface or through a compromise of the infrastructure.

**TODO:**

## Usage threats

A usage threat is a potential for an adversary to exploit behavior of the
consumer.

### (H) Package selection

The consumer requests a package that it did not intend.

<details><summary>Dependency confusion</summary>

*Threat:* Register a package name in a public registry that shadows a name used
on the victim's internal registry, and wait for a misconfigured victim to fetch
from the public registry instead of the internal one.

**TODO:** fill out the rest of this section

</details>
<details><summary>Typosquatting</summary>

*Threat:* Register a package name that is similar looking to a popular package
and get users to use your malicious package instead of the benign one.

*Mitigation:* **Mostly outside the scope of SLSA.** That said, the requirement
to make the source available can be a mild deterrent, can aid investigation or
ad-hoc analysis, and can complement source-based typosquatting solutions.

</details>

### (I) Usage

**TODO:** What should we put here?

## Availability threats

**TODO:** Merge into the list above rather than having a separate section.

An availability threat is a potential for an adversary to deny someone from
reading a source and its associated change history, or from building a package.

SLSA v1.0 does not address availability threats, though future versions might.

<details><summary>(A)(B) Delete the code</summary>

*Threat:* Perform a build from a particular source revision and then delete that
revision or cause it to get garbage collected, preventing anyone from inspecting
the code.

*Mitigation:* Some system retains the revision and its version control history,
making it available for inspection indefinitely. Users cannot delete the
revision except as part of a transparent legal or privacy process.

*Example:* An adversary submits malicious code to the MyPackage GitHub repo,
builds from that revision, then does a force push to erase that revision from
history (or requests that GitHub delete the repo.) This would make the revision
unavailable for inspection. Solution: Verifier rejects the package because it
lacks a positive attestation showing that some system, such as GitHub, ensured
retention and availability of the source code.

</details>
<details><summary>A dependency becomes temporarily or permanently unavailable to the build process</summary>

*Threat:* Unable to perform a build with the intended dependencies.

*Mitigation:* **Outside the scope of SLSA.** That said, some solutions to
support hermetic and reproducible builds may also reduce the impact of this
threat.

</details>
<details><summary>De-list artifact</summary>

*Threat:* The package registry stops serving the artifact.

*Mitigation:* N/A - This threat is out of scope of SLSA v1.0.

</details>
<details><summary>De-list provenance</summary>

*Threat:* The package registry stops serving the provenance.

*Mitigation:* N/A - This threat is out of scope of SLSA v1.0.

</details>

## Verification threats

Threats that can compromise the ability to prevent or detect the supply chain
security threats above.

<details><summary>Tamper with recorded expectations</summary>

*Threat:* Modify the verifier's recorded expectations, causing the verifier to
accept an unofficial package artifact.

*Mitigation:* Changes to recorded expectations requires some form of
authorization, such as two-party review.

*Example:* The package ecosystem records its expectations for a given package
name in a configuration file that is modifiable by that package's producer. The
configuration for MyPackage expects the source repository to be
`good/my-package`. The adversary modifies the configuration to also accept
`evil/my-package`, and then builds from that repository and uploads a malicious
version of the package. Solution: Changes to the recorded expectations require
two-party review.

</details>
<details><summary>Forge change metadata</summary>

*Threat:* Forge the change metadata to alter attribution, timestamp, or
discoverability of a change.

*Mitigation:* Source control platform strongly authenticates actor identity,
timestamp, and parent revisions.

*Example:* Adversary submits a git commit with a falsified author and timestamp,
and then rewrites history with a non-fast-forward update to make it appear to
have been made long ago. Solution: Consumer detects this by seeing that such
changes are not strongly authenticated and thus not trustworthy.

</details>
<details><summary>Exploit cryptographic hash collisions</summary>

*Threat:* Exploit a cryptographic hash collision weakness to bypass one of the
other controls.

*Mitigation:* Require cryptographically secure hash functions for commit
checksums and provenance subjects, such as SHA-256.

*Examples:* Construct a benign file and a malicious file with the same SHA-1
hash. Get the benign file reviewed and then submit the malicious file.
Alternatively, get the benign file reviewed and submitted and then build from
the malicious file. Solution: Only accept cryptographic hashes with strong
collision resistance.

</details>

</article>

<!-- Links -->

[authentic]: requirements.md#provenance-authentic
[exists]: requirements.md#provenance-exists
[isolated]: requirements.md#isolated
[unforgeable]: requirements.md#provenance-unforgeable
[supply chain threats]: threats-overview
