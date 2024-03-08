---
title: Azure AI 언어 연습
permalink: index.html
layout: home
---

# Azure AI 언어 연습

다음 연습은 [자연어 솔루션 개발](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/)을 위한 Microsoft Learn 모듈을 지원하도록 설계되었습니다.


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
