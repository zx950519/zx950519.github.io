---
layout: page
title: About
description: CS will change the world！
keywords: Zhou Xiang, 周详
comments: true
menu: 关于
permalink: /about/
---

我是周详。

迷信「科学」。

不聪明也不笨。

个人格言：不积跬步 无以至千里。

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
