---
layout: use-case
---

# Use cases

## Publishing a software package

A developer (e.g. BarInc) wants to protect consumers of their software from malicious changes to the BarImage container image they publish. They want to prevent any negative consequences and damage to their reputation which would occur if that happened. They want to have access to metadata for auditing and ad hoc analysis.

| Subject   | Description                              |
|:----------|:-----------------------------------------|
| **Users** | Developers                               |
| **Goals** | Protecting users from malicious changes  |
|           | Protecting their company’s reputation    |
|           | Access to metadata for auditing/analysis |

### How to do it

BarInc can achieve these goals when publishing the container image by:

-   Upgrading their source control systems to meet higher SLSA levels
-   Upgrading their build system to meet higher SLSA levels
-   Ensuring BarImage must go through a secure control-point in order to be published
-   Having the control-point check the candidate BarImage against its provenance, checking that – for example:
    -   The expected builder created it
    -   The builder meets a minimum SLSA level
    -   The source repositories listed in the provenance meets a minimum SLSA level
    -   The build entry point listed in the provenance is expected
    -   The binary dependencies listed in the provenance meets a minimum SLSA level
-   Only publishing the container image if all the above checks pass
-   Storing the provenance and all other attestations for future reference

### Limitations

This approach doesn’t protect their users from a published BarImage being tampered with after publication. There may be other ways to address these concerns such as code signing after verification, or encouraging [use of the SLSA framework by their software consumers](/consuming-third-party-software.md).
