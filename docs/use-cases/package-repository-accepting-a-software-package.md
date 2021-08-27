---
layout: use-case
---

# Use cases

## Package Repository accepting a software package

A package repository (e.g. Docker Hub) wants to protect their users from malicious changes to the container images uploaded to the repository. They could do this by the following steps, which could protect users of protected repositories from malicious tampering without requiring all users to perform their own policy checks on each image they consume.

| Subject   | Description                                                 |
|:----------|:------------------------------------------------------------|
| **Users** | Package managers                                            |
| **Goals** | Protecting users from malicious changes                     |
|           | Not requiring users to perform policy checks on every image |

### How to do it

-   Requesting publishers of artifacts publish the [in-toto SLSA provenance](https://slsa.dev/provenance) and any additional attestations (e.g. [source control attestations](https://github.com/in-toto/attestation/issues/47)) each time a new image is pushed to the repository
-   Requesting publishers publish the public keys a builder uses to sign the attestations
-   Requesting publishers confirm what SLSA level their builder and source control system meet
-   Determining what policy to apply to published artifacts
    -   This policy could be created on first use based on the data in the in-toto SLSA provenance. Any significant deviations (e.g. the builder or source repository changed) would cause a push failure
    -   Or the package repository could have publishers configure their specific policy as part of their repository, and could make these policies publicly readable to its users
-   Checking new artifacts and their provenance (as well as any other attestations) against the policy above, ensuring any attestations evaluated by the policy are signed by the publisher's public key
-   Preventing artifacts that fail the above check from being made public

### Limitations

This use case relies on the package repository trusting the SLSA information that publishers provide. In the future, there may be accreditation bodies that confirm this information.

The community is debating [how to distribute public keys used to sign attestations](https://github.com/slsa-framework/slsa/issues/101).
