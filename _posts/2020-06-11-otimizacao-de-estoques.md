---
title: Uma formulação matemática simples para otimização de estoques
tags:
  - Python
  - Optimization
---

Motivado por um desafio interno na [SLC Agrícola](https://www.slcagricola.com.br/), resolvi pesquisar mais a fundo sobre otimização de estoques, especificamente em estratégias que diminuem desequilíbrios de oferta e demanda de um determinado item em várias localidades/armazéns. Acontece que este é um dos objetos de estudo da [pesquisa operacional](https://pt.wikipedia.org/wiki/Investiga%C3%A7%C3%A3o_operacional), que é uma área do conhecimento que busca aplicar soluções matemáticas e estatísticas à tomada de decisão em geral.

# Estratégia

Podemos formular esse problema como sendo de programação linear, mais precisamente de [fluxo de commodities](https://en.wikipedia.org/wiki/Multi-commodity_flow_problem), se tratarmos as quantidades planejadas e o estoque lógico, respectivamente, como demandas e ofertas de um determinado produto nas diferentes localidades. Como não há restrições de fluxo total de produtos entre as unidades, podemos resolver o problema separadamente para vários produtos que possam existir.

Definindo os seguintes vetores e matrizes:

$$f \in F = \{1,2,\dots,F,\textrm{resto}\}$$: localidades

$$c_{f,g}$$: custo da transação da localidade $$f$$ para $$g$$

$$b_{f}$$: oferta líquida do item na localidade $$f$$ (pode ser negativa)

$$x_{f,g}$$: fluxo do item da localidade $$f$$ para $$g$$ (as variáveis de decisão)

O objetivo do problema será minimizar o custo total das transferências entre localidades, atendendo os desequilíbrios entre oferta e demanda de itens da melhor maneira possível:

$$
\min_{x_{f,g}} \sum_{f,g \in F} c_{f,g} x_{f,g} \\
\textrm{s.a.} \\
\begin{align}
\sum_{f \neq g} (x_{f,g}-x_{g,f}) &\leq b_{f}~(1) \\
x_{f,g} &\geq 0~(2) \\
x_{\textrm{resto},f} &= 0~(3)
\end{align}
$$

O significado das restrições:

1. O saldo das transações deve ser menor ou igual ao excesso de oferta do item na localidade $$f$$;

2. As transferências devem ser não-negativas;

3. Não deve haver transferências do "resto" de/para outras localidades. O *resto* (ou *sink*) é uma localidade teórica que compensa o excesso de demanda/oferta total do sistema.

## Representação gráfica

É possível representar as localidades e suas capacidades como [grafos](https://pt.wikipedia.org/wiki/Teoria_dos_grafos) direcionados. Assim, o problema acima se torna um problema de fluxo ótimo entre os nós do grafo.

![img](https://upload.wikimedia.org/wikipedia/commons/thumb/a/ae/Network_flow_residual_SVG.svg/332px-Network_flow_residual_SVG.svg.png)

Se as localidades de interesse são apenas um subconjunto de uma rede com um grande número de localidades, é possível simplificar o problema criando uma sub-rede cujas ligações representam o caminho mais curto (ou custo mínimo) de deslocamento entre essas localidades.

# Na prática

Desenvolvi um *dashboard* em Python que aplica estes conceitos em um cenário urbano, usando dados simulados. O código está no link:

[Repositório no GitHub](https://github.com/angelosalton/dash-inventory-flow).