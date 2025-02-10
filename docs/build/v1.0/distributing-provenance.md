---
title: Distributing provenance
description: This page covers the detailed technical requirements for distributing provenance at each SLSA level. The intended audience is platform implementers and software distributors.
layout: specifications
---

In order to make provenance for artifacts available after generation
for verification, SLSA requires the distribution and verification of provenance
metadata in the form of SLSA attestations.

This document provides specifications for distributing provenance, and the
relationship between build artifacts and provenance (build attestations). It is
primarily concerned with artifacts for ecosystems that distribute build
artifacts, but some attention is also paid to ecosystems that distribute
container images or only distribute source artifacts, as many of the same
principles generally apply to any artifact or group of artifacts.

In addition, this document is primarily for the benefit of artifact
distributors, to understand how they can adopt the distribution of SLSA
provenance. It is primarily concerned with the means of distributing
attestations and the relationship of attestations to build artifacts, and not
with the specific format of the attestation itself.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Background

The [package ecosystem]'s maintainers are responsible for reliably
redistributing artifacts and provenance, making the producers' expectations
available to consumers, and providing tools to enable safe artifact consumption
(e.g. whether an artifact meets its producer's expectations).

## Relationship between releases and attestations

Attestations SHOULD be bound to artifacts, not releases.

A single "release" of a project, package, or library might include multiple
artifacts. These artifacts result from builds on different platforms,
architectures or environments. The builds need not happen at roughly the same
point in time and might even span multiple days.

It is often difficult or impossible to determine when a release is 'finished'
because many ecosystems allow adding new artifacts to old releases when adding
support for new platforms or architectures. Therefore, the set of attestations
for a given release MAY grow over time as additional builds and attestations
are created.

Thus, package ecosystems SHOULD support multiple individual attestations per
release. At the time of a given build, the relevant provenance for that build
can be added to the release, depending on the relationship to the given
artifacts.

## Relationship between artifacts and attestations

Package ecosystems SHOULD support a one-to-many relationship from build
artifacts to attestations to ensure that anyone is free to produce and publish
any attestation they might need. However, while there are lots of possible
attestations that can have a relationship to a given artifact, in this context
SLSA is primarily concerned with build attestations, i.e. provenance, and as
such, this specification only considers build attestations, produced by the
same maintainers as the artifacts themselves.

By providing provenance alongside an artifact in the manner specified by a
given ecosystem, maintainers are considered to be 'elevating' these build
attestations above all other possible attestations that could be provided by
third parties for a given artifact. The ultimate goal is for maintainers to
provide the provenance necessary for a repository to be able to verify some
potential policy that requires a certain SLSA level for publication, not
support the publication of arbitrary attestations by third parties.

As a result, this provenance SHOULD accompany the artifact at publish time, and
package ecosystems SHOULD provide a way to map a given artifact to its
corresponding attestations. The mappings can be either implicit (e.g. require a
custom filename schema that uniquely identifies the provenance over other
attestation types) or explicit (e.g. it could happen as a de-facto standard
based on where the attestation is published).

The provenance SHOULD have a filename that is directly related to the build
artifact filename. For example, for an artifact `<filename>.<extension>`, the
attestation is `<filename>.attestation` or some similar extension (for example
[in-toto](https://in-toto.io/) recommends `<filename>.intoto.jsonl`.)

## Where attestations are published

There are a number of opportunities and venues to publish attestations during
and after the build process. Producers MUST publish attestations in at least
one place, and SHOULD publish attestations in more than one place:

-   **Publish attestations alongside the source repository releases**: If the
    source repository hosting provider offers an artifact "release" feature,
    such as [GitHub
    releases](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases)
    or [GitLab releases](https://docs.gitlab.com/ee/user/project/releases/),
    producers SHOULD include provenance as part of such releases. This option
    has the benefit of requiring no changes to the package registry to support
    provenance formats, but has the disadvantage of putting the source
    repository hosting providing in the critical path for installers that want to
    verify policy at build-time.
-   **Publish attestations alongside the artifact in the package registry**:
    Many software repositories already support some variety of publishing 1:1
    related files alongside an artifact, sometimes known as “sidecar files”.
    For example, PyPI supports publishing `.asc` files representing the PGP
    signature for an artifact with the same filename (but different extension).
    This option requires the mapping between artifact and attestation (or
    attestation vessel) to be 1:1.
-   **Publish attestations elsewhere, record their existence in a transparency
    log**: Once an attestation has been generated and published for a build, a
    hash of the attestation and a pointer to where it is indexed SHOULD be
    published to a third-party transparency log that exists outside the source
    repository and package registry. Not only are transparency logs such as
    [Rekor from Sigstore](https://github.com/sigstore/rekor) guaranteed
    to be immutable, but they typically also make monitoring easier.
    Requiring the presence of the attestation in a monitored transparency log
    during verification helps ensure the attestation is trustworthy.

Combining these options gives us a process for bootstrapping SLSA adoption
within an ecosystem, even if the package registry doesn't support publishing
attestations. First, interested projects modify their release process to
produce SLSA provenance. Then, they publish that provenance to their source
repository. Finally, they publish the provenance to the package registry, if
and when the registry supports it.

Long-term, package registries SHOULD support uploading and distributing
provenance alongside the artifact. This model is preferred for two reasons:

-   trust: clients already trust the package registry as the source of their
    artifacts, and don't need to trust an additional service;
-   reliability: clients already depend on the package registry as part of
    their critical path, so distributing provenance via the registry avoids
    adding an additional point of failure.

Short term, consumers of build artifacts can bootstrap a manual policy by using
the source repository only for projects that publish all artifacts and
attestations to the source repository, and later extend this to all artifacts
published to the package registry via the canonical installation tools once
a given ecosystem supports them.

## Immutability of attestations

Attestations SHOULD be immutable. Once an attestation is published as it
corresponds to a given artifact, that attestation is immutable and cannot be
overwritten later with a different attestation that refers to the same
artifact. Instead, a new release (and new artifacts) SHOULD be created.

## Format of the attestation

The provenance is available to the consumer in a format that the consumer
accepts. The format SHOULD be in-toto [SLSA Provenance](/provenance), but
another format MAY be used if both producer and consumer agree and it meets all
the other requirements.

## Considerations for source-based ecosystems

Some ecosystems have support for installing directly from source repositories
(an option for Python/`pip`, Go, etc). In these cases, there is no need to
publish or verify provenance because there is no "build" step that translates
between a source repository and an artifact that is being installed.

However, for ecosystems that install from source repositories _via_ some
intermediary (e.g. [Homebrew installing from GitHub release artifacts generated
from the repository or GitHub Packages](https://docs.brew.sh/Bottles), [Go
installing through the Go module proxy](https://proxy.golang.org/)), these
ecosystems distribute "source archives" that are not the bit-for-bit identical
form from version control. These intermediaries are transforming the original
source repository in some way that constitutes a "build" and as a result SHOULD
be providing build provenance for this "package", and the recommendations
outlined here apply.

[package ecosystem]: verifying-artifacts.md#package-ecosystem
