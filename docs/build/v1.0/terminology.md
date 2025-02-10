---
title: Terminology
description: Before diving into the SLSA Build track specification, we need to establish a core set of terminology and models to describe what we're protecting.
layout: specifications
---
<!-- Note on updating docs:
* Using terms such as "developer," "maintainer," "producer," "author," and
  "publisher" interchangeably can cause confusion.
  *  For consistency: Whenever possible, default to "producer," in line with the
     model of producer--consumer--infrastructure provider. "Maintainer" is reserved
     for sections specifying the act of continuing to maintain a project after its
     creation, or when used in a less technical context where it is unlikely to cause
     confusion. Author is reserved for the act of making source code commits or
     reviews. Individual is used when the context's focus is specifying a single
     person (i.e., "an individual's workstation" or "compromised individual").
* Using terms such as "platform," "system," and "service" interchangeably can cause
  confusion.
  * For consistency: Whenever possible, default to "platform." Instead of using "service,"
    a reference to a "hosted platform" should be used. A reference to some specific
    software or tools internal to a platform can be made with "platform component" unless
    there is a more appropriate definition to use directly like "control plane." External
    self-described services and systems can continue to be called by these terms.
-->

Before diving into the [Build track levels](levels.md), we need to establish a
core set of terminology and models to describe what we're protecting. This
extends the [general terminology].

## Software supply chain

### Build model

<p align="center"><img src="images/build-model.svg" alt="Model Build"></p>

We model a build as running on a multi-tenant *build platform*, where each
execution is independent.

1.  A tenant invokes the build by specifying *external parameters* through an
    *interface*, either directly or via some trigger. Usually, at least one of
    these external parameters is a reference to a *dependency*. (External
    parameters are literal values while dependencies are artifacts.)
2.  The build platform's *control plane* interprets these external parameters,
    fetches an initial set of dependencies, initializes a *build environment*,
    and then starts the execution within that environment.
3.  The build then performs arbitrary steps, which might include fetching
    additional dependencies, and then produces one or more *output* artifacts.
    The steps within the build environment are under the tenant's control.
    The build platform isolates build environments from one another to some
    degree (which is measured by the SLSA Build Level).
4.  Finally, for SLSA Build L2+, the control plane outputs *provenance*
    describing this whole process.

Notably, there is no formal notion of "source" in the build model, just external
parameters and dependencies. Most build platforms have an explicit "source"
artifact to build from, which is often a git repository; in the build model, the
reference to this artifact is an external parameter while the artifact itself is
a dependency.

