---
layout: research
title: "Delendum - Writing"
description: "We support inventions in blockchain infrastructure, private computing, and zero-knowledge proof applications"
tags: writing
---

<p class="text-black text-research-para">
    We curated a list of resources below, aiming to provide a knowledge base and to cover developer needs. We will add more sections soon, such as project evaluations. We are also open to advise projects, provide reviews, and co-author research publications. 
</p>
<ul class="no-list-style">
{% for post in site.writings reversed %}
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
