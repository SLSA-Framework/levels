---
title: How to SLSA for implementers
description: If you're looking to jump straight in and try SLSA, here's a quick start guide for the steps to take to reach the different SLSA levels.
layout: standard
---

This is a quick-start guide for infrastructure providers such as build platforms
or package registries that want to support SLSA for their users. The work
involved in supporting SLSA differs depending on the sort of infrastructure or
tooling you provide.

## Types of infrastructure

### Build platform

1.  Verify that your infrastructure is suitable to produce SLSA provenance. To
learn more about verifying your system for SLSA conformance, see
[Verifying build platforms](/spec/v1.0/verifying-systems).
2.  Add support for generating SLSA provenance. To learn more about producing
provenance, see [Producing artifacts](/spec/v1.0/requirements). To learn more
about the SLSA provenance format, see [Provenance](/provenance/v1).

### [Package registry](/spec/v1.0/terminology.md#package-model)

1.  Verify provenance for the software you distribute. To
learn more about verifying provenance, see
[Verifying artifacts](/spec/v1.0/verifying-artifacts).
2.  Distribute provenance for the software you distribute. To learn more about
distributing provenance, see
[Distributing provenance](/spec/v1.0/distributing-provenance).

### Compiler or other CLI build tool

1.  Do nothing. While your tool can produce SLSA provenance, it will never be
able to reach Build levels beyond Build Level 1. Instead, encourage your users
to produce SLSA provenance in their build platform.
