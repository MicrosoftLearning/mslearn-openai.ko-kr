---
title: Azure OpenAI 연습
permalink: index.html
layout: home
---

# Azure OpenAI 연습

다음 연습은 [Microsoft Learn](https://learn.microsoft.com/training/browse/?terms=OpenAI) 모듈을 지원하도록 설계되었습니다.


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
