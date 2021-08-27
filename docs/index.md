---
title: Overview
---
# Improving artifact integrity across the supply chain

<!--{% if false %}-->

**NOTE: This site is best viewed at https://slsa.dev.**

<!--{% endif %}-->

<span class="subtitle">

SLSA ("[salsa](https://www.google.com/search?q=how+to+pronounce+salsa)") is **Supply-chain Levels for Software Artifacts.**

A security framework from source to service, giving anyone working with software a common language for increasing levels of software security and supply chain integrity.

</span>

<!-- Levels overview -->
<section class="breakout">

<div class="wrapper">
<span class="subtitle flushed">Overview</span>

## Security levels

Each level provides requirements, processes and best practices to increase trust in software. These look at the integrity of the source and build services, available information about the code, reproducibility and resilience against tampering or human error.

<div class="level-icons m-b-l m-t-xl">

<div class="level">

<div class="level-badge">

![Level 1](images/levelBadge1.svg)

</div>

### Basic protection

Provenance checks to help evaluate risks and security

</div>

<div class="level">

<div class="level-badge">

![Level 2](images/levelBadge2.svg)

</div>

### Medium protection

Further checks against the origin of the software

</div>

<div class="level">

<div class="level-badge">

![Level 3](images/levelBadge3.svg)

</div>

### Advanced protection

Extra resistance to specific classes of threats

</div>
<div class="level">

<div class="level-badge">

![Level 4](images/levelBadge4.svg)

</div>

### Maximum protection

Strict auditability and reliability checks

</div>

</div>

<div class="buttons-horizontal">

<div class="pseudo-button">

[Learn more](levels.md)

</div>

</div>

</div>

</section>

</section>

<!-- Supply chain diagram -->
<section class="content-block">
<span class="subtitle flushed">The supply chain</span>

## Protecting each stage of development

<div class="m-b-l">

### How do you mitigate threats and risks?

Any software can introduce vulnerabilities into a supply chain, with recent high profile cases proving how costly an attack can be. The steps that make up the SLSA framework aim to empower developers and software consumers to easily and automatically check the integrity of software artifacts, developed in direct response to [known supply chain attacks](levels.md#threats).

</div>

<!-- System threats diagram -->
<div class="diagram-wrapper">

<div class="diagram">

![Supply Chain Threats](images/supply-chain-threats.svg)

</div>

<div class="annotation m-t-s">
Where threats and risks occur in a supply chain
</div>

</div>

<div class="m-t-xl">

### Standard security guidelines that scale

SLSA levels are a way to better understand your current security posture, protect yourself from potential threats and plan for the future. If you’re a software consumer, you can check that the security information for any software in your supply chain is accurate, whether it provides the exact level of security you need, and help develop, share and promote tools that automate the process.

<div class="pseudo-button m-t-l">

[Read the requirements](requirements.md)

</div>

</div>

</section>
<!-- Future -->
<section class="breakout">

<div class="wrapper">
<span class="subtitle flushed">Ethos</span>

## Building towards the future

<span class="subtitle">
Today’s projects, products and services are increasingly complex and open to attack. As that trend continues, we need to scale up our effort to provide more secure, accessible ways to protect the development, distribution and consumption of the software we use, and all the impacted communities behind it.
</span>

<div class="pseudo-button m-t-l">

[Get involved](getinvolved.md)

</div>

</div>

</section>

<!-- Two column wrap-up -->
<section class="col-2 content-block">
<span>

## Currently in alpha

The framework is constantly being improved, and is now ready to be tried out and tested. Google has been using an internal version of SLSA since 2013 and requires it for all of their production workloads.

<div class="pseudo-button m-t-l">

[See our roadmap](roadmap.md)

</div>
</span>

<span>

## Get involved

We rely on feedback from other organizations to improve, and we’d love to hear from you. Are the levels achievable in your project? Would you add or remove anything from the framework?

<div class="pseudo-button m-t-l">

[Join the conversation](getinvolved.md)

</div>

</span>
</section>

<!-- Future -->
<section class="breakout">

<div class="wrapper">
<span class="subtitle flushed">Get started</span>

## See a GitHub Actions demo

Check our demonstration for SLSA level 1 with [a provenance generator for GitHub Actions](https://github.com/slsa-framework/github-actions-demo).

</div>

</section>
