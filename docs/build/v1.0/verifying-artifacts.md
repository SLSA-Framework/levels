---
title: Verifying artifacts
description: SLSA uses provenance to indicate whether an artifact is authentic or not, but provenance doesn't do anything unless somebody inspects it. SLSA calls that inspection verification, and this page describes how to verify artifacts and their SLSA provenenance. The intended audience is platform implementers, security engineers, and software consumers.
layout: specifications
---

SLSA uses provenance to indicate whether an artifact is authentic or not, but
provenance doesn't do anything unless somebody inspects it. SLSA calls that
inspection **verification**, and this page describes recommendations for how to
verify artifacts and their SLSA provenance.

This page is divided into several sections. The first describes the process
for verifying an artifact and its provenance against a set of expectations. The
second describes how to form the expectations used to verify provenance. The
third discusses architecture choices for where provenance verification can
happen.

## How to verify

Verification SHOULD include the following steps:

-   Ensuring that the builder identity is one of those in the map of trusted
    builder id's to SLSA level.
-   Verifying the signature on the provenance envelope.
-   Ensuring that the values for `buildType` and `externalParameters` in the
    provenance match the expected values. The package ecosystem MAY allow
    an approved list of `externalParameters` to be ignored during verification.
    Any unrecognized `externalParameters` SHOULD cause verification to fail.

![Threats covered by each step](images/supply-chain-threats-build-verification.svg)

See [Terminology](terminology.md) for an explanation of supply chain model and
[Threats & mitigations] for a detailed explanation of each threat.

**Note:** This section assumes that the provenance is in the recommended
[provenance format](/provenance/v1). If it is not, then the verifier SHOULD
perform equivalent checks on provenance fields that correspond to the ones
referenced here.

### Step 1: Check SLSA Build level

[Step 1]: #step-1-check-slsa-build-level

First, check the SLSA Build level by comparing the artifact to its provenance
and the provenance to a preconfigured root of trust. The goal is to ensure that
the provenance actually applies to the artifact in question and to assess the
trustworthiness of the provenance. This mitigates some or all of [threats] "E",
"F", "G", and "H", depending on SLSA Build level and where verification happens.

Once, when bootstrapping the verifier:

-   Configure the verifier's roots of trust, meaning the recognized builder
    identities and the maximum SLSA Build level each builder is trusted up to.
    Different verifiers might use different roots of trust, but usually a
    verifier uses the same roots of trust for all packages. This configuration
    is likely in the form of a map from (builder public key identity,
    `builder.id`) to (SLSA Build level) drawn from the SLSA Conformance
    Program (coming soon).

    <details>
    <summary>Example root of trust configuration</summary>

    The following snippet shows conceptually how a verifier's roots of trust
    might be configured using made-up syntax.

    ```jsonc
    "slsaRootsOfTrust": [
        // A builder trusted at SLSA Build L3, using a fixed public key.
        {
            "publicKey": "HKJEwI...",
            "builderId": "https://somebuilder.example.com/slsa/l3",
            "slsaBuildLevel": 3
        },
        // A different builder that claims to be SLSA Build L3,
        // but this verifier only trusts it to L2.
        {
            "publicKey": "tLykq9...",
            "builderId": "https://differentbuilder.example.com/slsa/l3",
            "slsaBuildLevel": 2
        },
        // A builder that uses Sigstore for authentication.
        {
            "sigstore": {
                "root": "global",  // identifies fulcio/rekor roots
                "subjectAlternativeNamePattern": "https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_generic_slsa3.yml@refs/tags/v*.*.*"
            }
            "builderId": "https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_generic_slsa3.yml@refs/tags/v*.*.*",
            "slsaBuildLevel": 3,
        }
        ...
    ],
    ```

    </details>

Given an artifact and its provenance:

1.  [Verify][validation-model] the envelope's signature using the roots of
    trust, resulting in a list of recognized public keys (or equivalent).
2.  [Verify][validation-model] that statement's `subject` matches the digest of
    the artifact in question.
3.  Verify that the `predicateType` is `https://slsa.dev/provenance/v1`.
4.  Look up the SLSA Build Level in the roots of trust, using the recognized
    public keys and the `builder.id`, defaulting to SLSA Build L1.

Resulting threat mitigation:

