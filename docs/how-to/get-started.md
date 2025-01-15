---
title: Get started
description: If you're looking to jump straight in and try SLSA, here's a quick start guide for the steps to take to reach the different SLSA levels.
layout: standard
---

If you're looking to jump straight in and try SLSA, here's a quick start guide
for the steps to take to reach the different SLSA levels.

## Choosing your SLSA level

For all [SLSA levels](/spec/v1.0/levels), you follow the same steps:

1)  Generate provenance, i.e., document your build process
2)  Make the provenance available, to allow downstream users to verify it

What differs for each level is the robustness of the build and provenance. For more information about provenance see [https://slsa.dev/provenance/](/provenance/).

The tools discussed in this guide are freely available and believed to meet SLSA expectations. The [Builder SLSA levels section](#builder-slsa-levels) provides a more complete list. If you think a build option is misclassified or want to add one, please [open an issue](https://github.com/slsa-framework/slsa/issues) or [submit a PR](https://github.com/slsa-framework/slsa/pulls) against this page.

SLSA levels are progressive: SLSA 3 includes all the guarantees of SLSA 2, and SLSA 2 includes all the guarantees of SLSA 1. Currently, though, the work required to achieve lower SLSA levels will not necessarily accrue toward the work needed for higher levels, because achieving a higher level may require migrating to a different build platform altogether. For that reason, **you should start with the highest level that’s possible for your project or organization to avoid wasted work**.

The SLSA level you implement depends on your current build situation:

-   If you are using GitHub Actions, jump directly to [SLSA 3](#SLSA3). You do not need to implement SLSA 1 or SLSA 2.
-   If you are using [FRSCA](https://github.com/buildsec/frsca), jump to [SLSA 2](#SLSA2). You do not need to implement SLSA 1.
-   If you’re using any other build platform, consult its documentation to find out what SLSA level it supports and how to proceed. If your build platform doesn't support the SLSA level you are aiming for or you do not use any build platform, you should consider adopting one that does, such as GitHub Actions and FRSCA discussed below.

### Provenance verification

Various build methods require different methods to verify provenance. The SLSA community already has some verification methods and is working on additional solutions for provenance verification. Even if there is currently no simple verification method for a particular build platform, adopting these builders now means you will be “SLSA ready” when more ecosystem tooling is released. Note that when you create provenance, you must supply to users a method for interpreting what you have created.

### Provenance formatting

Your provenance format depends on who will be consuming it. See [provenance](/provenance) for an explanation of which format to choose.

### Provenance storage

Containers have a standard place to put the provenance in the OCI container registry. With time, the SLSA community hopes to create standard ecosystem-based repositories for provenance. For now, the convention is to keep the provenance attestation with your artifact. Though [Sigstore](https://www.sigstore.dev/) is becoming more and more popular, the format of the provenance is currently tool-specific.

<a id="SLSA1"></a>

## SLSA 1

As mentioned before, if you don't already use a build platform or CI/CD, you should consider adopting a platform that supports SLSA 2 or SLSA 3. This will make the following steps easier and provide for higher SLSA levels later on. Individual developers who wish to put a minimal amount of security on their builds can use SLSA 1.

SLSA 1 requires that the build process is documented. Some tools suggested below also support signed provenance. Though not required for SLSA 1, signing your provenance increases trust in the document by showing that it has not been tampered with.

### Tooling

A build configuration file (i.e., GitHub workflow) qualifies for SLSA 1. It would be considered unsigned, unformatted provenance.

### Build platform plugins or extensions

The following options work with your build platform to produce unsigned, formatted provenance. They do not qualify for SLSA 2 because they are unsigned and not run by the hosted server:

-   [Azure DevOps extension](https://github.com/slsa-framework/azure-devops-demo)
-   [Jenkins SLSA generator](https://github.com/slsa-framework/slsa-jenkins-generator)
-   [Jenkins plugin](https://plugins.jenkins.io/in-toto/)

Downstream users may verify the provenance with [Cue Policies](https://cuelang.org/docs/).

### Build observers with hosted platforms

The following options are user-configured inside a hosted platform. They observe the build process and produce signed, formatted provenance. These options do not qualify for SLSA 2 because they are configured by users, not the hosted platform.

Downstream users may verify the provenance with Cue policies and the signature with Cosign.

**Note:** If you are using one of these options with GitHub Actions, jump to SLSA 3 and use the builder itself to generate provenance.

-   If you’re using [Tejolote](https://github.com/kubernetes-sigs/tejolote) with GitHub Actions, jump to SLSA 3 and generate provenance directly from the builder.
-   [Tekton Chains](https://tekton.dev/docs/chains/signed-provenance-tutorial/) – custom resource definition controller that can generate provenance for Kubernetes OCI containers

<a id="SLSA2"></a>

## SLSA 2

To achieve SLSA 2, the goals are to:

-   Run your build on a hosted platform that generates and signs provenance
-   Publish the provenance to allow downstream users to verify it

The following is a SLSA 2 builder:

-   [Factory for Repeatable Secure Creation of Artifacts (FRSCA)](https://github.com/buildsec/frsca)

FRSCA is an OpenSSF project that aims at offering a full build pipeline. It is not yet generally available. It qualifies as a SLSA 2 builder because regular users of the platform are not able to inject or alter the contents of the provenance it generates. FRSCA produces signed, formatted provenance that can be verified by the generic SLSA verifier.

<a id="SLSA3"></a>

## SLSA 3

To achieve SLSA 3, you must:

-   Run your build on a hosted platform that generates and signs provenance
-   Ensure that build runs cannot influence each other
-   Produce signed provenance that can be verified as authentic

### GitHub Actions

If you are building on GitHub Actions, adopt the [Language-agnostic GitHub provenance generator / builder](https://github.com/slsa-framework/slsa-github-generator) to make your build qualify for SLSA 3. Consumers can use the [Generic SLSA Verifier](https://github.com/slsa-framework/slsa-verifier) for provenance verification.

## Builder SLSA levels

The following table shows known build software packages and the potential SLSA level they are believed to qualify for.  Potential levels are reached with proper provenance. Again, if you think a build option is misclassified or want to add one, please [open an issue](https://github.com/slsa-framework/slsa/issues) or [submit a PR](https://github.com/slsa-framework/slsa/pulls) against this page.

Note that this list is provided "as is". OpenSSF makes no claim as to the reliability of this information. A certification program is under development to provide a more definitive list.

| Builder                  | Potential SLSA Level
|--------------------------|:--------------------:
| FRSCA                    |           2
| GitHub Actions           |           3
| Google Cloud Build       |           3
| No Hosted Build Platform |           1
