---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
hero_image: /assets/images/bg.jpg
---

# Bem-vindo!

Esta é a minha página pessoal. Na barra superior você pode encontrar outras informações de contato. Abaixo, links das
minhas redes.

<a class="button is-light"
  href="https://www.dropbox.com/s/z1vlhud8dggaw89/Curriculum%20Vitae%20Angelo%20Salton.pdf?dl=0"><i
    class="fas fa-user-tie"></i>&nbsp; Curriculum Vitae</a>
<a class="button is-light" href="https://www.linkedin.com/in/angelo-salton"><i class="fab fa-linkedin"></i>&nbsp;
  LinkedIn</a>
<a class="button is-light" href="https://instagram.com/salton.a"><i class="fab fa-instagram"></i>&nbsp; Instagram</a>
<a class="button is-light" href="https://github.com/angelosalton/"><i class="fab fa-github"></i>&nbsp; GitHub</a>
<a class="button is-light" href="https://soundcloud.com/salton1"><i class="fab fa-soundcloud"></i>&nbsp; Soundcloud</a>

# Últimos posts

{% for post in site.posts limit:3 %}
<div class="box">
  <a href="{{ post.url }}">
    <h2 class="subtitle">{{ post.title }}</h2>
  </a>
  <i>{{ post.date | date_to_string }}</i>
  <p>{{ post.excerpt }}</p>
</div>
{% endfor %}

<a class="button is-primary" href="/posts.html">Mais posts</a>