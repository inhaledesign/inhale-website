---
layout: main
sidebar_link: true
sidebar_sort_order: 4
title: Articles
---

{% assign posts = site.posts | where_exp: "series", "series != nil" %}

{% if posts.size > 0 %}

  {% assign seriesSet = "" | split: "," %}

  {% for post in posts %}
    {% assign seriesSet = seriesSet | push: post.series | uniq %}
  {% endfor %}

  {% for currentSeries in seriesSet %}

## {{ currentSeries }}

    {% assign seriesPosts = posts | where: "series", currentSeries | sort: "series_order" %}
    {% for post in seriesPosts %}
* [{{ post.title }}]({{ post.url }}){% if post.description %} -- {{ post.description }}{% endif %}
    {% endfor %}

  {% endfor %}

{% endif %}