---
title: Exercícios de linguagem de IA do Azure
permalink: index.html
layout: home
---

# Exercícios de linguagem de IA do Azure

Os exercícios a seguir foram planejados para dar suporte aos módulos no Microsoft Learn para [Desenvolvimento de soluções de linguagem natural](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/).


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %} {% unless activity.url contains 'ai-foundry' %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endunless %} {% endfor %}
