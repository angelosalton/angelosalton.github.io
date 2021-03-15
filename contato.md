---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
title: Contato
hero_height: is-small
---

<form action="https://formspree.io/f/mzbknejo" method="POST" enctype="multipart/form-data">
  <div class="field">
    <label class="label">Nome:</label>
    <div class="control">
      <input class="input" type="message" name="/textarea">
    </div>
  </div>

  <div class="field">
    <label class="label">E-mail:</label>
    <div class="control">
      <input class="input" type="email" name="_replyto">
    </div>
  </div>

  <div class="field">
    <label class="label">Mensagem:</label>
    <div class="control">
      <textarea class="textarea" type="message" name="message"></textarea>
    </div>
  </div>

  <button class="button is-primary" type="submit">Enviar</button>
</form>