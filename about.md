---
layout: page
title: About Me
permalink: /about/
published: true
---

<div class="page" markdown="1">

{% capture page_subtitle %}
<img
    class="me"
    alt="{{ author.name }}"
    src="{{ site.author.photo | relative_url }}"
    srcset="{{ site.author.photo2x | relative_url }} 2x"
/>
{% endcapture %}

{% include page/title.html title=page.title subtitle=page_subtitle %}

## Profile

一个毕业于武汉一所双非大学的普通人

一个对编程人菜瘾大的小菜鸟

一个不甘平凡的平凡人

## Why be a programmer

小时候看过哈利波特的电影,魔法世界就印到了我的脑中.现在也一直在想,为什么还没收到霍格沃兹是入学通知书.
言归正传,一切只是兴趣使然.在魔法世界中,魔法无所不能,可以改变一切.而编程不就是现实世界的魔法吗?

</div>
