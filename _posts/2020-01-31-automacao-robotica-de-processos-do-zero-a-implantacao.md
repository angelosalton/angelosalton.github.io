---
title: 'Automação robótica de processos: do zero à implantação'
tags:
  - RPA
---

Neste post, vou comentar desafios e experiências pessoais que tive na implantação de uma solução de automação robótica de processos (conhecida também pelo seu acrônimo em inglês, RPA) em uma empresa de grande porte. 

Em várias empresas de maior porte, ou mais tradicionais, existem políticas e processos bem definidos para as atividades centrais. Na minha experiência, dois fatores foram chave para o andamento

# Por onde começar?

O ponto de partida foi identificar, através do contato com analistas de processo e pela convivência com o _workflow_ diário dos colegas, processos com maior potencial de retorno com automação. Evidentemente, processos frequentes, altamente repetitivos e com regras de negócio simples acabam sendo os principais candidatos.

![](https://imgs.xkcd.com/comics/is_it_worth_the_time.png)

Fonte: xkcd

Neste caso, a tarefa é a de entrada de dados cadastrais de fornecedores na solução de ERP da empresa. O processo tem uma boa dose de trabalho manual e conferências, além de dados não-estruturados e por vezes inconsistentes. Além disso, toda a comunicação sobre o processo se dá por e-mail. Avaliamos que para facilitar a tarefa de automação seria necessário tornar o processo mais ágil, e com informações centralizadas ("single source of truth"). Assim, criamos um *database* de solicitações de cadastro e mudamos o formulário de cadastro para que o solicitante possa consultar o status de determinado processo, diminuindo a comunicação por e-mail.

# Desenvolvimento

A ferramenta escolhida foi o [UiPath Community Edition](https://www.uipath.com). Trata-se de uma poderosa ferramenta de automação, que combina uma interface gráfica amigável e uma série de recursos chamados *atividades* que refletem tarefas comuns de escritório e simulam ações de um usuário humano. Ainda, o suporte às linguagens VB.Net e C# adicionam flexibilidade à solução de automação.

Aqui, o principal desafio foi naturalmente ganhar intimidade com o software. Felizmente, a interface do usuário é relativamente amigável! Em todo caso foi bastante importante realizar os cursos básicos na [UiPath Academy](https://academy.uipath.com/learn), uma plataforma de treinamento para o software, pois há alguns paradigmas que devem ser observados, como por exemplo a diferença entre variáveis e argumentos (o primeiro é interno à tarefa de automação, enquanto o segundo é externo). Noções de lógica de programação também foram importantes, pois a lógica dos processos de negócio podem evoluir rapidamente em complexidade. Ainda, quando trabalhamos com interfaces do usuário, cada ação nos leva a diferentes estados do sistema, e o desenvolvedor precisa definir saídas para os possíveis erros que o sistema pode apresentar.

Seguem mais algumas estratégias que ajudaram bastante nessa caminhada de alguns meses:

- Mini-provas de conceito: é possível testar cada passo do projeto, definido fluxogramas para cada caso requerido pelos sistemas que interagem com o robô. Neste caso, assegurar-se que as "peças menores" funcionam dá segurança para avançar na complexidade do projeto;
- Modularidade: cada elemento que foi programado pode ser posteriormente reutilizado, aumentando o valor do projeto para desenvolvimentos futuros. Para isso, é interessante desenvolver os processos de automação da forma mais geral e flexível possível, evitando as famosas "gambiarras" que se restringem a casos bastante particulares;
- Controle de versão: manter um controle de versão do projeto (neste caso, no git) facilita bastante a visão do histórico do projeto e do trabalho em equipe. O fato de os arquivos do UiPath serem _machine-readable_ ao invés de binários também simplificam a visualização de diferenças entre *commits*.

## *On premises* ou *cloud*?

Em determinado ponto fiquei em dúvida se poderíamos manter todos os controles da automação localmente. Também, o UiPath disponibiliza um [template](https://github.com/UiPath/ReFrameWork) que facilita a integração com a [plataforma *cloud*](https://platform.uipath.com/portal_/cloudrpa). A vantagem é que na plataforma é possível fazer toda a gestão das filas de processos, logs e provisão de máquinas/robôs para múltiplas tarefas.

# Final

A automatização de processos já é uma realidade e um dos pontos chave da chamada transformação digital que chega aos locais de trabalho em todo lugar. Felizmente, existem ferramentas que possibilitam a usuários de negócios desenvolver soluções com relativamente baixo código e alta eficiência e escalabilidade.

*(Nota: as visões aqui pontuadas são de inteira responsabilidade do autor e não refletem a posição oficial do seu empregador e seus gestores, acionistas e demais colaboradores.)*