-   [Threat "E"]: SLSA Build L3 requires protection against compromise of the
    build process and provenance generation by an external adversary, such as
    persistence between builds or theft of the provenance signing key. In other
    words, SLSA Build L3 establishes that the provenance is accurate and
    trustworthy, assuming you trust the build platform.
    -   IMPORTANT: SLSA Build L3 does **not** cover compromise of the build
        platform itself, such as by a malicious insider. Instead, verifiers
        SHOULD carefully consider which build platforms are added to the roots
        of trust. For advice on establishing trust in build platforms, see
        [Verifying build platforms](verifying-systems).
-   [Threat "F"]: SLSA Build L2 covers tampering of the artifact or provenance
    after the build. This is accomplished by verifying the `subject` and
    signature in the steps above.
-   [Threat "G"]: Verification by the consumer or otherwise outside of the
    package registry covers compromise of the registry itself. (Verifying within
    the registry at publication time is also valuable, but does not cover Threat
    "G" or "I".)
-   [Threat "I"]: Verification by the consumer covers compromise of the package
    in transit. (Many ecosystems also address this threat using package
    signatures or checksums.)
    -   NOTE: SLSA does not yet cover adversaries tricking a consumer to use an
        unintended package, such as through typosquatting. Those threats are
        discussed in more detail under [Threat "H"].

[Threat "E"]: ../../spec/v1.0/threats#e-build-process
[Threat "F"]: ../../spec/v1.0/threats#f-artifact-publication
[Threat "G"]: ../../spec/v1.0/threats#g-distribution-channel
[Threat "H"]: ../../spec/v1.0/threats#h-package-selection
[Threat "I"]: ../../spec/v1.0/threats#i-usage

[validation-model]: https://github.com/in-toto/attestation/blob/main/docs/validation.md#validation-model

### Step 2: Check expectations

[verify-step-2]: #check-expectations

Next, check that the package's provenance meets your expectations for that
package in order to mitigate [threat "D"].

In our threat model, the adversary has ability to invoke a build and to publish
to the registry. The adversary is not able to write to the source repository, nor do
they have insider access to any trusted systems. Your expectations SHOULD be
sufficient to detect or prevent this adversary from injecting unofficial
behavior into the package.

You SHOULD compare the provenance against expected values for at least the
following fields:

| What | Why
| ---- | ---
| Builder identity from [Step 1] | To prevent an adversary from building the correct code on an unintended platform
| Canonical source repository | To prevent an adversary from building from an unofficial fork (or other disallowed source)
| `buildType` | To ensure that `externalParameters` are interpreted as intended
| `externalParameters` | To prevent an adversary from injecting unofficial behavior

Verification tools SHOULD reject unrecognized fields in `externalParameters` to
err on the side of caution. It is acceptable to allow a parameter to have a
range of values (possibly any value) if it is known that any value in the range
is safe. JSON comparison is sufficient for verifying parameters.

TIP: Difficulty in forming meaningful expectations about `externalParameters` can
be a sign that the `buildType`'s level of abstraction is too low. For example,
`externalParameters` that record a list of commands to run is likely impractical
to verify because the commands change on every build. Instead, consider a
`buildType` that defines the list of commands in a configuration file in a
source repository, then put only the source repository in
`externalParameters`. Such a design is easier to verify because the source
repository is constant across builds.

[Threat "D"]: ../../spec/v1.0/threats#d-external-build-parameters

### Step 3: (Optional) Check dependencies recursively

[verify-step-3]: #step-3-optional-check-dependencies-recursively

Finally, recursively check the `resolvedDependencies` as available and to the
extent desired. Note that SLSA v1.0 does not have any requirements on the
completeness or verification of `resolvedDependencies`. However, one might wish
to verify dependencies in order to mitigate [dependency threats] and protect against
threats further up the supply chain. If `resolvedDependencies` is incomplete,
these checks can be done on a best-effort basis.

