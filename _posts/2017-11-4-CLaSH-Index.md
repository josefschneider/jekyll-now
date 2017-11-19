---
layout: post
title: CLaSH for Hardware Engineers
---

For the last few months I have been trying to get my head around the CLaSH hardware description language in an attempt to boost my productivity when designing digital circuits for FPGA. My experiences have been mostly positive, but coming from a non functional background (I'm mostly familiar with C++, Python and VHDL) I found getting started very slow and tedious. It also took me a while to figure out what CLaSH can and can't do, and to determine whether it was worth the effort. Here is my take on it, as a CLaSH newbie, together with some info that I would have found useful when I started.

Please note this page is targeted at developers who are already familiar with HDL design. There is also a lot I still don't know, so if you have any suggestions or want to point out mistakes please let me know!

{% assign atopics = site.clash-for-hw-engineers | sort: 'order' %}
{% for atopic in atopics %}
<ul>
  <li {% if page.url == atopic.url %} class="active"{% endif %}>
    <a href="{{ atopic.url }}">{{ atopic.title }}</a>
  </li>
</ul>
{% endfor %}

