---
layout: page
title: About
description: 个人介绍页
keywords: Zhou Xiang, 周详
comments: true
menu: 关于
permalink: /about/
---

我是周详，一个普通而又不普通的程序猿。

信仰「科学」以及「玄学」。

坚信熟能生巧，努力改变人生。

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
