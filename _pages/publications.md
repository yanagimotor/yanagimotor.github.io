---
layout: archive
title: "Publications/Preprints"
permalink: /publications/
author_profile: true
---
{% include base_path %}
First-authored Papers
---
{% for post in site.publications reversed %}
{% if post.featured %}
  {% include archive-single.html %}
{%endif%}
{% endfor %}

Co-authored Papers
---
{% for post in site.publications reversed %}
{% if (post.featured) %}
{% else %}
{% include archive-single.html %}
{%endif%}
{% endfor %}