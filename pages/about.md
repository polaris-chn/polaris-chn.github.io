---
layout: page
title: About
description: 计算改变世界
keywords: Polaris
comments: true
menu: 关于
permalink: /about/
---

我是Polaris，集成电路专业学生

会一些数字电路前端设计，对计算机专业也感兴趣

前几年欠账太多，现在正在拼命补课

树立坚持学习的信念，养成良好的学习方法，提升自己的学习效率，构建完整的知识体系

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'mazhuang.org' %}
<li>
微信公众号：<br />
<img style="height:192px;width:192px;border:1px solid lightgrey;" src="{{ assets_base_url }}/assets/images/qrcode.jpg" alt="闷骚的程序员" />
</li>
{% endif %}
</ul>


## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
