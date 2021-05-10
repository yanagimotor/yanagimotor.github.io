---
layout: archive
title: "Publications/Preprints"
permalink: /publications/
author_profile: true
---
{% include base_path %}
Featured Researches
---
{% for post in site.publications reversed %}
{% if post.featured %}
  {% include archive-single.html %}<br/>
{%endif%}
{% endfor %}

Other researches
---
{% for post in site.publications reversed %}
{% if (post.featured) %}
{% else %}
{% include archive-single.html %}
{%endif%}
{% endfor %}