---
title: Producing artifacts
description: This page covers the detailed technical requirements for producing artifacts at each SLSA level. The intended audience is platform implementers and security engineers.
layout: specifications
---


This page covers the detailed technical requirements for producing artifacts at
each SLSA level. The intended audience is platform implementers and security
engineers.

For an informative description of the levels intended for all audiences, see
[Security Levels](levels.md). For background, see [Terminology](terminology.md). To
better understand the reasoning behind the requirements, see
[Threats and mitigations].

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Overview

### Build levels

In order to produce artifacts with a specific build level, responsibility is
split between the [Producer] and [Build platform]. The build platform MUST
strengthen the security controls in order to achieve a specific level while the
producer MUST choose and adopt a build platform capable of achieving a desired
build level, implementing any controls as specified by the chosen platform.

<table class="no-alternate">
<tr>
  <th>Implementer
  <th>Requirement
  <th>Degree
  <th>L1<th>L2<th>L3
<tr>
  <td rowspan=3><a href="#producer">Producer</a>
  <td colspan=2><a href="#choose-an-appropriate-build-platform">Choose an appropriate build platform</a>
  <td>✓<td>✓<td>✓
<tr>
  <td colspan=2><a href="#follow-a-consistent-build-process">Follow a consistent build process</a>
  <td>✓<td>✓<td>✓
<tr>
  <td colspan=2><a href="#distribute-provenance">Distribute provenance</a>
  <td>✓<td>✓<td>✓
<tr>
  <td rowspan=5><a href="#build-platform">Build platform</a>
  <td rowspan=3><a href="#provenance-generation">Provenance generation</a>
  <td><a href="#provenance-exists">Exists</a>
  <td>✓<td>✓<td>✓
<tr>
  <td><a href="#provenance-authentic">Authentic</a>
  <td> <td>✓<td>✓
<tr>
  <td><a href="#provenance-unforgeable">Unforgeable</a>
  <td> <td> <td>✓
<tr>
  <td rowspan=2><a href="#isolation-strength">Isolation strength</a>
  <td><a href="#hosted">Hosted</a>
  <td> <td>✓<td>✓
<tr>
  <td><a href="#isolated">Isolated</a>
  <td> <td> <td>✓
</table>

### Security Best Practices

While the exact definition of what constitutes a secure platform is beyond the
scope of this specification, all implementations MUST use industry security
best practices to be conformant to this specification. This includes, but is
not limited to, using proper access controls, securing communications,
implementing proper management of cryptographic secrets, doing frequent updates,
and promptly fixing known vulnerabilities.

