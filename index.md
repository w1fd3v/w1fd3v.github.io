---
layout: default
title: Inicio
---

# 💀 Hacking Portfolio de w1fd3v

Bienvenido a mi bitácora de aprendizaje. Aquí encontrarás mis writeups de Hack The Box y apuntes de Linux.

## 📝 Últimos Writeups

<ul>
  {% for post in site.posts %}
    <li>
      <span style="color: #666;">[{{ post.date | date: "%d/%m/%Y" }}]</span>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>