---
layout: landing
---

<div class="row" style="margin-bottom:20px;">
  
  <div class="divbox" style="width:96%;margin-right:0px;padding-right:10px;">
    <h1 id="start-now" style="margin-left: 0px; margin-right: 0px; font-size: 22px;">最新博文</h1>
    <div style="margin-left:-15px">
    {% assign posts_all = site.posts %}
    {% assign count = 5 %}
    {% include custom/posts_all %}
  </div>
  </div>
  
</div>