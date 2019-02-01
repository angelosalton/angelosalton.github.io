---
title: "Ferramentas de produtividade"
layout: page
---

Este é uma seleção de aplicativos de produtividade para o desktop, com exemplos de uso. São em geral programas de terminal Linux, mas que podem ser usados no Windows sem problemas, seja por versões nativas ou com o [Windows Subsystem for Linux](http://wsl-guide.org/en/latest/).

[**pandoc**](pandoc.org) - Conversor de arquivos de texto para uma série de formatos, como Word, LaTeX, Markdown, HTML.

```bash
$ pandoc -f latex -t docx arquivo.tex
```

[**pdfgrep**](https://sourceforge.net/projects/pdfgrep/) - Programa de terminal para procurar por sentenças em arquivos PDF com texto embutido.

```bash
$ pdfgrep *.pdf "palavra"
```

[**pdfsandwich**](http://www.tobias-elze.de/pdfsandwich/) - Esse programa serve para reconhecer texto em arquivos PDF, fazendo automaticamente a conversão. Possui suporte para uma série de idiomas, inclusive o português.

```bash
$ pdfsandwich arquivo.pdf
```

[**convert**](https://imagemagick.org/script/convert.php) - É um script do pacote ImageMagick, que serve para converter e tratar arquivos de imagens.

```bash
$ convert foto.jpg foto.png
```

[**Funções ZZ**](funcoeszz.net) - Funções no terminal para uma infinidade de tarefas cotidianas (manipulações simples de números e texto, consultas e informações de todo tipo).

```bash
$ zzporcento 1.749,00-10%
```