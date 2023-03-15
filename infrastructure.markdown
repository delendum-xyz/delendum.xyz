---
layout: product
title: "Delendum - Infrastructure"
description: "We support inventions in blockchain infrastructure, private computing, and zero-knowledge proof applications"
tags: infrastructure
---

<p class="text-black text-research-para">
    These projects will be open source MIT-licensed. Please check back in April for
implementation details.
</p>
<ul class="no-list-style">
{% for post in site.infrastructure %}
    <li class="no-list-style post-container">
        <div class="text-black text-large">
            <a class="text-black" href="{{ post.url }}">
                {{ post.title }}
            </a>
        </div>
        <div class="text-black">
            {{post.author}} 
        </div> 
        {{ post.excerpt | strip_html | strip_newlines | truncate: 200 }}  
    </li>
{% endfor %}
</ul>
