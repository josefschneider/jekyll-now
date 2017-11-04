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

{% if page.comments %}
<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/

var disqus_config = function () {
this.page.url = "https://josefschneider.github.io/";  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = "CLaSH index"; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};

(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://https-josefschneider-github-io.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
{% endif %}
