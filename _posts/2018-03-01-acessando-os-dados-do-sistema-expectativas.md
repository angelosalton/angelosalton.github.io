---
layout: post
title: Acessando os dados do Sistema Expectativas do Bacen no R
tags:
  - R
---

Recentemente o Banco Central do Brasil divulgou a disponibilização dos dados do Sistema de Expectativas de Mercado no seu Portal de Dados Abertos através de uma API, facilitando consultas mais complexas de dados. O Sistema compreende todas as informações sobre projeções de mercado dos principais agregados macroeconômicos (PIB, meta de juros, inflação, balança comercial), coletados em uma série de instituições financeiras, sendo assim um importante termômetro da economia brasileira.

A seguir, uma função do R que extrai os dados do Sistema, em um determinado período de tempo, que deve ser fornecido como argumento. O objetivo é simplesmente montar a URL de chamada para a API disponibilizada no Portal de Dados Abertos. A função faz o download de todas as variáveis existentes no banco, assim uma melhoria futura seria possibilitar a extração de subconjuntos dos dados. Existe uma série de bases de dados no Sistema Expectativas, com diferentes frequências de projeção. Neste exemplo, o acesso é direcionado às expectativas anuais.

```r
#' Title BCB Annual Sistema Expectativas interface
#' @author Angelo Salton
#' @param variables Measures to be selected. Possible options: "Media", "Mediana", "DesvioPadrao",
#' "CoeficienteVariacao", "Minimo", "Maximo".
#' @param data_inicial Initial date at which the data was projected, in ISO format.
#' @param data_final Final date at which the data was projected, in ISO format.
#' @param ...
#'
#' @import jsonlite
#' @return A data.frame.
#' @export
#'
#' @examples
#' 
bcbexpect_annual <- function(variables = "Mediana", start_date, end_date, ... ){
  # Required packages
  require(jsonlite)
  
  # variaveis
  variaveis_c <- paste("Indicador", "IndicadorDetalhe", "Data",
                      "DataReferencia", variables, sep = ",") # devem sempre estar presentes
  
  # datas
  timespan <- paste0("Data%20gt%20'", start_date, "'%20and%20", "Data%20lt%20'", end_date,"'")
  
  # montar a URL de query
  baseurl <- "https://olinda.bcb.gov.br/olinda/servico/Expectativas/versao/v1/odata/"
  query_url <- paste(baseurl, "ExpectativasMercadoAnuais", "?%24select=",
                    variaveis_c, "&%24filter=", timespan, sep = "", collapse = "")
  
  data <- fromJSON(query_url)$value
  return(data)
}

```
A função deve retornar um data frame com as variáveis de interesse, a data em que as projeções foram coletadas, o horizonte de previsão e os números em si. É possível fazer consultas que caracterizam toda a distribuição de frequência dos dados que se queira analisar.

_P.S.: Essa funcionalidade foi incluída no pacote [BETS](https://github.com/nmecsys/BETS), mantido pelo pessoal do IBRE/FGV._