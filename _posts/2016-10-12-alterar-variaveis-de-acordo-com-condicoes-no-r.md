---
author: Angelo Salton
title: Alterar variáveis de acordo com condições no R
tags:
  - R
comments: true
---

Para gerar ou alterar condicionalmente valores de variáveis no R, comumente usamos a notação de linhas e colunas de um `data.frame`:

```r
dados[condicao == 1, var1] <- 1
```

onde `var1` pode ser uma ou um vetor de variáveis. Entretanto, as funções base do R não possuem uma sintaxe simples para alterar valores de variáveis condicionalmente, avaliando o comando por grupos. Para exemplificar o problema, esta é a operação equivalente no software Stata:

```stata
by id, sort: replace var2 = 1 if condicao == 1
```

Entra em ação o pacote `data.table`, que possui várias funcionalidades para manipulação de dados. Primeiro, é preciso instalar e carregar o pacote e declarar a base de dados como um `data.table`, e depois aplicar o comando:

```r
install.packages("data.table")
library(data.table)

dados <- data.table(dados)

dados[condicao == 1, var2 := 1, by=id]
```

Note o uso do operador `:=`. Ele preserva a estrutura atual dos dados ao gerar a nova variável. Outra opção é usar `=`, que transforma a base e entrega apenas um resultado calculado para cada elemento de `id`. É possível usar várias variáveis para identificar grupos (ex.: `by=.(id1,id2)`).