Various relevant standards and guides can be consulted for that matter such as
the [CIS Critical Security
Controls](https://www.cisecurity.org/controls/cis-controls-list).

## Producer

[Producer]: #producer

A package's <dfn>producer</dfn> is the organization that owns and releases the
software. It might be an open-source project, a company, a team within a
company, or even an individual.

NOTE: There were more requirements for producers in the initial
[draft version (v0.1)](../v0.1/requirements#scripted-build) which impacted
how a package can be built. These were removed in the v1.0 specification and
will be reassessed and re-added as indicated in the
[future directions].

### Choose an appropriate build platform

The producer MUST select a build platform that is capable of reaching their
desired SLSA Build Level.

For example, if a producer wishes to produce a Build Level 3 artifact, they MUST
choose a builder capable of producing Build Level 3 provenance.

### Follow a consistent build process

The producer MUST build their artifact in a consistent
manner such that verifiers can form expectations about the build process. In
some implementations, the producer MAY provide explicit metadata to a verifier
about their build process. In others, the verifier will form their expectations
implicitly (e.g. trust on first use).

If a producer wishes to distribute their artifact through a [package ecosystem]
that requires explicit metadata about the build process in the form of a
configuration file, the producer MUST complete the configuration file and keep
it up to date. This metadata might include information related to the artifact's
source repository and build parameters.

### Distribute provenance

The producer MUST distribute provenance to artifact consumers. The producer
MAY delegate this responsibility to the
[package ecosystem], provided that the package ecosystem is capable of
distributing provenance.

## Build Platform

[Build platform]: #build-platform

A package's <dfn>build platform</dfn> is the infrastructure used to transform the
software from source to package. This includes the transitive closure of all
hardware, software, persons, and organizations that can influence the build. A
build platform is often a hosted, multi-tenant build service, but it could be a
system of multiple independent rebuilders, a special-purpose build platform used
by a single software project, or even an individual's workstation. Ideally, one
build platform is used by many different software packages so that consumers can
[minimize the number of trusted platforms]. For more background,
see [Build Model](terminology.md#build-model).

The build platform is responsible for providing two things: [provenance
generation] and [isolation between builds]. The [Build level](levels.md#build-track) describes
the degree to which each of these properties is met.

### Provenance generation

[Provenance generation]: #provenance-generation

The build platform is responsible for generating provenance describing how the
package was produced.

The SLSA Build level describes the overall provenance integrity according to
minimum requirements on its:

-   *Completeness:* What information is contained in the provenance?
-   *Authenticity:* How strongly can the provenance be tied back to the builder?
-   *Accuracy:* How resistant is the provenance generation to tampering within
    the build process?

<table>
<tr><th>Requirement<th>Description<th>L1<th>L2<th>L3

<tr id="provenance-exists"><td>Provenance Exists<td>

The build process MUST generate provenance that unambiguously identifies the
output package by cryptographic digest and describes how that package was
produced. The format MUST be acceptable to the [package ecosystem] and/or
[consumer](verifying-artifacts.md#consumer).

It is RECOMMENDED to use the [SLSA Provenance] format and [associated suite]
because it is designed to be interoperable, universal, and unambiguous when
used for SLSA. See that format's documentation for requirements and
implementation guidelines.

If using an alternate format, it MUST contain the equivalent information as SLSA
Provenance at each level and SHOULD be bi-directionally translatable to SLSA
Provenance.

-   *Completeness:* Best effort. The provenance at L1 SHOULD contain sufficient
    information to catch mistakes and simulate the user experience at higher
    levels in the absence of tampering. In other words, the contents of the
    provenance SHOULD be the same at all Build levels, but a few fields MAY be
    absent at L1 if they are prohibitively expensive to implement.
-   *Authenticity:* No requirements.
-   *Accuracy:* No requirements.

[SLSA Provenance]: ../../spec/v1.0/provenance.md
[associated suite]: ../../attestation-model#recommended-suite

<td>✓<td>✓<td>✓
<tr id="provenance-authentic"><td>Provenance is Authentic<td>

*Authenticity:* Consumers MUST be able to validate the authenticity of the
provenance attestation in order to:

-   *Ensure integrity:* Verify that the digital signature of the provenance
    attestation is valid and the provenance was not tampered with after the
    build.
-   *Define trust:* Identify the build platform and other entities that are
    necessary to trust in order to trust the artifact they produced.

This SHOULD be through a digital signature from a private key accessible only
to the build platform component that generated the provenance attestation.

While many constraints affect choice of signing methodologies, it is
RECOMMENDED that build platforms use signing methodologies which improve the
ability to detect and remediate key compromise, such as methods which rely on
transparency logs or, when transparency isn't appropriate, time stamping
services.

Authenticity allows the consumer to trust the contents of the provenance
attestation, such as the identity of the build platform.

*Accuracy:* The provenance MUST be generated by the control plane (i.e. within
the trust boundary [identified in the provenance]) and not by a tenant of the
build platform (i.e. outside the trust boundary), except as noted below.

-   The data in the provenance MUST be obtained from the build platform, either
    because the generator *is* the build platform or because the provenance
    generator reads the data directly from the build platform.
-   The build platform MUST have some security control to prevent tenants from
    tampering with the provenance. However, there is no minimum bound on the
    strength. The purpose is to deter adversaries who might face legal or
    financial risk from evading controls.
-   Exceptions for fields that MAY be generated by a tenant of the build platform:
    -   The names and cryptographic digests of the output artifacts, i.e.
        `subject` in [SLSA Provenance]. See [forge output digest of the
        provenance] for explanation of why this is acceptable.
    -   Any field that is not marked as REQUIRED for Build L2. For example,
        `resolvedDependencies` in [SLSA Provenance] MAY be tenant-generated at
        Build L2. Builders SHOULD document any such cases of tenant-generated
        fields.

*Completeness:* SHOULD be complete.

-   There MAY be [external parameters] that are not sufficiently captured in
    the provenance.
-   Completeness of resolved dependencies is best effort.

<td> <td>✓<td>✓
<tr id="provenance-unforgeable"><td>Provenance is Unforgeable<td>

*Accuracy:* Provenance MUST be strongly resistant to forgery by tenants.

-   Any secret material used for authenticating the provenance, for example the
    signing key used to generate a digital signature, MUST be stored in a secure
    management system appropriate for such material and accessible only to the
    build service account.
-   Such secret material MUST NOT be accessible to the environment running
    the user-defined build steps.
-   Every field in the provenance MUST be generated or verified by the build
    platform in a trusted control plane. The user-controlled build steps MUST
    NOT be able to inject or alter the contents, except as noted in [Provenance
    is Authentic](#provenance-authentic). (Build L3 does not require additional
    fields beyond those of L2.)

*Completeness:* SHOULD be complete.

-   [External parameters] MUST be fully enumerated.
-   Completeness of resolved dependencies is best effort.

Note: This requirement was called "non-falsifiable" in the initial
[draft version (v0.1)](../../spec/v0.1/requirements#non-falsifiable).

<td> <td> <td>✓
</table>

### Isolation strength

[Isolation strength]: #isolation-strength
[Isolation between builds]: #isolation-strength

The build platform is responsible for isolating between builds, even within the
same tenant project. In other words, how strong of a guarantee do we have that
the build really executed correctly, without external influence?

The SLSA Build level describes the minimum bar for isolation strength. For more
information on assessing a build platform's isolation strength, see
[Verifying build platforms](verifying-systems.md).

<table>
<tr><th>Requirement<th>Description<th>L1<th>L2<th>L3

<tr id="hosted">
<td>Hosted
<td>

All build steps ran using a hosted build platform on shared or dedicated
infrastructure, not on an individual's workstation.

Examples: GitHub Actions, Google Cloud Build, Travis CI.

<td> <td>✓<td>✓
<tr id="isolated">
<td>Isolated
<td>

The build platform ensured that the build steps ran in an isolated environment,
free of unintended external influence. In other words, any external influence on
the build was specifically requested by the build itself. This MUST hold true
even between builds within the same tenant project.

The build platform MUST guarantee the following:

-   It MUST NOT be possible for a build to access any secrets of the build
    platform, such as the provenance signing key, because doing so would
    compromise the authenticity of the provenance.
-   It MUST NOT be possible for two builds that overlap in time to influence one
    another, such as by altering the memory of a different build process running
    on the same machine.
-   It MUST NOT be possible for one build to persist or influence the build
    environment of a subsequent build. In other words, an ephemeral build
    environment MUST be provisioned for each build.
-   It MUST NOT be possible for one build to inject false entries into a build
    cache used by another build, also known as "cache poisoning". In other
    words, the output of the build MUST be identical whether or not the cache is
    used.
-   The build platform MUST NOT open services that allow for remote influence
    unless all such interactions are captured as `externalParameters` in the
    provenance.

There are no sub-requirements on the build itself. Build L3 is limited to
ensuring that a well-intentioned build runs securely. It does not require that
a build platform prevents a producer from performing a risky or insecure build. In
particular, the "Isolated" requirement does not prohibit a build from calling
out to a remote execution service or a "self-hosted runner" that is outside the
trust boundary of the build platform.

NOTE: This requirement was split into "Isolated" and "Ephemeral Environment"
in the initial [draft version (v0.1)].

NOTE: This requirement is not to be confused with "Hermetic", which roughly
means that the build ran with no network access. Such a requirement requires
substantial changes to both the build platform and each individual build, and is
considered in the [future directions].

<td> <td> <td>✓
</table>

[external parameters]: ../../spec/v1.0/provenance.md#externalParameters
[identified in the provenance]: ../../spec/v1.0/provenance.md#model
[package ecosystem]: verifying-artifacts.md#package-ecosystem
[draft version (v0.1)]: ../../spec/v0.1/requirements.md
[future directions]: ../../spec/v1.0/future-directions.md
[Threats and mitigations]: ../../spec/v1.0/threats.md
[forge output digest of the provenance]: ../../spec/v1.0/threats#forged-digest
[minimize the number of trusted platforms]: ../../spec/v1.0/principles.md
