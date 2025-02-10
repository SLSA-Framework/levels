---
title: Verifying source
description: |
  SLSA uses attestations to indicate security claims associated with a repository revision, but attestations don't do anything unless somebody inspects them.
  SLSA calls that inspection verification, and this page describes how to verify properties of source revisions using their SLSA source provenance attestations.
  The intended audience is platform implementers, security engineers, and software consumers.
layout: specifications
---

SLSA uses attestations to indicate security claims associated with a repository revision, but attestations don't do anything unless somebody inspects them.
SLSA calls that inspection **verification**, and this page describes how to verify properties of source revisions using their SLSA source provenance attestations.

Source Control Systems (SCSs) may issue attestations of the process that was used to create specific revisions of a repository.

A Verification Summary Attestation (VSA) can make verification more efficient by recording the result of prior verifications.
VSA may be issued by a VSA provider to make a SLSA source level determination based on the content of those attestations.

## How to verify SLSA source level

TODO: this section needs work.

### SLSA Level 1

Because there is no required attestation document at this level, the verification strategy is dependent upon the technologies being used.

-   If you can prove that the revision comes from the expected canonical location for the repository, the source meets level 1.
-   If you cannot, it does not meet level 1.

### SLSA Level 2

Because there is no required attestation document at this level, the verification strategy is dependent upon the technologies being used.

-   If the repository also has basic history protections, it meets level 2.

### SLSA Level 3+

Repos at this level report their claims in a signed attestation.

1.  For each repository you have, identify the repository id (such as the URL) and the revision id (such as the git commit SHA)
2.  Acquire all attestations for that repository id and revision id combination from preconfigured trusted attestation servers.
3.  Ensure that the source provenance attestations match the revision id.
4.  Ensure that the source provenance attestation documents were signed by a preconfigured root of trust.
5.  Ensure that the claims in the revision's provenance attestation meet your expectations for that repository.

## Common Source Controls and their Applications

Source attestations provide a trustworthy way to communicate security claims, but what should you be looking for in those claims?
Here are a few extremely common examples and important implementation details.

In addition to the requirements for SLSA Source L3, most organizations will require multiple of these controls as part of their required branch protections.

If an organization has indicated that use of these these controls are part of their repository's expectations, consumers SHOULD verify that the process was followed for the revision they are consuming.

### Changes are pre-authorized by two different authorized actors

Summary: All changes to the source are approved by two trusted actors prior to acceptance.

Intended for: Enterprise repositories and mature open source projects.

Benefits: A compromise of a single account does not result in compromise of the source.

Requirements:

#### Two authorized actors

All changes to the source are approved by two authorized actors prior to acceptance.
If the proposer is also an authorized actor, the proposer MAY approve their own proposal and count as one of the two required actors.

#### Different actors

It MUST NOT be possible for a single actor to control more than one voting accounts.

Should the organization discover that it issued multiple accounts to the same actors, it MUST act to rectify the situation.
For example, it might revoke project privileges for all but one of the accounts and perform retroactive code reviews on any changes where that actors' accounts are the author and/or code reviewer(s).

#### Post-approval changes allowed

