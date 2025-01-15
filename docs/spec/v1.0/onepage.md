---
title: SLSA Specification
description: A one-page rendering of all that is included in SLSA v1.0.
noindex: true
---
{%- comment -%}
A single page containing all the following files as different sections
{%- endcomment -%}

{% assign dir = "/spec/v1.0/" %}
{% assign filenames = "whats-new,about,threats-overview,use-cases,principles,faq,future-directions,terminology,levels,requirements,distributing-provenance,verifying-artifacts,verifying-systems,threats,attestation-model,provenance,verification_summary" %}

{% include onepage.liquid dir=dir filenames=filenames %}
