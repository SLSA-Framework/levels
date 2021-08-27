# Requirements

<span class="subtitle">

This document covers all of the detailed requirements for an artifact to meet SLSA. For a broader overview, including basic terminology and threat model, see [overview](index.md).

</span>

_Reminder: SLSA is in [alpha](roadmap.md). The definitions below are not yet finalized and subject to change, particularly SLSA 3-4._

-   [What is SLSA?](#what-is-slsa)
-   [Definitions](#definitions)
-   [Source requirements](#source-requirements)
-   [Build requirements](#build-requirements)
-   [Provenance requirements](#provenance-requirements)
-   [Common requirements](#common-requirements)

## What is SLSA?

SLSA is a set of incrementally adoptable security guidelines, established by industry consensus. The standards set by SLSA are guiding principles for both software producers and consumers: producers can follow the guidelines to make their software more secure, and consumers can make decisions based on a software package's security posture. SLSA's [four levels](levels.md) are designed to be incremental and actionable, and to protect against specific integrity attacks. SLSA 4 represents the ideal end state, and the lower levels represent milestones with corresponding integrity guarantees.

### Terminology

SLSA's framework addresses every step of the software supply chain - the sequence of steps resulting in the creation of an artifact. We represent a supply chain as a [directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) of sources, builds, dependencies, and packages. One artifact's supply chain is a combination of its dependencies' supply chains plus its own sources and builds.

![Software Supply Chain Model](images/supply-chain-model.svg)

| Term       | Description                                                                                                                                                                 | Example                                                                                                                                                                                     |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Artifact   | An immutable blob of data; primarily refers to software, but SLSA can be used for any artifact.                                                                             | A file, a git commit, a directory of files (serialized in some way), a container image, a firmware image.                                                                                   |
| Source     | Artifact that was directly authored or reviewed by persons, without modification. It is the beginning of the supply chain; we do not trace the provenance back any further. | Git commit (source) hosted on GitHub (platform).                                                                                                                                            |
| Build      | Process that transforms a set of input artifacts into a set of output artifacts. The inputs may be sources, dependencies, or ephemeral build outputs.                       | .travis.yml (process) run by Travis CI (platform).                                                                                                                                          |
| Package    | Artifact that is "published" for use by others. In the model, it is always the output of a build process, though that build process can be a no-op.                         | Docker image (package) distributed on DockerHub (platform). A ZIP file containing source code is a package, not a source, because it is built from some other source, such as a git commit. |
| Dependency | Artifact that is an input to a build process but that is not a source. In the model, it is always a package.                                                                | Alpine package (package) distributed on Alpine Linux (platform).                                                                                                                            |

## Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

<a id="immutable-reference"></a>**Immutable reference:** An identifier that is
guaranteed to always point to the same, immutable artifact. This MUST allow the
consumer to locate the artifact and SHOULD include a cryptographic hash of the
artifact's contents to ensure integrity. Examples: git URL + branch/tag/ref \+
commit ID; cloud storage bucket ID + SHA-256 hash; Subversion URL (no hash).

[immutable reference]: #immutable-reference
[immutable references]: #immutable-reference

**Platform:** Infrastructure or service that hosts the source, build, or
distribution of software. Examples: GitHub, Google Cloud Build, Travis CI,
[Mozilla's self-hosted Mercurial server](https://hg.mozilla.org).

**Provenance**: Metadata about how an artifact was produced.

**Revision:** An immutable, coherent state of a source. In Git, for example, a
revision is a commit in the history reachable from a specific branch in a
specific repository. Different revisions within one repo MAY have different
levels. Example: the most recent revision on a branch meets SLSA 4 but very old
historical revisions before the cutoff do not.

**Trusted persons:** Set of persons who are granted the authority to maintain a
software project. For example, https://github.com/MarkLodato/dotfiles has just
one trusted person (MarkLodato), while https://hg.mozilla.org/mozilla-central
has a set of trusted persons with write access to the mozilla-central
repository.

## Source requirements

<table>
<tr><th>Requirement<th>Description<th>L1<th>L2<th>L3<th>L4
<tr id="version-controlled">
<td>Version controlled
<td>

Every change to the source is tracked in a version control system that meets the
following requirements:

-   **[Change history]** There exists a record of the history of changes
    that went into the revision. Each change must contain: the identities of
    the uploader and reviewers (if any), timestamps of the reviews (if any)
    and submission, the change description/justification, the content of
    the change, and the parent revisions.

-   **\[Immutable reference]** There exists a way to indefinitely reference
    this particular, immutable revision. In git, this is the {repo URL +
    branch/tag/ref + commit ID}.

Most popular version control system meet this requirement, such as git,
Mercurial, Subversion, or Perforce.

NOTE: This does NOT require that the code, uploader/reviewer identities, or
change history be made public. Rather, some organization must attest to the fact
that these requirements are met, and it is up to the consumer whether this
attestation is sufficient.

<td> <td>✓<td>✓<td>✓
<tr id="verified-history">
<td>Verified history
<td>

Every change in the revision's history has at least one strongly authenticated
actor identity (author, uploader, reviewer, etc.) and timestamp. It must be
clear which identities were verified, and those identities must use [two-step
verification](https://www.google.com/landing/2step/) or similar. (Exceptions
noted below.)

-   **[First-parent history]** In the case of a non-linear version control
    system, where a revision can have more than one parent, only the "first
    parent history" is in scope. In other words, when a feature branch is merged
    back into the main branch, only the merge itself is in scope.
-   **[Historical cutoff]** There is some TBD exception to allow existing
    projects to meet SLSA 3/4 even if historical revisions were present in the
    history. Current thinking is that this could be either last N months or a
    platform attestation guaranteeing that future changes in the next N months
    will meet the requirements.

<td> <td> <td>✓<td>✓
<tr id="retained-indefinitely">
<td>Retained indefinitely
<td>

The revision and its change history are preserved indefinitely and cannot be
deleted, except when subject to an established and transparent policy for
obliteration, such as a legal or policy requirement.

-   **[Immutable history]** It must not be possible for persons to delete or
    modify the history, even with multi-party approval, except by trusted
    platform admins with two-party approval following the obliterate policy.
-   **[Limited retention for SLSA 2]** At SLSA 2 (but not 3), it is acceptable
    for the retention to be limited to 18 months, as attested by the source
    control platform.
    -   Example: If a commit is made on 2020-04-05 and then a retention
        attestation is generated on 2021-01-01, the commit must be retained
        until at least 2022-07-01.

<td> <td> <td>18 mo.<td>✓
<tr id="two-person-reviewed">
<td>Two-person reviewed
<td>

Every change in the revision's history was agreed to by two trusted persons
prior to submission, and both of these trusted persons were strongly
authenticated. (Exceptions from [Verified History](#verified-history) apply here
as well.)

-   The following combinations are acceptable:
    -   Uploader and reviewer are two different trusted persons.
    -   Two different reviewers are trusted persons.
-   **[Different persons]** The platform ensures that no person can use
    alternate identities to bypass the two-person review requirement.
    -   Example: if a person uploads with identity X then reviews with alias Y,
        the platform understands that this is the same person and does not
        consider the review requirement satisfied.
-   **[Informed review]** The reviewer is able and encouraged to make an
    informed decision about what they're approving. The reviewer should be
    presented with a full, meaningful content diff between the proposed revision
    and the previously reviewed revision. For example, it is not sufficient to
    just indicate that file changed without showing the contents.
-   **[Context-specific approvals]** Approvals are for a specific context, such
    as a repo + branch in git. Moving fully reviewed content from one context to
    another still requires review. (Exact definition of "context" depends on the
    project, and this does not preclude well-understood automatic or reviewless
    merges, such as cutting a release branch.)
    -   Git example: If a fully reviewed commit in one repo is merged into a
        different repo, or a commit in one branch is merged into a different
        branch, then the merge still requires review.

<td> <td> <td> <td>✓
</table>

## Build requirements

Requirements on build process:

<table>
<tr><th>Requirement<th>Description<th>L1<th>L2<th>L3<th>L4
<tr id="scripted-build">
<td>Scripted build
<td>

All build steps were fully defined in some sort of "build script". The
only manual command, if any, was to invoke the build script.

Examples:

-   Build script is Makefile, invoked via `make all`.
-   Build script is .github/workflows/build.yaml, invoked by GitHub Actions.

<td>✓<td>✓<td>✓<td>✓
<tr id="build-service">
<td>Build service
<td>

All build steps ran using some build service, not on a developer's
workstation.

Examples: GitHub Actions, Google Cloud Build, Travis CI.

<td> <td>✓<td>✓<td>✓
<tr id="config-as-code">
<td>Config as code
<td>

The build service configuration is defined in source control and drives the build automatically.

Examples: cloudbuild.yaml, ./github/workflows/build.yaml, zuul.yaml.
<td> <td> <td>✓<td>✓
<tr id="ephemeral-environment">
<td>Ephemeral environment
<td>

The build service ensured that the build steps ran in an ephemeral environment,
such as a container or VM, provisioned solely for this build, and not reused
from a prior build.

<td> <td> <td>✓<td>✓
<tr id="isolated">
<td>Isolated
<td>

The build service ensured that the build steps ran in an isolated environment
free of influence from other build instances, whether prior or concurrent.

-   It MUST NOT be possible for a build to access any secrets of the build
    service, such as the provenance signing key.
-   It MUST NOT be possible for two builds that overlap in time to
    influence one another.
-   It MUST NOT be possible for one build to persist or influence the
    build environment of a subsequent build.
-   Build caches, if used, MUST be purely content-addressable to prevent
    tampering.

<td> <td> <td>✓<td>✓
<tr id="parameterless">
<td>Parameterless
<td>

The build output cannot be affected by user parameters other than the build
entry point and the top-level source location. In other words, the build is
fully defined through the build script and nothing else.

Examples:

-   GitHub Actions
    [workflow_dispatch](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#workflow_dispatch)
    `inputs` MUST be empty.
-   Google Cloud Build
    [user-defined substitutions](https://cloud.google.com/build/docs/configuring-builds/substitute-variable-values)
    MUST be empty. (Default substitutions, whose values are defined by the
    server, are acceptable.)

<td> <td> <td> <td>✓
<tr id="hermetic">
<td>Hermetic
<td>

All transitive build steps, sources, and dependencies were fully declared up
front with [immutable references], and the build steps ran with no network
access.

The user-defined build script:

-   MUST declare all dependencies, including sources and other build steps,
    using [immutable references] in a format that the build service understands.

The build service:

-   MUST fetch all artifacts in a trusted control plane.
-   MUST NOT allow mutable references.
-   MUST verify the integrity of each artifact.
    -   If the [immutable reference] includes a cryptographic hash, the service
        MUST verify the hash and reject the fetch if the verification fails.
    -   Otherwise, the service MUST fetch the artifact over a channel that
        ensures transport integrity, such as TLS or code signing.
-   MUST prevent network access while running the build steps.
    -   This requirement is "best effort." It SHOULD deter a reasonable team
        from having a non-hermetic build, but it need not stop a determined
        adversary. For example, using a container to prevent network access is
        sufficient.

<td> <td> <td> <td>✓
<tr id="reproducible">
<td>Reproducible
<td>

Re-running the build steps with identical input artifacts results in bit-for-bit
identical output. Builds that cannot meet this MUST provide a justification why
the build cannot be made reproducible.

"○" means that this requirement is "best effort". The user-provided build script
SHOULD declare whether the build is intended to be reproducible or a
justification why not. The build service MAY blindly propagate this intent
without verifying reproducibility. A consumer MAY reject the build if it does
not reproduce.

<td> <td> <td> <td>○
</table>

## Provenance requirements

Requirements on the process by which provenance is generated and consumed:

<table>
<tr><th>Requirement<th>Description<th>L1<th>L2<th>L3<th>L4
<tr id="available">
<td>Available
<td>

The provenance is available to the consumer in a format that the consumer
accepts. The format SHOULD be in-toto [SLSA Provenance](provenance/index.html),
but another format MAY be used if both producer and consumer agree and it meets
all the other requirements.

<td>✓<td>✓<td>✓<td>✓
<tr id="authenticated">
<td>Authenticated
<td>

The provenance's authenticity and integrity can be verified by the consumer.
This SHOULD be through a digital signature from a private key accessible only to
the service generating the provenance.

<td> <td>✓<td>✓<td>✓
<tr id="service-generated">
<td>Service generated
<td>

The data in the provenance MUST be obtained from the build service (either because
the generator _is_ the build service or because the provenance generator reads the
data directly from the build service).

Regular users of the service MUST NOT be
able to inject or alter the contents, except as noted below.

The following provenance fields MAY be generated by the user-controlled build
steps:

-   The output artifact hash from [Identifies Artifact](#identifies-artifact).
    -   Reasoning: This only allows a "bad" build to falsely claim that it
        produced a "good" artifact. This is not a security problem because the
        consumer MUST accept only "good" builds and reject "bad" builds.
-   The "reproducible" boolean and justification from
    [Reproducible](#reproducible).

<td> <td>✓<td>✓<td>✓
<tr id="non-falsifiable">
<td>Non-falsifiable
<td>

Provenance cannot be falsified by the build service's users.

NOTE: This requirement is a stricter version of [Service Generated](#service-generated).

-   The provenance signing key MUST be stored in a secure key management system
    accessible only to the build service account.
-   The provenance signing key MUST NOT be accessible to the environment running
    the user-defined build steps.
-   Every field in the provenance MUST be generated or verified by the build
    service in a trusted control plane. The user-controlled build steps MUST
    NOT be able to inject or alter the contents, except as noted below.

The following provenance fields MAY be generated by the user-controlled build
steps without the build service verifying their correctness:

-   The output artifact hash from [Identifies Artifact](#identifies-artifact).
    -   Reasoning: This only allows a "bad" build to falsely claim that it
        produced a "good" artifact. This is not a security problem because the
        consumer MUST accept only "good" builds and reject "bad" builds.
-   The "reproducible" boolean and justification from
    [Reproducible](#reproducible).

<td> <td> <td>✓<td>✓
<tr id="dependencies-complete">
<td>Dependencies complete
<td>

Provenance records all build dependencies that were available while running the
build steps. This includes the initial state of the machine, VM, or container of
the build worker.

-   MUST include all user-specified build steps, sources, dependencies.
-   SHOULD include all service-provided artifacts.

<td> <td> <td> <td>✓
</table>

Requirements on the contents of the provenance:

<table>
<tr><th>Requirement<th>Description<th>L1<th>L2<th>L3<th>L4
<tr id="identifies-artifact">
<td>Identifies artifact
<td>

The provenance MUST identify the output artifact via at least one
cryptographic hash. The provenance MAY provide multiple identifying
cryptographic hashes using different algorithms. When only one hash is
provided, the RECOMMENDED algorithm is SHA-256 for cross-system
compatibility. If another algorithm is used, it SHOULD be resistant to
collisions and second preimages.

<td>✓<td>✓<td>✓<td>✓
<tr id="identifies-builder">
<td>Identifies builder
<td>

The provenance identifies the entity that performed the build and generated the
provenance. This represents the entity that the consumer must trust. Examples:
"GitHub Actions with a GitHub-hosted worker", "jdoe@example.com's machine".

<td>✓<td>✓<td>✓<td>✓
<tr id="identifies-build-instructions">
<td>Identifies build instructions
<td>

Identifies the top-level instructions used to execute the build.

The identified instructions SHOULD be at the highest level available to the build. 
If the build uses config-as-code, this SHOULD be the source repo and entry point
of the build config and NOT the build steps within the build config.  If the
build does not use config-as-code it MAY list details of the build steps.

Config-as-code example:

```json5
"recipe": {
  "type": "https://github.com/Attestations/GitHubActionsWorkflow@v1",
  // Identifies the source repo of the build config.
  "definedInMaterial": 0,
  // Identifies the entrypoint within the source repo at the path
  // .github/workflows/build.yaml, using the job "build".
  "entryPoint": ".github/workflows/build.yaml:build",
},
…
"materials": [{
   "uri": "git+https://the.repo/foo",
   "digest": {"sha1": "abc123..."}
}]
```

Build steps example:

```json5
"recipe": {
  // Build steps were provided as an argument. No `definedInMaterial` or
  // `entryPoint`.
  "type": "https://tekton.dev/chains/recipe/buildSteps@v1",
  "arguments": {
      "steps": [
          {
            "entryPoint": "",
            "arguments": null,
            "environment": {
              "container": "git-source-repo-jwqcl",
              "image": "gcr.io/tekton-releases/..."
            },
            "annotations": null
          },
          {
            "entryPoint": "#!/usr/bin/env bash\nset -x\n\ncd ...",
            "arguments": null,
            "environment": {
              "container": "build",
              "image": "gcr.io/cloud-marketplace-containers/..."
            },
            "annotations": null
          }
    ]
}
```
<td>✓<td>✓<td>✓<td>✓
<tr id="identifies-entry-point">
<td>Identifies entry point
<td>

The provenance identifies the "entry point" of the build service configuration
used to drive the build including what source repo the configuration was read
from.

Example:
- source repo: git URL + branch/tag/ref + commit ID
- entrypoint: path to config file(s) (e.g. ./.zuul.yaml) + job name within config
    (e.g. envoy-build-arm64)

<td><td><td>✓<td>✓
<tr id="includes-all-params">
<td>Includes all build parameters
<td>

The provenance includes all build parameters under a user's control. See
[Parameterless](#parameterless) for details. (At L3, the parameters must be
listed; at L4, they must be empty.)

<td> <td> <td>✓<td>✓
<tr id="includes-all-deps">
<td>Includes all transitive dependencies
<td>

The provenance includes all transitive dependencies listed in
[Dependencies Complete](#dependencies-complete).

<td> <td> <td> <td>✓
<tr id="includes-reproducible-info">
<td>Includes reproducible info
<td>

The provenance includes a boolean indicating whether build is intended to be
reproducible and, if so, all information necessary to reproduce the build. See
[Reproducible](#reproducible) for more details.

<td> <td> <td> <td>✓
<tr id="includes-metadata">
<td>Includes metadata
<td>

The provenance includes metadata to aid debugging and investigations. This
SHOULD at least include start and end timestamps and a permalink to debug logs.

"○" = RECOMMENDED.

<td>○<td>○<td>○<td>○
</table>

## Common requirements

Common requirements for every trusted system involved in the supply chain
(source, build, distribution, etc.)

<table>
<tr><th>Requirement<th>Description<th>L1<th>L2<th>L3<th>L4
<tr id="security">
<td>Security
<td>

The system meets some TBD baseline security standard to prevent compromise.
(Patching, vulnerability scanning, user isolation, transport security, secure
boot, machine identity, etc. Perhaps
[NIST 800-53](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r5.pdf)
or a subset thereof.)

<td> <td> <td> <td>✓
<tr id="access">
<td>Access
<td>

All physical and remote access must be rare, logged, and gated behind
multi-party approval.

<td> <td> <td> <td>✓
<tr id="superusers">
<td>Superusers
<td>

Only a small number of platform admins may override the guarantees listed here.
Doing so MUST require approval of a second platform admin.

<td> <td> <td> <td>✓
</table>
