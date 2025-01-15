---
title: How to SLSA for organizations
description: If you're looking to jump straight in and try SLSA, here's a quick start guide for the steps to take to reach the different SLSA levels.
layout: standard
---

This is a quick-start guide for organizations that want to adopt SLSA. Your
organization has two major responsibilities: choosing a target SLSA level for
your organization, and selecting tools that support your desired SLSA level.

## Choosing your SLSA level

For all [SLSA levels](/spec/v1.0/levels.md), you follow the same steps:

1)  Generate provenance, i.e., document your build process
2)  Make the provenance available, to allow downstream users to verify it

What differs for each level is the robustness of the build and provenance. For
more information about SLSA provenance see
[https://slsa.dev/provenance/](/provenance/v1).

SLSA Build levels are progressive: SLSA Build L3 includes all the guarantees of
SLSA Build L2, and SLSA Build L2 includes all the guarantees of SLSA Build L1.
Currently, though, the work required to achieve lower SLSA levels will not
necessarily accrue toward the work needed for higher levels, because achieving a
higher level may require migrating to a different build platform altogether.
For that reason, you should set a target level for your project or
organization to work towards and choose a build platform which supports the
target level, so as to avoid wasted work.

<a id="tooling"></a>

## Choose appropriate tools

Your organization's tooling needs depend on what role your organization plays in
the software supply chain. Organizations can be software producers, software
consumers, or both. In practice, most organizations that produce software also
consume software.

### For software consumers

As a software consumer, you need to verify that the software you consume meets
your chosen SLSA Build level.

Ideally, [package ecosystems](/spec/v1.0/terminology.md#package-model) verify
SLSA provenance for the packages they distribute. Check with the package
ecosystem where you get software to see if they verify SLSA provenance. If they
do, then inspect its verification practices to ensure that they meet your
needs. If they do not, then you may want to request that they add SLSA support.

Regardless of whether your package ecosystem verifies SLSA provenance, you may
wish to verify SLSA provenance yourself using tools such as
[`slsa-verifier`](https://github.com/slsa-framework/slsa-verifier).

To learn more about the verification process, see
[Verifying Artifacts](/spec/v1.0/verifying-artifacts.md).

### For software producers

As a software producer, you need to follow the requirements for your target
Build level. You also need to produce and distribute attestations that
demonstrate your software meets your desired SLSA Build level. SLSA
recommends either the [SLSA provenance format](/provenance/) or,
if you wish to keep the details of your build pipeline confidential, a
[SLSA Verification Summary Attestation (VSA)](/verification_summary/) although
the SLSA specification allows for other formats.

Ideally, build platforms produce SLSA provenance rather than individual
developers. If your organization...

-   does not use a build platform, then consider adopting one that supports
SLSA.
-   uses a third-party build platform, then check with its provider to see
if they support generating SLSA provenance at your desired level. If they do,
then follow their instructions for producing provenance. If they do not, then
consider requesting that they add SLSA support or adopting a tool that does
support SLSA.
-   maintains its own build platform, then add support for generating SLSA
provenance.

For more information about producing provenance, see
[Producing artifact](/spec/v1.0/requirements) and
[Verifying build platforms](/spec/v1.0/verifying-systems).

Ideally, [package ecosystems](/spec/v1.0/terminology.md#package-model)
distribute provenance alongside packages. If your organization...

-   distributes software through a third-party package ecosystem, then check
with its provider to see if they support distributing SLSA provenance. If they
do, then follow their instructions for distributing provenance. If they do not,
then consider requesting that they add SLSA support or using a package ecosystem
that does support SLSA.
-   distributes software itself (e.g. directly to consumers), then add SLSA
provenance to your set of package artifacts.

For more information about distributing provenance, see
[Distributing provenance](/spec/v1.0/distributing-provenance).