A [Verification Summary Attestation (VSA)][VSA] can make dependency verification
more efficient by recording the result of prior verifications. A trimming
heuristic or exception mechanism is almost always necessary when verifying
dependencies because there will be transitive dependencies that are SLSA Build
L0. (For example, consider the compiler's compiler's compiler's ... compiler.)

[dependency threats]: ../../spec/v1.0/threats#dependency-threats
[VSA]: /verification_summary
[threats]: ../../spec/1.0/threats

## Forming Expectations

<dfn>Expectations</dfn> are known provenance values that indicate the
corresponding artifact is authentic. For example, a package ecosystem may
maintain a mapping between package names and their canonical source
repositories. That mapping constitutes a set of expectations.

Possible models for forming expectations include:

-   **Trust on first use:** Accept the first version of the package as-is. On
    each version update, compare the old provenance to the new provenance and
    alert on any differences. This can be augmented by having rules about what
    changes are benign, such as a parameter known to be safe or a heuristic
    about safe git branches or tags.

-   **Defined by producer:** The package producer tells the verifier what their
    expectations ought to be. In this model, the verifier SHOULD provide an
    authenticated communication mechanism for the producer to set the package's
    expectations, and there SHOULD be some protection against an adversary
    unilaterally modifying them. For example, modifications might require
    two-party control, or consumers might have to accept each policy change
    (another form of trust on first use).

-   **Defined in source:** The source repository tells the verifier what their
    expectations ought to be. In this model, the package name is immutably bound
    to a source repository and all other external parameters are defined in the
    source repository. This is how the Go ecosystem  works, for example, since
    the package name *is* the source repository location.

It is important to note that expectations are tied to a *package name*, whereas
provenance is tied to an *artifact*. Different versions of the same package name
will likely have different artifacts and therefore different provenance. Similarly, an
artifact might have different names in different package ecosystems but use the same
provenance file.

## Architecture options

There are several options (non-mutually exclusive) for where provenance verification
can happen: the package ecosystem at upload time, the consumers at download time, or
via a continuous monitoring system. Each option comes with its own set of
considerations, but all are valid and at least one SHOULD be used.

More than one component can verify provenance. For example, even if a package
ecosystem verifies provenance, consumers who get artifacts from that package
ecosystem might wish to verify provenance themselves for defense in depth. They
can do so using either client-side verification tooling or by polling a
monitor.

<!-- **TODO** Add a diagram. -->

### Package ecosystem

[Package ecosystem]: #package-ecosystem

A <dfn>package ecosystem</dfn> is a set of rules and conventions governing
how packages are distributed. Every package artifact has an ecosystem, whether it is
formal or ad-hoc. Some ecosystems are formal, such as language distribution
(e.g. [Python/PyPA](https://www.pypa.io)), operating system distribution (e.g.
[Debian/Apt](https://wiki.debian.org/DebianRepository/Format)), or artifact
distribution (e.g. [OCI](https://github.com/opencontainers/distribution-spec)).
Other ecosystems are informal, such as a convention used within a company. Even
ad-hoc distribution of software, such as through a link on a website, is
considered an "ecosystem". For more background, see
[Package Model](terminology.md#package-model).

During package upload, a package ecosystem can ensure that the artifact's
provenance matches the expected values for that package name's provenance before
accepting it into the package registry.  This option is RECOMMENDED whenever
possible because doing so benefits all of the package ecosystem's clients.

The package ecosystem is responsible for making its expectations available to
consumers, reliably redistributing artifacts and provenance, and providing tools
to enable safe artifact consumption (e.g. whether an artifact meets
expectations).

### Consumer

[Consumer]: #consumer

A package artifact's <dfn>consumer</dfn> is the organization or individual that uses the
package artifact.

Consumers can form their own expectations for artifacts or use the default
expectations provided by the package producer and/or package ecosystem.
When forming their own expectations, the consumer uses client-side verification tooling to ensure
that the artifact's provenance matches their expectations for that package
before use (e.g. during installation or deployment). Client-side verification
tooling can be either standalone, such as
[slsa-verifier](https://github.com/slsa-framework/slsa-verifier), or built into
the package ecosystem client.

### Monitor

[Monitor]: #monitor

A <dfn>monitor</dfn> is a service that verifies provenance for a set
of packages and publishes the result of that verification. The set of
packages verified by a monitor is arbitrary, though it MAY mimic the set
of packages published through one or more package ecosystems. The monitor
SHOULD publish its expectations for all the packages it verifies.

Consumers can continuously poll a monitor to detect artifacts that
do not meet the monitor's expectations. Detecting artifacts that fail
verification is of limited benefit unless a human or automated system takes
action in response to the failed verification.

[Threats & mitigations]: ../../spec/v1.0/threats.md
