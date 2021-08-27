---
layout: use-case
---

# Use cases

## Consuming third party software

A developer (e.g. FooCo) using third party software (e.g. HowDo developed by HowInc) wants to ensure it hasn’t been tampered with before using it. They could take these steps to protect the organization from the software they consume without having to rely on any trust in intermediate package repositories.

| Subject   | Description                   |
|:----------|:------------------------------|
| **Users** | Software consumers            |
| **Goals** | Protecting against tampering  |

### How to do it

-   Requesting HowInc publish the [in-toto SLSA Provenance](https://slsa.dev/provenance) and any additional attestations (such as [source control attestations](https://github.com/in-toto/attestation/issues/47)) for HowDo each time it’s released
-   Requesting HowInc publish the public keys its builder uses to sign the attestations
-   Requesting HowInc confirm what SLSA level their builder and source control system meet
-   Determining what policy to apply to HowDo
    -   This policy could be created on first use based on the data in the in-toto SLSA Provenance. Any significant deviations (e.g. the builder or source repository changed) would cause failure
    -   Or HowInc could publish a suggested policy for HowDo users on their website
-   Establishing a secure control-point that any HowDo versions/artifacts must pass through to be used
    -   e.g. on import to a local Docker registry
-   Having the control-point check the candidate HowDo and its provenance (as well as any other published attestations) against the policy mentioned above, ensuring any attestations evaluated by the policy are signed by HowInc's public key(s)
-   Only importing the software artifact if all the above checks pass

### Limitations

This use case relies on FooCo trusting the SLSA information that HowInc provides. In the future, there may be accreditation bodies that confirm this information.

The community is debating [how to distribute public keys used to sign attestations](https://github.com/slsa-framework/slsa/issues/101).
