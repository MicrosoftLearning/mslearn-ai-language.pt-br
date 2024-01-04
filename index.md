---
title: Exercícios de Linguagem de IA do Azure
permalink: index.html
layout: home
---

# Exercícios de Linguagem de IA do Azure

Os exercícios a seguir foram planejados para apoiar aos módulos no Microsoft Learn de [Desenvolvimento de soluções de linguagem natural](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/).


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
