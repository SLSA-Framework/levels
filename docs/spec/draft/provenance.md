---
title: Provenance
description: Description of SLSA provenance specification for verifying where, when, and how something was produced.
layout: standard
---
To trace software back to the source and define the moving parts in a complex
supply chain, provenance needs to be there from the very beginning. It's the
verifiable information about software artifacts describing where, when and how
something was produced. For higher SLSA levels and more resilient integrity
guarantees, provenance requirements are stricter and need a deeper, more
technical understanding of the predicate.

This document defines the following predicate type within the [in-toto
attestation] framework:

```json
"predicateType": "https://slsa.dev/provenance/v1"
```

> Important: Always use the above string for `predicateType` rather than what is
> in the URL bar. The `predicateType` URI will always resolve to the latest
> minor version of this specification. See [parsing rules](#parsing-rules) for
> more information.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Purpose

Describe how an artifact or set of artifacts was produced so that:

-   Consumers of the provenance can verify that the artifact was built according
    to expectations.
-   Others can rebuild the artifact, if desired.

This predicate is the RECOMMENDED way to satisfy the SLSA v1.0 [provenance
requirements].

## Model

Provenance is an attestation that a particular build platform produced a set of
software artifacts through execution of the `buildDefinition`.

![Build Model](images/provenance-model.svg)

The model is as follows:

-   Each build runs as an independent process on a multi-tenant build platform.
    The `builder.id` identifies this platform, representing the transitive
    closure of all entities that are [trusted] to faithfully run the build and
    record the provenance. (Note: The same model can be used for platform-less
    or single-tenant build platforms.)

    -   The build platform implementer SHOULD define a security model for the build
        platform in order to clearly identify the platform's boundaries, actors,
        and interfaces. This model SHOULD then be used to identify the transitive
        closure of the trusted build platform for the `builder.id` as well as the
        trusted control plane.

-   The build process is defined by a parameterized template, identified by
    `buildType`. This encapsulates the process that ran, regardless of what
    platform ran it. Often the build type is specific to the build platform
    because most build platforms have their own unique interfaces.

-   All top-level, independent inputs are captured by the parameters to the
    template. There are two types of parameters:

    -   `externalParameters`: the external interface to the build. In SLSA,
        these values are untrusted; they MUST be included in the provenance and
        MUST be verified downstream.

    -   `internalParameters`: set internally by the platform. In SLSA, these
        values are trusted because the platform is trusted; they are OPTIONAL
        and need not be verified downstream. They MAY be included to enable
        reproducible builds, debugging, or incident response.

-   All artifacts fetched during initialization or execution of the build
    process are considered dependencies, including those referenced directly by
    parameters. The `resolvedDependencies` captures these dependencies, if
    known. For example, a build that takes a git repository URI as a parameter
    might record the specific git commit that the URI resolved to as a
    dependency.

-   During execution, the build process might communicate with the build
    platform's control plane and/or build caches. This communication is not
    captured directly in the provenance, but is instead implied by `builder.id`
    and subject to [SLSA Build Requirements](../../build/v1.0/requirements.md). Such
    communication SHOULD NOT influence the definition of the build; if it does,
    it SHOULD go in `resolvedDependencies` instead.

-   Finally, the build process outputs one or more artifacts, identified by
    `subject`.

For concrete examples, see [index of build types](#index-of-build-types).

## Parsing rules

This predicate follows the in-toto attestation [parsing rules]. Summary:

-   Consumers MUST ignore unrecognized fields unless otherwise noted.
-   The `predicateType` URI includes the major version number and will always
    change whenever there is a backwards incompatible change.
-   Minor version changes are always backwards compatible and "monotonic."
    Such changes do not update the `predicateType`.
-   Unset, null, and empty field values MUST be interpreted equivalently.

## Schema

### Summary

*NOTE: This summary (in cue) is informative. In the event of a
disagreement with the text description, the text is authoritative.*

<!-- Note: While this happens to be a valid cue file, we're really just using it
as a human-readable summary of the schema. We don't want readers to have to
understand cue. For that reason, we are not using any special cue syntax or
features. -->

```javascript
{% include_relative schema/provenance.cue %}
```

<details>
<summary>Protocol buffer schema</summary>

*NOTE: This summary (in protobuf) is informative. In the event of a
disagreement with the text description, the text is authoritative.*

Link: [provenance.proto](schema/provenance.proto)

*NOTE: This protobuf definition prioritises being a human-readable summary
of the schema for readers of the specification. A version of the protobuf
definition useful for code generation is maintained in the
[in-toto attestation] repository.*

```proto
{% include_relative schema/provenance.proto %}
```

</details>

### Provenance

*NOTE: This section describes the fields within `predicate`. For a description
of the other top-level fields, such as `subject`, see [Statement].*

[Provenance]: #provenance

REQUIRED for SLSA Build L1: `buildDefinition`, `runDetails`

<table>
<tr><th>Field<th>Type<th>Description

<tr id="buildDefinition"><td><code>buildDefinition</code>
<td><a href="#builddefinition">BuildDefinition</a><td>

The input to the build. The accuracy and completeness are implied by
`runDetails.builder.id`.

<tr id="runDetails"><td><code>runDetails</code>
<td><a href="#rundetails">RunDetails</a><td>

Details specific to this particular execution of the build.

</table>

### BuildDefinition

[BuildDefinition]: #builddefinition

REQUIRED for SLSA Build L1: `buildType`, `externalParameters`

<table>
<tr><th>Field<th>Type<th>Description

<tr id="buildType"><td><code>buildType</code>
<td>string (<a href="https://github.com/in-toto/attestation/blob/7aefca35a0f74a6e0cb397a8c4a76558f54de571/spec/v1/field_types.md#typeuri">TypeURI</a>)<td>

Identifies the template for how to perform the build and interpret the
parameters and dependencies.

The URI SHOULD resolve to a human-readable specification that includes: overall
description of the build type; schema for `externalParameters` and
`internalParameters`; unambiguous instructions for how to initiate the build given
this BuildDefinition, and a complete example. Example:
https://slsa-framework.github.io/github-actions-buildtypes/workflow/v1

<tr id="externalParameters"><td><code>externalParameters</code>
<td>object<td>

The parameters that are under external control, such as those set by a user or
tenant of the build platform. They MUST be complete at SLSA Build L3, meaning that
that there is no additional mechanism for an external party to influence the
build. (At lower SLSA Build levels, the completeness MAY be best effort.)

The build platform SHOULD be designed to minimize the size and complexity of
`externalParameters`, in order to reduce fragility and ease [verification].
Consumers SHOULD have an expectation of what "good" looks like; the more
information that they need to check, the harder that task becomes.

Verifiers SHOULD reject unrecognized or unexpected fields within
`externalParameters`.

<tr id="internalParameters"><td><code>internalParameters</code>
<td>object<td>

The parameters that are under the control of the entity represented by
`builder.id`. The primary intention of this field is for debugging, incident
response, and vulnerability management. The values here MAY be necessary for
reproducing the build. There is no need to [verify][Verification] these
parameters because the build platform is already trusted, and in many cases it is
not practical to do so.

<tr id="resolvedDependencies"><td><code>resolvedDependencies</code>
<td>array (<a href="https://github.com/in-toto/attestation/blob/7aefca35a0f74a6e0cb397a8c4a76558f54de571/spec/v1/resource_descriptor.md">ResourceDescriptor</a>)<td>

Unordered collection of artifacts needed at build time. Completeness is best
effort, at least through SLSA Build L3. For example, if the build script
fetches and executes "example.com/foo.sh", which in turn fetches
"example.com/bar.tar.gz", then both "foo.sh" and "bar.tar.gz" SHOULD be
listed here.

</table>

The BuildDefinition describes all of the inputs to the build. It SHOULD contain
all the information necessary and sufficient to initialize the build and begin
execution.

The `externalParameters` and `internalParameters` are the top-level inputs to the
template, meaning inputs not derived from another input. Each is an arbitrary
JSON object, though it is RECOMMENDED to keep the structure simple with string
values to aid verification. The same field name SHOULD NOT be used for both
`externalParameters` and `internalParameters`.

The parameters SHOULD only contain the actual values passed in through the
interface to the build platform. Metadata about those parameter values,
particularly digests of artifacts referenced by those parameters, SHOULD instead
go in `resolvedDependencies`. The documentation for `buildType` SHOULD explain
how to convert from a parameter to the dependency `uri`. For example:

```json
"externalParameters": {
    "repository": "https://github.com/octocat/hello-world",
    "ref": "refs/heads/main"
},
"resolvedDependencies": [{
    "uri": "git+https://github.com/octocat/hello-world@refs/heads/main",
    "digest": {"gitCommit": "7fd1a60b01f91b314f59955a4e4d4e80d8edf11d"}
}]
```

Guidelines:

-   Maximize the amount of information that is implicit from the meaning of
    `buildType`. In particular, any value that is boilerplate and the same
    for every build SHOULD be implicit.

-   Reduce parameters by moving configuration to input artifacts whenever
    possible. For example, instead of passing in compiler flags via an external
    parameter that has to be [verified][Verification] separately, require the
    flags to live next to the source code or build configuration so that
    verifying the latter automatically verifies the compiler flags.

-   In some cases, additional external parameters might exist that do not impact
    the behavior of the build, such as a deadline or priority. These extra
    parameters SHOULD be excluded from the provenance after careful analysis
    that they indeed pose no security impact.

-   If possible, architect the build platform to use this definition as its
    sole top-level input, in order to guarantee that the information is
    sufficient to run the build.

-   When build configuration is evaluated client-side before being sent to the
    server, such as transforming version-controlled YAML into ephemeral JSON,
    some solution is needed to make [verification] practical. Consumers need a
    way to know what configuration is expected and the usual way to do that is
    to map it back to version control, but that is not possible if the server
    cannot verify the configuration's origins. Possible solutions:

    -   (RECOMMENDED) Rearchitect the build platform to read configuration
        directly from version control,  recording the server-verified URI in
        `externalParameters` and the digest in `resolvedDependencies`.

    -   Record the digest in the provenance[^digest-param] and use a separate
        provenance attestation to link that digest back to version control. In
        this solution, the client-side evaluation is considered a separate
        "build" that SHOULD be independently secured using SLSA, though securing
        it can be difficult since it usually runs on an untrusted workstation.

-   The purpose of `resolvedDependencies` is to facilitate recursive analysis of
    the software supply chain. Where practical, it is valuable to record the
    URI and digest of artifacts that, if compromised, could impact the build. At
    SLSA Build L3, completeness is considered "best effort".

[^digest-param]: The `externalParameters` SHOULD reflect reality. If clients
    send the evaluated configuration object directly to the build server, record
    the digest directly in `externalParameters`. If clients upload the
    configuration object to a temporary storage location and send that location
    to the build server, record the location in `externalParameters` as a URI
    and record the `uri` and `digest` in `resolvedDependencies`.

### RunDetails

[RunDetails]: #rundetails

REQUIRED for SLSA Build L1: `builder`

<table>
<tr><th>Field<th>Type<th>Description

<tr id="runDetailsBuilder"><td><code>builder</code>
<td><a href="#builder">Builder</a><td>

Identifies the build platform that executed the invocation, which is trusted to
have correctly performed the operation and populated this provenance.

<tr id="runDetailsMetadata"><td><code>metadata</code>
<td><a href="#buildmetadata">BuildMetadata</a><td>

Metadata about this particular execution of the build.

<tr id="runDetailsByproducts"><td><code>byproducts</code>
<td>array (<a href="https://github.com/in-toto/attestation/blob/7aefca35a0f74a6e0cb397a8c4a76558f54de571/spec/v1/resource_descriptor.md">ResourceDescriptor</a>)<td>

Additional artifacts generated during the build that are not considered
the "output" of the build but that might be needed during debugging or
incident response. For example, this might reference logs generated during
the build and/or a digest of the fully evaluated build configuration.

In most cases, this SHOULD NOT contain all intermediate files generated during
the build. Instead, this SHOULD only contain files that are likely to be useful
later and that cannot be easily reproduced.

</table>

### Builder

[Builder]: #builder

REQUIRED for SLSA Build L1: `id`

<table>
<tr><th>Field<th>Type<th>Description

<tr id="builder.id"><td><code>id</code>
<td>string (<a href="https://github.com/in-toto/attestation/blob/7aefca35a0f74a6e0cb397a8c4a76558f54de571/spec/v1/field_types.md#typeuri">TypeURI</a>)<td>

URI indicating the transitive closure of the trusted build platform. This is
[intended](../../build/v1.0/verifying-artifacts#step-1-check-slsa-build-level)
to be the sole determiner of the SLSA Build level.

If a build platform has multiple modes of operations that have differing
security attributes or SLSA Build levels, each mode MUST have a different
`builder.id` and SHOULD have a different signer identity. This is to minimize
the risk that a less secure mode compromises a more secure one.

The `builder.id` URI SHOULD resolve to documentation explaining:

-   The scope of what this ID represents.
-   The claimed SLSA Build level.
-   The accuracy and completeness guarantees of the fields in the provenance.
-   Any fields that are generated by the tenant-controlled build process and not
    verified by the trusted control plane, except for the `subject`.
-   The interpretation of any extension fields.

<tr id="builderDependencies"><td><code>builderDependencies</code>
<td>array (<a href="https://github.com/in-toto/attestation/blob/7aefca35a0f74a6e0cb397a8c4a76558f54de571/spec/v1/resource_descriptor.md">ResourceDescriptor</a>)<td>

Dependencies used by the orchestrator that are not run within the workload
and that do not affect the build, but might affect the provenance generation
or security guarantees.

<tr id="builder.version"><td><code>version</code>
<td>map (string→string)<td>

Map of names of components of the build platform to their version.

</table>

The build platform, or <dfn>builder</dfn> for short, represents the transitive
closure of all the entities that are, by necessity, [trusted] to faithfully run
the build and record the provenance. This includes not only the software but the
hardware and people involved in running the service. For example, a particular
instance of [Tekton](https://tekton.dev/) could be a build platform, while
Tekton itself is not. For more info, see [Build
model](../../build/v1.0/terminology#build-model).

The `id` MUST reflect the trust base that consumers care about. How detailed to
be is a judgement call. For example, GitHub Actions supports both GitHub-hosted
runners and self-hosted runners. The GitHub-hosted runner might be a single
identity because it's all GitHub from the consumer's perspective. Meanwhile,
each self-hosted runner might have its own identity because not all runners are
trusted by all consumers.

Consumers MUST accept only specific signer-builder pairs. For example, "GitHub"
can sign provenance for the "GitHub Actions" builder, and "Google" can sign
provenance for the "Google Cloud Build" builder, but "GitHub" cannot sign for
the "Google Cloud Build" builder.

Design rationale: The builder is distinct from the signer in order to support
the case where one signer generates attestations for more than one builder, as
in the GitHub Actions example above. The field is REQUIRED, even if it is
implicit from the signer, to aid readability and debugging. It is an object to
allow additional fields in the future, in case one URI is not sufficient.

### BuildMetadata

[BuildMetadata]: #buildmetadata

REQUIRED: (none)

<table>
<tr><th>Field<th>Type<th>Description

<tr id="invocationId"><td><code>invocationId</code>
<td>string<td>

Identifies this particular build invocation, which can be useful for finding
associated logs or other ad-hoc analysis. The exact meaning and format is
defined by `builder.id`; by default it is treated as opaque and case-sensitive.
The value SHOULD be globally unique.

<tr id="startedOn"><td><code>startedOn</code>
<td>string (<a href="https://github.com/in-toto/attestation/blob/7aefca35a0f74a6e0cb397a8c4a76558f54de571/spec/v1/field_types.md#timestamp">Timestamp</a>)<td>

The timestamp of when the build started.

<tr id="finishedOn"><td><code>finishedOn</code>
<td>string (<a href="https://github.com/in-toto/attestation/blob/7aefca35a0f74a6e0cb397a8c4a76558f54de571/spec/v1/field_types.md#timestamp">Timestamp</a>)<td>

The timestamp of when the build completed.

</table>

### Extension fields

[Extension fields]: #extension-fields

Implementations MAY add extension fields to any JSON object to describe
information that is not captured in a standard field. Guidelines:

-   Extension fields SHOULD use names of the form `<vendor>_<fieldname>`, e.g.
    `examplebuilder_isCodeReviewed`. This practice avoids field name collisions
    by namespacing each vendor. Non-extension field names never contain an
    underscore.
-   Extension fields MUST NOT alter the meaning of any other field. In other
    words, an attestation with an absent extension field MUST be interpreted
    identically to an attestation with an unrecognized (and thus ignored)
    extension field.
-   Extension fields SHOULD follow the [monotonic principle][parsing rules],
    meaning that deleting or ignoring the extension SHOULD NOT turn a DENY
    decision into an ALLOW.

## Verification

[Verification]: ../../build/v1.0/verifying-artifacts

Please see [Verifying Artifacts][Verification] for a detailed discussion of
provenance verification.

## Index of build types

The following is a partial index of build type definitions. Each contains a
complete example predicate.

<!-- Sort alphabetically -->

-   [GitHub Actions Workflow (community-maintained)](https://slsa-framework.github.io/github-actions-buildtypes/workflow/v1)
-   [Google Cloud Build (community-maintained)](https://slsa-framework.github.io/gcb-buildtypes/triggered-build/v1)

To add an entry here, please send a pull request on GitHub.

## Migrating from 0.2

To migrate from [version 0.2](/provenance/v0.2) (`old`), use the following
pseudocode. The meaning of each field is unchanged unless otherwise noted.

```javascript
{
    "buildDefinition": {
        // The `buildType` MUST be updated for v1.0 to describe how to
        // interpret `inputArtifacts`.
        "buildType": /* updated version of */ old.buildType,
        "externalParameters":
            old.invocation.parameters + {
            // It is RECOMMENDED to rename "entryPoint" to something more
            // descriptive.
            "entryPoint": old.invocation.configSource.entryPoint,
            // It is OPTIONAL to rename "source" to something more descriptive,
            // especially if "source" is ambiguous or confusing.
            "source": old.invocation.configSource.uri,
        },
        "internalParameters": old.invocation.environment,
        "resolvedDependencies":
            old.materials + [
            {
                "uri": old.invocation.configSource.uri,
                "digest": old.invocation.configSource.digest,
            }
        ]
    },
    "runDetails": {
        "builder": {
            "id": old.builder.id,
            "builderDependencies": null,  // not in v0.2
            "version": null,  // not in v0.2
        },
        "metadata": {
            "invocationId": old.metadata.buildInvocationId,
            "startedOn": old.metadata.buildStartedOn,
            "finishedOn": old.metadata.buildFinishedOn,
        },
        "byproducts": null,  // not in v0.2
    },
}
```

The following fields from v0.2 are no longer present in v1.0:

-   `entryPoint`: Use `externalParameters[<name>]` instead.
-   `buildConfig`: No longer inlined into the provenance. Instead, either:
    -   If the configuration is a top-level input, record its digest in
        `externalParameters["config"]`.
    -   Else if there is a known use case for knowing the exact resolved
        build configuration, record its digest in `byproducts`. An example use
        case might be someone who wishes to parse the configuration to look for
        bad patterns, such as `curl | bash`.
    -   Else omit it.
-   `metadata.completeness`: Now implicit from `builder.id`.
-   `metadata.reproducible`: Now implicit from `builder.id`.

## Change history

### v1.0

Major refactor to reduce misinterpretation, including a minor change in model.

-   Significantly expanded all documentation.
-   Altered the model slightly to better align with real-world build platforms,
    align with reproducible builds, and make verification easier.
-   Grouped fields into `buildDefinition` vs `runDetails`.
-   Renamed:
    -   `parameters` -> `externalParameters` (slight change in semantics)
    -   `environment` -> `internalParameters` (slight change in semantics)
    -   `materials` -> `resolvedDependencies` (slight change in semantics)
    -   `buildInvocationId` -> `invocationId`
    -   `buildStartedOn` -> `startedOn`
    -   `buildFinishedOn` -> `finishedOn`
-   Removed:
    -   `configSource`: No longer special-cased. Now represented as
        `externalParameters`  + `resolvedDependencies`.
    -   `buildConfig`: No longer inlined into the provenance. Can be replaced
        with a reference in `externalParameters` or `byproducts`, depending on
        the semantics, or omitted if not needed.
    -   `completeness` and `reproducible`: Now implied by `builder.id`.
-   Added:
    -   ResourceDescriptor:  `annotations`, `content`, `downloadLocation`,
        `mediaType`, `name`
    -   Builder: `builderDependencies` and `version`
    -   `byproducts`
-   Changed naming convention for extension fields.

Differences from RC1 and RC2:

-   Renamed `systemParameters` (RC1 + RC2) -> `internalParameters` (final).
-   Changed naming convention for extension fields (in RC2).
-   Renamed `localName` (RC1) -> `name` (RC2).
-   Added `annotations` and `content` (in RC2).

### v0.2

Refactored to aid clarity and added `buildConfig`. The model is unchanged.

-   Replaced `definedInMaterial` and `entryPoint` with `configSource`.
-   Renamed `recipe` to `invocation`.
-   Moved `invocation.type` to top-level `buildType`.
-   Renamed `arguments` to `parameters`.
-   Added `buildConfig`, which can be used as an alternative to `configSource`
    to validate the configuration.

### rename: slsa.dev/provenance

Renamed to "slsa.dev/provenance".

### v0.1.1

-   Added `metadata.buildInvocationId`.

### v0.1

Initial version, named "in-toto.io/Provenance"

[Statement]: https://github.com/in-toto/attestation/blob/7aefca35a0f74a6e0cb397a8c4a76558f54de571/spec/v1/statement.md
[in-toto attestation]: https://github.com/in-toto/attestation
[parsing rules]: https://github.com/in-toto/attestation/blob/7aefca35a0f74a6e0cb397a8c4a76558f54de571/spec/v1/README.md#parsing-rules
[purl]: https://github.com/package-url/purl-spec
[threats]: threats
[trusted]: principles#trust-systems-verify-artifacts
[provenance requirements]: ../../build/v1.0/requirements#provenance-generation
