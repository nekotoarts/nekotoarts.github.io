---
layout: archive
title: "Sample Code Files"
permalink: /code_samples/
author_profile: true
---

Here's a compiled list of useful code snippets. I mostly use this to share code that I use very often or I need to share with people very often.

<ul>
{% for code in site.data.codes %}
    <li><a href="{{ code.filename | datapage_url: '/code_samples' }}">{{code.header}}</a></li>
{% endfor %}
</ul>
