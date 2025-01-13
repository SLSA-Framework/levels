# SLSA Source Track

## Outstanding TODOs

Open issues are tracked with the [source-track](https://github.com/slsa-framework/slsa/issues?q=is%3Aissue+is%3Aopen+label%3Asource-track) label in the [slsa-framework/slsa](https://github.com/slsa-framework/slsa) repository.
Source Track issues are triaged on the [SLSA Source Track](https://github.com/orgs/slsa-framework/projects/5) project board.

## Objective

The SLSA source track describes increasing levels of trustworthiness and completeness in a repository revision's provenance (e.g. how it was generated, who the contributors were, etc...).

The Source track is scoped to revisions of a single repository that is controlled by an organization.
That organization determines the intent of the software in the repository, what Source level should apply to the repository and administers technical controls to enforce that level.

The primary purpose of the Source track is to enable verification that the creation of a revision followed the expected process.
Consumers can examine the various source provenance attestations to determine if all sources used during the build meet their requirements.

## Definitions

| Term | Description
| --- | ---
| Source | An identifiable set of text and binary files and associated metadata. Source is regularly used as input to a build system (see [SLSA Build Track](requirements.md)).
| Organization | A collection of people who collectively create the Source. Examples of organizations include open-source projects, a company, or a team within a company. The organization defines the goals and methods of the source.
| Version Control System (VCS)| Software for tracking and managing changes to source. Git and Subversion are examples of version control systems.
| Revision | A specific state of the source with an identifier provided by the version control system. As an example, you can identify a git revision by its tree hash.
| Source Control System (SCS) | A suite of tools and services (self-hosted or SaaS) relied upon by the organization to produce new revisions of the source. The role of the SCS may be fulfilled by a single service (e.g., GitHub / GitLab) or rely on a combination of services (e.g., GitLab with Gerrit code reviews, GitHub with OpenSSF Scorecard, etc).
| Source Provenance | Information about how a revision came to exist, where it was hosted, when it was generated, what process was used, who the contributors were, and what parent revisions it was based on.
| Repository / Repo | A uniquely identifiable instance of a VCS. The repository controls access to the Source in the VCS. The objective of a repository is to reflect the intent of the organization that controls it.
| Branch | A named pointer to a revision. Branches may be modified by authorized actors. Branches may have different security requirements.
| Change | A set of modifications to the source in a specific context. A change can be proposed and reviewed before being accepted.
| Change History | A record of the history of revisions that preceded a specific revision.
| Push / upload / publish | When an actor authenticates to a Repository to add or modify content. Typically makes a new revision reachable from a branch.
| Review / approve / vote | When an actor authenticates to a change review tool to comment upon, endorse, or reject a source change proposal.

## Source Roles

| Role | Description
| --- | ---
| Administrator | A human who can perform privileged operations on one or more projects. Privileged actions include, but are not limited to, modifying the change history and modifying project- or organization-wide security policies.
| Trusted person | A human who is authorized by the organization to propose and approve changes to the source.
| Trusted robot | Automation with an authentic identity that is authorized by the organization to propose and/or approve changes to the source.
| Untrusted person | A human who has limited access to the project. They MAY be able to read the source. They MAY be able to propose or review changes to the source. They MAY NOT approve changes to the source or perform any privileged actions on the project.
| Proposer | An actor that proposes (or uploads) a particular change to the source.
| Reviewer / Voter / Approver | An actor that reviews (or votes on) a particular change to the source.
| Merger | An actor that applies a change to the source. This actor may be the proposer.

## Safe Expunging Process

SCSs MAY allow the organization to expunge (remove) content from a repository and its change history without leaving a public record of the removed content,
but the organization MUST only allow these changes in order to meet legal or privacy compliance requirements.
Content changed under this process includes changing files, history, references, or any other metadata stored by the SCS.

### Warning

Removing a revision from a repository is similar to deleting a package version from a registry: it's almost impossible to estimate the amount of downstream supply chain impact.
For example, in VCSs like Git, removal of a revision changes the object IDs of all subsequent revisions that were built on top of it, breaking downstream consumers' ability to refer to source they've already integrated into their products.

It may be the case that the specific set of changes targeted by a legal takedown can be expunged in ways that do not impact consumed revisions, which can mitigate these problems.

It is also the case that removing content from a repository won't necessarily remove it everywhere.
The content may still exist in other copies of the repository, either in backups or on developer machines.

### Process

An organization MUST document the Safe Expunging Process and describe how requests and actions are tracked and SHOULD log the fact that content was removed.
Different organizations and tech stacks may have different approaches to the problem.

SCSs SHOULD have technical mechanisms in place which require an Administrator plus, at least, one additional 'trusted person' to trigger any expunging (removals) made under this process.

The application of the safe expunging process and the resulting logs MAY be private to both prevent calling attention to potentially sensitive data (e.g. PII) or to comply with local laws
and regulations which may require the change to be kept private to the extent possible.
Organizations SHOULD prefer to make logs public if possible.

## Levels

### Level 1: Version controlled

Summary:
The source is stored and managed through a modern version control system.

Intended for: Organizations currently storing source in non-standard ways who want to quickly gain some benefits of SLSA and better integrate with the SLSA ecosystem with minimal impact to their current workflows.

Benefits:
Migrating to the appropriate tools is an important first step on the road to operational maturity.

### Level 2: Branch History

Summary:
Clarifies which branches in a repo are consumable and guarantees that all changes to protected branches are recorded.

Intended for:
All organizations of any size producing software of any kind.

Benefits:
Allows source consumers to track changes to the software over time and attribute those changes to the people that made them.

### Level 3: Authenticatable and Auditable Provenance

Summary:
The SCS generates credible, tamper-resistant, and contemporaneous evidence of how a specific revision was created.
It is provided to authorized users of the source repository in a documented format.
of how a specific revision was created to authorized users of the source repository.

Intended for:
Organizations that want strong guarantees and auditability of their change management processes.

Benefits:
Provides authenticatable and auditable information to policy enforcement tools and reduces the risk of tampering
within the SCS's storage systems.

## Requirements

Many examples in this document use the [git version control system](https://git-scm.com/), but use of git is not a requirement to meet any level on the SLSA source track.

### Organization Requirements

<table>
<tr><th>Requirement<th>Description<th>L1<th>L2<th>L3

<tr id="use-modern-tools"><td>Use modern tools<td>

The organization MUST manage the source using tools specifically designed to manage source code.
Tools like git, Perforce, Subversion are great examples.
They may be self-hosted or hosted in the cloud using vendors like GitLab, GitHub, Bitbucket, etc.

When self-hosting a solution, local, unauthenticated storage is not acceptable.

Branch protection is not required, nor are there any other constraints on the configuration of the tools.

<td>✓<td>✓<td>✓
<tr id="canonical-location"><td>Canonical location<td>

The source MUST have a location where the "official" revisions are stored and managed.

<td>✓<td>✓<td>✓
<tr id="distribute-summary-attestations"><td>Distribute summary attestations<td>

The organization MUST document how [summary attestations](#summary-attestation) are distributed
for relevant source repositories.
<td>✓<td>✓<td>✓
<tr id="distribute-provenance-attestations"><td>Distribute provenance attestations<td>

The organization MUST document how the detailed [provenance attestations](#provenance-attestations)
are distributed for relevant source repositories.
<td><td><td>✓

</table>

### Source Control System Requirements

<table>
<tr><th>Requirement<th>Description<th>L1<th>L2<th>L3
<tr id="immutable-revisions"><td>Revisions are immutable and uniquely identifiable<td>

This requirement ensures that a consumer can determine that the source revision they have is the same as a canonical revision.
The SCS MUST provide a deterministic way to identify a particular revision.

Virtually all modern tools provide this guarantee via a combination of the repository ID and revision ID.

<td>✓<td>✓<td>✓
<tr id="repository-ids"><td>Repository IDs<td>

The repository ID is defined by the SCS and MUST be unique in the context of that instance of the SCS.

<td>✓<td>✓<td>✓
<tr id="revision-ids"><td>Revision IDs<td>

When the revision ID is a digest of the content of the revision (as in git) nothing more is needed.
When the revision ID is a number or otherwise not a digest, then the SCS MUST document how the immutability of the revision is established.
The same revision ID MAY be present in multiple repositories.
See also [Use cases for non-cryptographic, immutable, digests](https://github.com/in-toto/attestation/blob/main/spec/v1/digest_set.md#use-cases-for-non-cryptographic-immutable-digests).

<td>✓<td>✓<td>✓
<tr id="source-summary"><td>Source Summary Attestations<td>

The SCS MUST generate [summary attestations](#summary-attestation) to enable users to determine the source level of a given revision.

If a consumer is authorized to access source on a particular branch, they MUST be able to fetch the summary attestations for revisions in the history of that branch.

It is possible that an SCS can make no claims about a particular revision.
For example, this would happen if the revision was created on another SCS, or if the revision was not the result of an accepted change management process.

<td>✓<td>✓<td>✓
<tr id="branches"><td>Branches<td>

If the SCS supports multiple branches, the organization MUST indicate which branches are intended for consumption.
This may be implied or explicit.

For example, an organization may declare that the `default` branch of a repo contains revisions intended for consumption my protected it.

They may also declare that branches named with the prefix `refs/heads/releases/*` contain revisions held to an even higher standard.

They may also declare all revisions are intended to be consumed "except those reachable only from branches beginning with `refs/heads/users/*`."
This is a typical setup for teams who leverage code review tools.

<td><td>✓<td>✓
<tr id="continuity"><td>Continuity<td>

For all branches intended for consumption, whenever a branch is updated to point to a new revision, that revision MUST document how it related to the previous revision.
Exceptions are allowed via the [safe expunging process](#safe-expunging-process).

It MUST NOT be possible to rewrite the history of branches intended for
consumption. In other words, when updating the branch to point to a new
revision, that revision must be a direct descendant of the current revision. In
an SCS that hosts a Git repository on systems like GitHub or GitLab, this can be
accomplished by enabling branch protection rules that prevent force pushes and
branch deletions.

It MUST NOT be possible to delete the entire repository (including all branches) and replace it with different source.
Exceptions are allowed via the [safe expunging process](#safe-expunging-process).

<td><td>✓<td>✓
<tr id="identity-management"><td>Identity Management<td>

There exists an identity management system or some other means of identifying
and authenticating actors. Depending on the SCS, identity management may be
provided by source control services (e.g., GitHub, GitLab), implemented using
cryptographic signatures (e.g., using gittuf to manage public keys for actors),
or extend existing authentication systems used by the organization (e.g., Active
Directory, Okta, etc.).

The SCS MUST document how actors are identified for the purposes of attribution.

Activities conducted on the SCS SHOULD be attributed to authenticated identities.

<td><td>✓<td>✓
<tr id="strong-authentication"><td>Strong Authentication<td>

User accounts that can modify the source or the project's configuration must use multi-factor authentication or its equivalent.
This strongly authenticated identity MUST be used for the generation of source provenance attestations.
The SCS MUST declare which forms of identity it considers to be trustworthy for this purpose.
For cloud-based SCSs, this will typically be the identity used to push to a repository.

Other forms of identity MAY be included as informational.
Examples include a git commit's "author" and "committer" fields and a gpg signature's "user id."
These forms of identity are user-provided and not typically verified by the source provenance attestation issuer.

See [source roles](#source-roles).

<td><td><td>✓
<tr id="source-provenance"><td>Source Provenance<td>

[Source Provenance](#provenance-attestations) are attestations that contain information about how a specific revision was created and how it came to exist in its present context
(e.g. the branches or tags that point, or pointed, at that revision).
They are associated with the revision identifier delivered to consumers and are a statement of fact from the perspective of the SCS.

At Source Level 3 Source Provenance MUST be created contemporaneously with the revision being made available such that they provide a credible,
auditable, record of changes.

If a consumer is authorized to access source on a particular branch, they MUST be able to fetch the source attestation documents for revisions in the history of that branch.

It is possible that an SCS can make no claims about a particular revision.
For example, this would happen if the revision was created on another SCS, or if the revision was not the result of an accepted change management process.

<td><td><td>✓
<tr id="change-management-process"><td>Enforced change management process<td>

The SCS MUST ensure that all technical controls governing changes to a [branch](#definitions)

1.  Are discoverable by authorized users of the repo.
2.  Cannot be bypassed except via the [Safe Expunging Process](#safe-expunging-process).

For example, this could be accomplished by:

-   The configuration of branch protection rules (e.g.[GitHub](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets), [GitLab](https://docs.gitlab.com/ee/user/project/repository/branches/protected.html)), or
-   the application and verification of [gittuf](https://github.com/gittuf/gittuf) policies, or
-   some other mechanism as enforced by the [Change management tool](#change-management-tool-requirements).

<td><td><td>✓
</table>

### Change management tool requirements

The change management tool MUST be able to authoritatively state that each new revision reachable from the protected branch represents only the changes managed via the [process](#change-management-process).

<table>
<tr><th>Requirement<th>Description<th>L1<th>L2<th>L3

<tr id="context"><td>Context<td>

The change management tool MUST record the specific code change (a "diff" in git) or instructions to recreate it.
In git, this typically defined to be three revision IDs: the tip of the "topic" branch, the tip of the target branch, and closest shared ancestor between the two (such as determined by `git-merge-base`).

The change management tool MUST record the "target" context for the change proposal and the previous revision in that context.
For example, for the git version control system, the change management tool MUST record the branch name that was updated.

Branches may have differing security postures, and a change can be approved for one context while being unapproved for another.

<td><td><td>✓
<tr id="verified-timestamps"><td>Verified Timestamps<td>

The change management tool MUST record timestamps for all contributions and review-related activities.
User-provided timestamps MUST NOT be used.

<td><td><td>✓
</table>

## Communicating source levels

SLSA source level details are communicated using attestations.
These attestations either refer to a source revision itself or provide context needed to evaluate an attestation that _does_ refer to a revision.

There are two broad categories of source attestations within the source track:

1.  Summary attestations: Used to communicate to downstream users what high level security properties a given source revision meets.
2.  Provenance attestations: Provide trustworthy, tamper-proof, metadata with the necessary information to determine what high level security properties a given source revision has.

To provide interoperability and ensure ease of use, it's essential that the summary attestations are applicable across all Source Control Systems.
Due to the significant differences in how SCSs operate and how they may chose to meet the Source Track requirements it is preferable to
allow for flexibility with the full attestations.  To that end SLSA leaves provenance attestations undefined and up to the SCSs to determine
what works best in their environment.

### Summary attestation

Summary attestations are issued by some authority that has sufficient evidence to make the determination of a given
revision's source level.  Summary attestations convey properties about the revision as a whole and summarize properties computed over all
the changes that contributed to that revision over its history.

The source track issues summary attestations using [Verification Summary Attestations (VSAs)](./verification_summary.md) as follows:

1.  `subject.uri` SHOULD be set to a human readable URI of the revision.
2.  `subject.digest` MUST include the revision identifier (e.g. `gitCommit`) and MAY include other digests over the contents of the revision (e.g. `gitTree`, `dirHash`, etc...).
SCSs that do not use cryptographic digests MUST define a canonical type that is used to identify immutable revisions (e.g. `svn_revision_id`)[^1].
3.  `subject.annotations.source_branches` SHOULD be set to a list of branches that pointed to this revision at any point in their history.
    -   git branches MUST be fully qualified (e.g. `refs/head/main`) to reduce the likelihood of confusing downstream tooling.
4.  `resourceUri` MUST be set to the URI of the repository, preferably using [SPDX Download Location](https://spdx.github.io/spdx-spec/v2.3/package-information/#77-package-download-location-field).
E.g. `git+https://github.com/foo/hello-world`.
5.  `verifiedLevels` MUST include the SLSA source track level the verifier asserts the revision meets. One of `SLSA_SOURCE_LEVEL_0`, `SLSA_SOURCE_LEVEL_1`, `SLSA_SOURCE_LEVEL_2`, `SLSA_SOURCE_LEVEL_3`.
MAY include additional properties as asserted by the verifier.  The verifier MUST include _only_ the highest SLSA source level met by the revision.
6.  `dependencyLevels` MAY be empty as source revisions are typically terminal nodes in a supply chain.

Verifiers MAY issue these attestations based on their understanding of the underlying system (e.g. based on design docs, security reviews, etc...),
but at SLSA Source Level 3 MUST use tamper-proof [provenance attestations](#provenance-attestations) appropriate to their SCS when making the assessment.

The SLSA source track MAY create additional tags to include in `verifiedLevels` which attest
to other properties of a revision (e.g. if it was code reviewed).  All SLSA source tags will start with `SLSA_SOURCE_`.

#### Populating source_branches

The summary attestation issuer may choose to populate `source_branches` in any way they wish.
Downstream users are expected to be familiar with the method used by the issuer.

Example implementations:

-   Issue a new VSA for each merged Pull Request and add the destination branch to `source_branches`.
-   Issue a new VSA each time a 'consumable branch' is updated to point to a new revision.

#### Example

```json
"_type": "https://in-toto.io/Statement/v1",
"subject": [{
  "uri": "https://github.com/foo/hello-world/commit/9a04d1ee393b5be2773b1ce204f61fe0fd02366a",
  "digest": {"gitCommit": "9a04d1ee393b5be2773b1ce204f61fe0fd02366a"},
  "annotations": {"source_branches": ["refs/heads/main", "refs/heads/release_1.0"]}
}],

"predicateType": "https://slsa.dev/verification_summary/v1",
"predicate": {
  "verifier": {
    "id": "https://example.com/source_verifier",
  },
  "timeVerified": "1985-04-12T23:20:50.52Z",
  "resourceUri": "git+https://github.com/foo/hello-world",
  "policy": {
    "uri": "https://example.com/slsa_source.policy",
  },
  "verificationResult": "PASSED",
  "verifiedLevels": ["SLSA_SOURCE_LEVEL_3"],
}
```

#### How to verify

-   VSAs for source revisions MUST follow [the standard method of VSA verification](./verification_summary.md#how-to-verify).
-   Users SHOULD check that an allowed branch is listed in `subject.annotations.source_branches` to ensure the revision is from an appropriate context within the repository.
-   Users SHOULD check that the expected `SLSA_SOURCE_LEVEL_` is listed within `verifiedLevels`.
-   Users MUST ignore any unrecognized values in `verifiedLevels`.

### Provenance attestations

Source provenance attestations provide tamper-proof evidence (ideally signed [in-toto attestations](https://github.com/in-toto/attestation/blob/main/README.md))
that can be used to determine what SLSA Source Level or other high level properties a given revision meets.
This evidence can be used by an authority as the basis for issuing a [Summary Attestation](#summary-attestation).

SCSs may have different methods of operating that necessitate different forms of evidence.
E.g. GitHub-based workflows may need different evidence than Gerrit-based workflows, which would both likely be different from workflows that
operate over Subversion repositories.

These differences also mean that depending on the configuration the issuers of provenance attestations may vary from implementation to implementation, often because entities with the knowledge to issue them may vary.
The authority that issues [summary-attestations](#summary-attestation) MUST understand which entity should issue each provenance attestation type and ensure the full attestations come from the appropriate issuer.

'Source provenance attestations' is a generic term used to refer to any type of attestation that provides evidence the process used to create a revision.

Example source provenance attestations:

-   A TBD attestation which describes the revision's parents and the actors involved in creating this revision.
-   A "code review" attestation which describes the basics of any code review that took place.
-   An "authentication" attestation which describes how the actors involved in any revision were authenticated.
-   A [Vuln Scan attestation](https://github.com/in-toto/attestation/blob/main/spec/predicates/vuln.md)
    which describes the results of a vulnerability scan over the contents of the revision.
-   A [Test Results attestation](https://github.com/in-toto/attestation/blob/main/spec/predicates/test-result.md)
 which describes the results of any tests run on the revision.
-   An [SPDX attestation](https://github.com/in-toto/attestation/blob/main/spec/predicates/spdx.md)
 which provides a software bill of materials for the revision.
-   A [SCAI attestation](https://github.com/in-toto/attestation/blob/main/spec/predicates/scai.md) used to
 describe which source quality tools were run on the revision.

[^1]: in-toto attestations allow non-cryptographic digest types: https://github.com/in-toto/attestation/blob/main/spec/v1/digest_set.md#supported-algorithms.