For examples of how this model applies to real-world build platforms, see [index
of build types](/provenance/v1#index-of-build-types).

| Primary Term | Description
| --- | ---
| <span id="platform">Platform</span> | System that allows tenants to run builds. Technically, it is the transitive closure of software and services that must be trusted to faithfully execute the build. It includes software, hardware, people, and organizations.
| Admin | A privileged user with administrative access to the platform, potentially allowing them to tamper with builds or the control plane.
| Tenant | An untrusted user that builds an artifact on the platform. The tenant defines the build steps and external parameters.
| Control plane | Build platform component that orchestrates each independent build execution and produces provenance. The control plane is managed by an admin and trusted to be outside the tenant's control.
| Build | Process that converts input sources and dependencies into output artifacts, defined by the tenant and executed within a single build environment on a platform.
| Steps | The set of actions that comprise a build, defined by the tenant.
| <span id="build-environment">Build environment</span> | The independent execution context in which the build runs, initialized by the control plane. In the case of a distributed build, this is the collection of all such machines/containers/VMs that run steps.
| Build caches | An intermediate artifact storage managed by the platform that maps intermediate artifacts to their explicit inputs. A build may share build caches with any subsequent build running on the platform.
| External parameters | The set of top-level, independent inputs to the build, specified by a tenant and used by the control plane to initialize the build.
| Dependencies | Artifacts fetched during initialization or execution of the build process, such as configuration files, source artifacts, or build tools.
| Outputs | Collection of artifacts produced by the build.
| <span id="provenance">Provenance</span> | Attestation (metadata) describing how the outputs were produced, including identification of the platform and external parameters.

<details><summary>Ambiguous terms to avoid</summary>

-   *Build recipe:* Could mean *external parameters,* but may include concrete
    steps of how to perform a build. To avoid implementation details, we don't
    define this term, but always use "external parameters" which is the
    interface to a build platform. Similar terms are *build configuration
    source* and *build definition*.
-   *Builder:* Usually means *build platform*, but might be used for *build
    environment*, the user who invoked the build, or a build tool from
    *dependencies*. To avoid confusion, we always use "build platform". The only
    exception is in the [provenance](/provenance/v1), where `builder` is used as
    a more concise field name.

</details>

### Build environment model

<p align="center"><img src="images/build-env-model.svg" alt="Build Environment Model"></p>

The Build Environment (BuildEnv) track expands upon the
[build model](#build-model) by explicitily separating the
[build image](#build-image) and [compute platform](#compute-platform) from the
abstract [build environment](#build-environment) and [build platform](#platform).
Specifically, the BuildEnv track defines the following roles, components, and concepts:

| Primary Term | Description
| --- | ---
| <span id="build-id">Build ID</span> | An immutable identifier assigned uniquely to a specific execution of a tenant's build. In practice, the build ID may be an identifier, such as a UUID, associated with the build execution.
| <span id="build-image">Build image</span> | The template for a build environment, such as a VM or container image. Individual components of a build image include the root filesystem, pre-installed guest OS and packages, the build executor, and the build agent.
| <span id="build-image-producer">Build image producer</span> | The party that creates and distributes build images. In practice, the build image producer may be the hosted build platform or a third party in a bring-your-own (BYO) build image setting.
| <span id="build-agent">Build agent</span> | A build platform-provided program that interfaces with the build platform's control plane from within a running build environment. The build agent is also responsible for executing the tenant’s build definition, i.e., running the build. In practice, the build agent may be loaded into the build environment after instantiation, and may consist of multiple components. All build agent components must be measured along with the build image.
| <span id="build-dispatch">Build dispatch</span> | The process of assigning a tenant's build to a pre-deployed build environment on a hosted build platform.
| <span id="compute-platform">Compute platform</span> | The compute system and infrastructure underlying a build platform, i.e., the host system (hypervisor and/or OS) and hardware. In practice, the compute platform and the build platform may be managed by the same or distinct organizations.
| <span id="host-interface">Host interface</span> | The component in the compute platform that the hosted build platform uses to request resources for deploying new build environments, i.e., the VMM/hypervisor or container orchestrator.
| <span id="boot-process">Boot process</span> | In the context of builds, the process of loading and executing the layers of firmware and/or software needed to start up a build environment on the host compute platform.
| <span id="measurement">Measurement</span> | The cryptographic hash of some component or system state in the build environment, including software binaries, configuration, or initialized run-time data.
| <span id="quote">Quote</span> | (Virtual) hardware-signed data that contains one or more (virtual) hardware-generated measurements. Quotes may additionally include nonces for replay protection, firmware information, or other platform metadata. (Based on the definition in [section 9.5.3.1](https://trustedcomputinggroup.org/wp-content/uploads/TPM-2.0-1.83-Part-1-Architecture.pdf) of the TPM 2.0 spec)
| <span id="reference-value">Reference value</span> | A specific measurement used as the good known value for a given build environment component or state.

**TODO:** Disambiguate similar terms (e.g., image, build job, build executor/runner)

#### Build environment lifecycle

A typical build environment will go through the following lifecycle:

1.  *Build image creation*: A [build image producer](#build-image-producer)
    creates different [build images](#build-image) through a dedicated build
 process. For the SLSA BuildEnv track, the build image producer outputs
 [provenance](#provenance) describing this process.
2.  *Build environment instantiation*: The [hosted build platform](#platform)
    calls into the [host interface](#host-interface) to create a new instance
 of a build environment from a given build image. The
 [build agent](#build-agent) begins to wait for an incoming
 [build dispatch](#build-dispatch).
 For the SLSA BuildEnv track, the host interface in the compute platform
 attests to the integrity of the environment's initial state during its
 [boot process](#boot-process).
3.  *Build dispatch*: When the tenant dispatches a new build, the hosted
    build platform assigns the build to a created build environment.
    For the SLSA BuildEnv track, the build platform attests to the binding
 between a build environment and [build ID](#build-id).
4.  *Build execution*: Finally, the build agent within the environment executes
    the tenant's build definition.

### Verification model

Verification in SLSA is performed in two ways. Firstly, the build platform is
certified to ensure conformance with the requirements at the level claimed by
the build platform. This certification should happen on a recurring cadence with
the outcomes published by the platform operator for their users to review and
make informed decisions about which builders to trust.

Secondly, artifacts are verified to ensure they meet the producer defined
expectations of where the package source code was retrieved from and on what
build platform the package was built.

![Verification Model](images/verification-model.svg)

| Term         | Description
|--------------|----
| Expectations | A set of constraints on the package's provenance metadata. The package producer sets expectations for a package, whether explicitly or implicitly.
| Provenance verification | Artifacts are verified by the package ecosystem to ensure that the package's expectations are met before the package is used.
| Build platform certification | [Build platforms are certified](verifying-systems.md) for their conformance to the SLSA requirements at the stated level.

The examples below suggest some ways that expectations and verification may be
implemented for different, broadly defined, package ecosystems.

<details><summary>Example: Small software team</summary>

| Term | Example
| ---- | -------
| Expectations | Defined by the producer's security personnel and stored in a database.
| Provenance verification | Performed automatically on cluster nodes before execution by querying the expectations database.
| Build platform certification | The build platform implementer follows secure design and development best practices, does annual penetration testing exercises, and self-certifies their conformance to SLSA requirements.

</details>

<details><summary>Example: Open source language distribution</summary>

| Term | Example
| ---- | -------
| Expectations | Defined separately for each package and stored in the package registry.
| Provenance verification | The language distribution registry verifies newly uploaded packages meet expectations before publishing them. Further, the package manager client also verifies expectations prior to installing packages.
| Build platform certification | Performed by the language ecosystem packaging authority.

</details>

<!-- Links definition -->

[general terminology]: ../../spec/v1.0/terminology
