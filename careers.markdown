---
layout: company
title: "Delendum - Team"
description: "We support inventions in blockchain infrastructure, private computing, and zero-knowledge proof applications"
collection: careers
---

<ul class="no-list-style">
{% for post in site.careers %}
    <li class="no-list-style post-container">
        <div class="text-black text-large">
            <a class="text-black" href="{{ post.url }}">
                {{ post.title }}
            </a>
        </div>
        <div class="text-black">
            {{post.location}} 
        </div>
    </li>
{% endfor %}
</ul>
