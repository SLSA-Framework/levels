---
title: Build Track
description: The SLSA Build track specification version 1.0.
layout: specifications
---

{{ page.description }}

SLSA is organized into a series of levels and tracks that provide increasing
supply chain security guarantees on various aspects of the supply chain
security. This specification defines the different security levels of the *SLSA
Build track*. For a general overview see the different [tracks and levels].

{%- for section in site.data.nav.main %}
{%- if section.url == page.url and section.children %}

{{ section.description }}

<!-- markdownlint-capture -->
<!-- markdownlint-disable MD055 MD056 -->
| Page | Description
| ---- | -----------
{%- for child in section.children %}
| [{{child.title}}]({{child.url | relative_url}}) | {{child.description}}
{%- endfor %}
<!-- markdownlint-restore -->

{%- endif %}
{%- endfor %}

<!-- Link definitions -->

[tracks and levels]: ../../spec/draft/levels
