# SLSA ("salsa") is Supply-chain Levels for Software Artifacts

<img align="right" src="https://github.com/slsa-framework/slsa/blob/main/docs/images/slsa-dancing-goose-logo.svg" alt="The OpenSSF mascot, a goose in armor, strikes a pose wearing a red salsa dress">

SLSA (pronounced ["salsa"](https://www.google.com/search?q=how+to+pronounce+salsa)) is a security framework from source to service, giving anyone working with software a common language for increasing levels of software security and supply chain integrity. It’s how you get from safe enough to being as resilient as possible, at any link in the chain.

## Learning about SLSA

See https://slsa.dev to learn about SLSA.

## What's in this repo?

The primary content of this repo is the [docs/](docs/) directory, which contains
the core SLSA specification and sources to the [slsa.dev] website. See the
README.md in that directory for instructions on how to build the site.

This repository also hosts SLSA's main [issue tracker], covering the website,
specification, and overall project management. Other git repositories within the
[slsa-framework](https://github.com/slsa-framework) organization have
repo-specific issue trackers.

## How to get involved

See https://slsa.dev/community for ways to get involved in SLSA development.

## Active workstreams

| Workstream | [Shepherd]
| ---------- | ----------
| [Build Level 4] | David A Wheeler (@david-a-wheeler)
| [Attested Build Environments Track] | Marcela Melara (@marcelamelara), Pavel Iakovenko (@paveliak)
| [Source Track] | Tom Hennen (@TomHennen)
| [Version 1.1 release] | Arnaud Le Hors (@lehors)

[Shepherd]: CONTRIBUTING.md#workstream-lifecycle
[Build Level 4]: https://github.com/slsa-framework/slsa/issues/977
[Attested Build Environments Track]: https://github.com/orgs/slsa-framework/projects/6
[Source Track]: https://github.com/orgs/slsa-framework/projects/5
[Version 1.1 release]: https://github.com/slsa-framework/slsa/issues/900

## URL Aliases

We have several [redirect](docs/_redirects) configured on slsa.dev for
convenience of the team:

-   https://slsa.dev/gh &rArr; SLSA GitHub repo
    -   https://slsa.dev/gh/issues
    -   https://slsa.dev/gh/pulls
    -   etc...
-   https://slsa.dev/notes &rArr; meeting notes
    -   https://slsa.dev/notes/community
    -   https://slsa.dev/notes/positioning
    -   https://slsa.dev/notes/specification
        (or [.../spec](https://slsa.dev/notes/spec))
    -   https://slsa.dev/notes/tooling

## Governance

SLSA is an [OpenSSF](https://openssf.org) project. See
[slsa-framework/governance](https://github.com/slsa-framework/governance) for
governance information, including current steering committee members.

To include the steering committee on GitHub, use
`@slsa-framework/slsa-steering-committee`.

## License

All SLSA specification content contributed following adoption of the Community
Specification governance model is provided under the
[Community Specification License 1.0](LICENSE.md).

Pre-existing portions of the SLSA specification from contributors who have not
subsequently contributed under the Community Specification License 1.0 following
its adoption are provided under the
[Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0.txt).

<!-- Links -->

[issue tracker]: https://github.com/slsa-framework/slsa/issues
[slsa.dev]: https://slsa.dev