When performing a review a reviewer may both approve the change and request modifications. Any modifications made need not reset the approval status of the change.  To protect against post-approval change see [Review Every Single Revision](#review-every-single-revision).

### Expert Code Review

Summary: All changes to the source are pre-approved by experts in those areas.

Intended for: Enterprise repositories and mature open source projects.

Benefits: Prevents mistakes made by developers who are unfamiliar with the area.

Requirements:

#### Code ownership

Each part of the source MUST have a clearly identified set of experts.

#### Approvals from all relevant experts

For each portion of the source modified by a change proposal, pre-approval MUST be granted by a member of the defined expert set.
A approval from an actor that is a member of multiple expert groups may satisfy the requirement for all groups in which they are a member.

### Review Every Single Revision

Summary: The final revision was reviewed by all relevant experts prior to submission.

Intended for: The highest-of-high-security-posture repos.

Benefits: Provides the maximum chance for experts to spot and reject problems before they ship.

Requirements:

#### Reset votes on all changes

If the proposal is modified after receiving expert approval, all previously granted approvals MUST be revoked.
A new approval MUST be granted from ALL required reviewers.

The new approval MAY be granted by an actor who approved a previous iteration.

### Automated testing

Summary:
The final revision was validated against a suite of automated tests.

Intended for:
All organizations and repositories.

Benefits:
Automatic testing has many benefits, including improved accuracy, error prevention and reduced workload on human developers.

Requirements:
The organization MUST configure a branch protection rule to require that only revisions with passing test results can be pointed-to by the branch.

Automatic tests SHOULD be executed in a trustworthy environment (see SLSA build track).

Results of each test (or an aggregate) MUST be collected by the change review tool and made available for verification.

Tests SHOULD be run against a revision created for testing by merging the topic branch (containing the proposed changes) into the target branch.

Use of the proposed merge commit should be preferred to using the tip of the topic branch.

### Every revision reachable from a branch was approved

Summary:
New revisions are created based ONLY on the changes that were approved.

Benefits:
Prevents a large class of internal threat attacks based on hiding a malicious commit in a series of good commits such that the malicious commit does not appear in the reviewed diff.

Requirements:

#### Context

In many organizations it is normal to review only the "net difference" between the tip of  the topic branch and the "best merge base", the closest shared commit between the topic and target branches computed at the time of review.

The topic branch may contain many commits of which not all were intended to represent a shippable state of the repository.

If a repository merges branches with a standard merge commit, all those unreviewed commits on the topic branch will become "reachable" from the protected branch by virtue of the multi-parent merge commit.

When a repo is cloned, all commits _reachable_ from the main branch are fetched and become accessible from the local checkout.

This combination of factors allows attacks where the victim performs a `git clone` operation followed by a `git reset --hard <unreviewed revision id>`.

#### Mitigations

##### Informed Review

The reviewer is able and encouraged to make an informed decision about what they're approving.
The reviewer MUST be presented with a full, meaningful content diff between the proposed revision and the previously reviewed revision.

It is not sufficient to indicate that a file changed without showing the contents.

##### Use only rebase operations on the protected branch

Require a squash merge strategy for the protected branch.

To guarantee that only commits representing reviewed diffs are cloned, the SCS MUST rebase (or "squash") the reviewed diff into a single new commit (the "squashed" commit) that has only a single parent (the revision previously pointed-to by the protected branch).
This is different than a standard merge commit strategy which would cause all the user-contributed commits to become reachable from the protected branch via the second parent.

It is not acceptable to replay the sequence of commits from the topic branch onto the protected branch.
The intent is to reduce the accepted changes to the exact diffs that were reviewed.
Constituent commits of the topic branch may or may not have been reviewed on an individual basis, and should not become reachable from the protected branch.

### Immutable Change Discussion

Summary:
The discussion around a change may often be as important as the change itself.

Intended for:
Large organizations, or any where discussion is an important part of the change management process.

Benefits:
From any revision, it's possible for future developers to read through the discussion that ultimately produced it.
This has many educational, forensics, and security auditing benefits.

Requirements:

The change management tool SHOULD record a description of the proposed change and all discussions / commentary related to it.

The change management tool MUST link this discussion to the revision itself. This is regularly done via commit metadata.

All collected content SHOULD be made immutable if the change is accepted.
It SHOULD NOT be possible to edit the discussion around a revision after it has been accepted.

### Fast moving repos and "merge trains"

Large organizations must keep the number of updates to key protected branches under certain limits to allow time for code review to happen.
For example, if a team tries to merge 60 change requests per hour into the `main` branch, the tip of the `main` branch would only be stable for about 1 minute.
This would leave only 1 minute for a new diff to be both generated and reviewed before it becomes stale again.

The normal way to work in this environment is to create a buffer branch (sometimes called a "train") to collect a certain number of approved changes.
In this model, when a change is approved for submission to the protected branch, it is added to the train branch instead.
After a certain amount of time, the train branch will be merged into the protected branch.
If there are problems detected with the contents on the train branch, it's normal for the whole train to be abandoned and a new train to be formed.
Approved changes will be re-applied to a new train in this scenario.

The key benefit to this approach is that the protected branch remains stable for longer, allowing more time for human and automatic code review.
A key downside to this approach is that organizations will not know the final revision id that represents a change until the entire train process completes.

A change review process will now be associated with multiple distinct revisions.

-   ID 1: The revision which was reviewed before concluding the change review process. It represents the ideal state of the protected branch applying only this proposed change.
-   ID 2: The revision created when the change is applied to the train branch. It represents the state of the protected branch _after other changes have been applied_.

It is important to note that no human or automatic review will have the chance to pre-approve ID2. This will appear to violate any organization policies that require pre-approval of changes before submission.
The SCS and the organization MUST protect this process in the same way they protect other artifact build pipelines.

At a minimum the SCS MUST issue an attestation that the revision id generated by a merged train is identical ("MERGESAME" in git terminology) to the state of the protected branch after applying each approved changeset in sequence.
No other content may be added or removed during this process.
