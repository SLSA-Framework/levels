---
title: Current activities
description: There's an active community of members, contributors and collaborators working to enhance the SLSA specification with updates to existing and new tracks. This page provides a summary of current ongoing activities.
layout: standard
---

Since the release of <a href="spec/v1.0/">SLSA v1.0</a> in 2023,
the SLSA community has been hard at work to expand the breadth
and depth of the specification with updates and new tracks.

Learn how you can [get involved](/community#get-involved)!

### Source track

A Source track will provide protection against tampering of the source code
prior to the build.

The current [draft version](/spec/draft/source-requirements.md) describes levels
of increasing tamper resistance and ways consumers might verify properties
of source revisions using SLSA source provenance attestations.

### Build Environment track

The goal of a Build Environment track is to enable the detection of tampering
with core components of the compute environment executing builds.

The current [draft version](/spec/draft/attested-build-env-levels.md)
of the Build Environment track includes the following requirements:

-   Generation and verification of SLSA Build Provenance for build images.
-   Validation of initial build environment system state against known good
    reference values.
-   Deployment of the hosted build platform on a compute system that supports
    system state measurement and attestation capabilities at the hardware level.

These requirements are **subject to significant change** while this track
is in draft.

### Dependency track

Building upon the foundation laid by [S2C2F](https://openssf.org/projects/s2c2f/), the dependency track defines
requirements for consuming dependencies.
