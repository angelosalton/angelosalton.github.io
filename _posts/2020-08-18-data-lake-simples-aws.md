---
title: Um data lake simples com Amazon Web Services
tags:
  - AWS
  - Python
---

Este post descreve uma solução de *data lake* de pequeno porte, resultado das minhas experiências iniciais com a [Amazon Web Services](http://aws.amazon.com/).

# Motivação

Cada vez mais as empresas e o meio acadêmico têm utilizado a computação em nuvem para resolver problemas voltados à tecnologia da informação. As descobertas recentes em inteligência artificial sem dúvida aceleram esse processo, e plataformas como AWS, [Google Cloud]() e [Azure]() têm como vantagem oferecer soluções prontas para atacar estes desafios.

Independente do porte das empresas, cada companhia se encontra em um determinado estágio de maturidade em soluções *data-driven*. No meu caso em particular, o desafio em questão estava em construir um banco de dados, consolidando informações de diversas fontes, que até então se encontravam dispersas em sites, APIs e planilhas.

# Vamos por partes

As plataformas *cloud* em geral oferecem soluções de ETL (*extract, transform and load*) bastante robustas. Em especial, o [Apache Spark](https://spark.apache.org/) é uma ferramenta especializada na gestão de grandes volumes de dados. Contudo, para o meu caso de uso específico eu precisava mesmo de uma ferramenta mais **simples**.

[O post](#referências) no blog da AWS sobre os jobs em shell Python do AWS Glue (???) foi um ótimo ponto de partida. Contextualizando: o [AWS Glue](https://aws.amazon.com/pt/glue/) facilita a descoberta e a catalogação de dados que estejam no S3, que é o espaço de armazenamento principal no *cloud* da Amazon.

Confesso que ~~bati cabeça~~ sofri bastante até chegar nessa solução, devido às complexidades adicionais do Apache Spark. Certamente as melhores práticas indicam o seu uso, mas aí já saímos do **simples** (e eu também não sou nenhum *data engineer*). A vantagem de usar o Python shell é a possibilidade de usar bibliotecas como o [aws-data-wrangler](https://github.com/awslabs/aws-data-wrangler) (o [pandas](https://pandas.pydata.org/) na AWS) ou qualquer outra que você precisar, basta adicionar como dependências do seu projeto. Você pode salvar o resultado da sua extração de dados como um `*.csv` no S3, mas é recomendado usar o formato [Parquet](https://parquet.apache.org/), mais rápido e econômico para o nosso uso de caso.

Abaixo, segue um diagrama com as ferramentas usadas na solução:

![png](/assets/images/aws-diagrama.png)

Na prática, o que acontece é o seguinte:

1. Glue é acionado (sob demanda ou periodicamente) em um *job* e roda o script Python, que lê o dado de algum lugar e grava em um *bucket* do S3
2. Em seguida, um *crawler* do Glue varre o *bucket*, gerando metadados sobre os dados encontrados (tabelas, tipos de dados)

Tudo isso pode ser orquestrado através dos *workflows* do AWS Glue. A cereja do bolo é que não é necessário mover tudo para um banco de dados relacional: o [AWS Athena](https://aws.amazon.com/pt/athena/) permite rodar queries SQL sobre o catálogo de dados do Glue, a um custo bem baixo. Ele é baseado no [Presto](https://prestodb.io/).

# Código

Os códigos descritos aqui podem ser encontrados no repositório [**GitHub**](http://github.com/angelosalton/aws-simple-data-lake). No repositório também há um script do CloudFormation que lança os principais serviços na sua instância da AWS. Você vai precisar ajustar os parâmetros conforme a necessidade. {: .notice--primary}

# Referências

[AWS Blog - Usando Python shell e Pandas no AWS Glue para processar datasets pequenos e médios](https://aws.amazon.com/pt/blogs/aws-brasil/usando-python-shell-e-pandas-no-aws-glue-para-processar-datasets-pequenos-e-medios/)