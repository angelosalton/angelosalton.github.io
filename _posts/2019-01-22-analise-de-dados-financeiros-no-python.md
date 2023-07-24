---
layout: post
title: Análise de dados financeiros com pandas e matplotlib
tags:
  - Python
---

Aqui veremos uma análise de dados financeiros e de performance de uma empresa fictícia, para mostrar as capacidades de análise e visualização de dados no Python, através dos pacotes [pandas](https://pandas.pydata.org/) e [matplotlib](https://matplotlib.org/). A empresa possui os seguintes dados:

- *Visitas*: Base de quantas visitas temos no site, separadas por departamento, produto e região do país
- *Vendas*: Base de quantas vendas tivemos, separadas por departamento, produto e região do país
- *Financeiro*: Dados financeiros da venda, como receita com preço (após descontos), receita com frete (após descontos), custo do produto, custo de entrega, gastos com marketing e descontos dados em preço e frete.

Propõe-se uma análise dos indicadores de performance da empresa, considerando as informações disponíveis. Vamos importar os pacotes necessários e os dados:


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

fin = pd.read_csv('~/Downloads/Financeiro.csv', sep=';', encoding='windows-1252', decimal=b',')
ven = pd.read_csv('~/Downloads/Vendas.csv', sep=';', encoding='windows-1252', decimal=b',')
vis = pd.read_csv('~/Downloads/visitas.csv', sep=';', encoding='windows-1252', decimal=b',')

df = pd.merge(fin, ven)
df = pd.merge(df, vis)
df.Mes = pd.to_datetime(df.Mes)
```

Uma amostra da base de dados inicial, com dados do setor financeiro, de vendas e visitas:


```python
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Mes</th>
      <th>Departamento</th>
      <th>UF</th>
      <th>Item</th>
      <th>Faturamento_Produto</th>
      <th>Faturamento_Frete</th>
      <th>Custo_Produto</th>
      <th>Custo_Frete</th>
      <th>Custo_Mkt</th>
      <th>Desconto_Produto</th>
      <th>Desconto_Frete</th>
      <th>Vendas</th>
      <th>Visitas</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2018-06-01</td>
      <td>Cadeiras</td>
      <td>SP</td>
      <td>1</td>
      <td>149250.0</td>
      <td>42984.0</td>
      <td>71640</td>
      <td>35820</td>
      <td>5970</td>
      <td>0.0</td>
      <td>4776.0</td>
      <td>590</td>
      <td>14304</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2018-06-01</td>
      <td>Cadeiras</td>
      <td>SP</td>
      <td>2</td>
      <td>148050.0</td>
      <td>30456.0</td>
      <td>76140</td>
      <td>25380</td>
      <td>4230</td>
      <td>0.0</td>
      <td>3384.0</td>
      <td>439</td>
      <td>14258</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2018-06-01</td>
      <td>Cadeiras</td>
      <td>SP</td>
      <td>3</td>
      <td>133650.0</td>
      <td>21384.0</td>
      <td>89100</td>
      <td>17820</td>
      <td>2970</td>
      <td>0.0</td>
      <td>2376.0</td>
      <td>292</td>
      <td>14325</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-07-01</td>
      <td>Cadeiras</td>
      <td>SP</td>
      <td>1</td>
      <td>314712.5</td>
      <td>94784.0</td>
      <td>177720</td>
      <td>88860</td>
      <td>74050</td>
      <td>55537.5</td>
      <td>23696.0</td>
      <td>1469</td>
      <td>14873</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2018-07-01</td>
      <td>Cadeiras</td>
      <td>SP</td>
      <td>2</td>
      <td>392105.0</td>
      <td>84352.0</td>
      <td>237240</td>
      <td>79080</td>
      <td>65900</td>
      <td>69195.0</td>
      <td>21088.0</td>
      <td>1333</td>
      <td>14849</td>
    </tr>
  </tbody>
</table>
</div>



Agora, vamos calcular faturamentos, custos e descontos totais, bem como os lucros de produto, frete e o lucro líquido:

$$F_{bruto} = F_{produto} + F_{frete}$$ (faturamentos)

$$C_{total} = C_{produto} + C_{frete} + C_{mkt}$$ (custos)

$$D_{total} = D_{produto} + D_{frete}$$ (descontos)

Os lucros:

$$L_{produto} = F_{produto} - C_{produto} - D_{produto}$$ (produto)

$$L_{frete} = F_{frete} - C_{frete} - D_{frete}$$ (fretes)

$$L_{liquido} = F_{bruto} - C_{total} - D_{total}$$ (líquido)

A seguir, calcula-se o custo marginal (o custo de se ofertar uma unidade a mais) dos itens pela relação:

$$C_{marginal} = \frac{C_{total}}{\textrm{Vendas}}$$

Podemos inferir o preço médio praticado, através da razão entre o faturamento bruto e as quantidades vendidas:

$$P_{médio} = \frac{F_{bruto}}{\textrm{Vendas}}$$

Agora, sabendo o custo marginal e o preço dos itens, podemos calcular as margens de lucro (_mark-ups_) em porcentagem:

$$M = \frac{P_{médio}-C_{marginal}}{P_{médio}} * 100$$


```python
df['Faturamento_Bruto'] = df.Faturamento_Produto + df.Faturamento_Frete
df['Custo_Total']       = df.Custo_Produto + df.Custo_Frete + df.Custo_Mkt
df['Desconto_Total']    = df.Desconto_Frete + df.Desconto_Produto

df['Lucro_Produto'] = df.Faturamento_Produto - df.Custo_Produto - df.Desconto_Produto
df['Lucro_Frete']   = df.Faturamento_Frete - df.Custo_Frete - df.Desconto_Frete
df['Lucro_Liquido'] = df.Faturamento_Bruto - df.Custo_Total - df.Desconto_Total

df['Custo_Marginal'] = (df.Custo_Total) / df.Vendas
df['Preco_Medio']    = df.Faturamento_Bruto / df.Vendas
df['Markup']      = (df.Preco_Medio - df.Custo_Marginal)/df.Preco_Medio * 100
```

# Análise da performance da empresa

## Faturamento e lucro

Inicialmente, vamos ver como foi a performance da empresa ao longo do período estudado (dados em milhares de R$):


```python
temp = df.groupby('Mes')['Faturamento_Bruto','Custo_Total','Desconto_Total','Lucro_Liquido'].agg('sum', margins=True)/1000
temp
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Faturamento_Bruto</th>
      <th>Custo_Total</th>
      <th>Desconto_Total</th>
      <th>Lucro_Liquido</th>
    </tr>
    <tr>
      <th>Mes</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2018-06-01</th>
      <td>3861.4535</td>
      <td>3162.896</td>
      <td>146.8465</td>
      <td>551.7110</td>
    </tr>
    <tr>
      <th>2018-07-01</th>
      <td>9071.9396</td>
      <td>8522.998</td>
      <td>1483.1344</td>
      <td>-934.1928</td>
    </tr>
    <tr>
      <th>2018-08-01</th>
      <td>7561.4144</td>
      <td>6807.553</td>
      <td>867.2816</td>
      <td>-113.4202</td>
    </tr>
    <tr>
      <th>2018-09-01</th>
      <td>4937.5492</td>
      <td>4093.750</td>
      <td>176.5648</td>
      <td>667.2344</td>
    </tr>
  </tbody>
</table>
</div>




```python
temp.plot(title='Resultados da empresa (R$ mil)')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1f0753aa780>




![png](/assets/images/testeb2/output_8_1.png)


Em junho, a empresa apresentou lucro líquido de R$ 551 mil, seguido de prejuízos em julho (-R$ 934 mil) e agosto (-R$ 113 mil), para uma recuperação no mês de setembro (R$ 667 mil). O resultado agregado no período foi positivo (R$ 171 mil).

Agora, vamos ver quais são os departamentos com maior faturamento e lucro, no somatório dos meses observados e dos produtos:


```python
df.groupby(['Departamento'])['Faturamento_Bruto','Lucro_Liquido'].sum().round(2)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Faturamento_Bruto</th>
      <th>Lucro_Liquido</th>
    </tr>
    <tr>
      <th>Departamento</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bolas de Gude</th>
      <td>464448.7</td>
      <td>111036.4</td>
    </tr>
    <tr>
      <th>Cadeiras</th>
      <td>6141458.0</td>
      <td>-234484.0</td>
    </tr>
    <tr>
      <th>Maquinas Fotogr ficas</th>
      <td>18826450.0</td>
      <td>294780.0</td>
    </tr>
  </tbody>
</table>
</div>



Vemos que, ao longo dos meses observados, o departamento de máquinas fotográficas obteve a maior rentabilidade, considerando todas as regiões e itens ofertados. Nota-se também que o setor de cadeiras apresentou prejuízos.

Agora, a evolução da performance mês a mês, na soma dos produtos:


```python
temp = df.groupby(['Mes','Departamento'])['Faturamento_Bruto','Lucro_Liquido'].sum().round(2)
temp.unstack()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="3" halign="left">Faturamento_Bruto</th>
      <th colspan="3" halign="left">Lucro_Liquido</th>
    </tr>
    <tr>
      <th>Departamento</th>
      <th>Bolas de Gude</th>
      <th>Cadeiras</th>
      <th>Maquinas Fotogr ficas</th>
      <th>Bolas de Gude</th>
      <th>Cadeiras</th>
      <th>Maquinas Fotogr ficas</th>
    </tr>
    <tr>
      <th>Mes</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2018-06-01</th>
      <td>84315.5</td>
      <td>868238.0</td>
      <td>2908900.0</td>
      <td>28395.0</td>
      <td>279796.0</td>
      <td>243520.0</td>
    </tr>
    <tr>
      <th>2018-07-01</th>
      <td>155948.6</td>
      <td>2431401.0</td>
      <td>6484590.0</td>
      <td>38980.2</td>
      <td>-214633.0</td>
      <td>-758540.0</td>
    </tr>
    <tr>
      <th>2018-08-01</th>
      <td>130574.4</td>
      <td>1792560.0</td>
      <td>5638280.0</td>
      <td>32044.8</td>
      <td>-329945.0</td>
      <td>184480.0</td>
    </tr>
    <tr>
      <th>2018-09-01</th>
      <td>93610.2</td>
      <td>1049259.0</td>
      <td>3794680.0</td>
      <td>11616.4</td>
      <td>30298.0</td>
      <td>625320.0</td>
    </tr>
  </tbody>
</table>
</div>



Aqui é possível ver uma tendência do mercado em geral: percebe-se um aquecimento das vendas nos meses de julho e agosto. O departamento de bolas de gude apresentou um crescimento expressivo de junho para julho. Para ter uma melhor ideia da evolução dos indicadores, vejamos os dados em termos da variação em relação ao mês anterior (perde-se uma observação, pois não dispomos dos dados de maio):


```python
temp2 = temp.unstack().pct_change()*100
temp2.round(2)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="3" halign="left">Faturamento_Bruto</th>
      <th colspan="3" halign="left">Lucro_Liquido</th>
    </tr>
    <tr>
      <th>Departamento</th>
      <th>Bolas de Gude</th>
      <th>Cadeiras</th>
      <th>Maquinas Fotogr ficas</th>
      <th>Bolas de Gude</th>
      <th>Cadeiras</th>
      <th>Maquinas Fotogr ficas</th>
    </tr>
    <tr>
      <th>Mes</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2018-06-01</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-07-01</th>
      <td>84.96</td>
      <td>180.04</td>
      <td>122.92</td>
      <td>37.28</td>
      <td>-176.71</td>
      <td>-411.49</td>
    </tr>
    <tr>
      <th>2018-08-01</th>
      <td>-16.27</td>
      <td>-26.27</td>
      <td>-13.05</td>
      <td>-17.79</td>
      <td>53.73</td>
      <td>-124.32</td>
    </tr>
    <tr>
      <th>2018-09-01</th>
      <td>-28.31</td>
      <td>-41.47</td>
      <td>-32.70</td>
      <td>-63.75</td>
      <td>-109.18</td>
      <td>238.96</td>
    </tr>
  </tbody>
</table>
</div>



Aqui percebem-se mais claramente as tendências de receitas. Para o faturamento bruto, observa-se um salto na variação de junho para julho, seguido de queda nos meses seguintes. Aqui, os setores de cadeiras e máquinas fotográficas merecem destaque, pois em alguns meses existe uma relação inversa entre o faturamento bruto e o lucro líquido dos departamentos. Isto sugere que precisamos analisar cuidadosamente a estrutura de custos e descontos destes setores, o que será feito adiante.

A seguir, veremos no gráfico a evolução desses indicadores:


```python
temp2['Faturamento_Bruto'].plot(title='Faturamento Bruto (var. %)')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1f0757306a0>




![png](/assets/images/testeb2/output_16_1.png)



```python
temp2['Lucro_Liquido'].plot(title='Lucro Líquido (var. %)')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1f07579cd30>




![png](/assets/images/testeb2/output_17_1.png)


Vamos analisar agora quais os itens mais rentáveis, em termos de lucro total ao longo do período:


```python
temp = df.groupby(['Departamento','Item'])['Faturamento_Bruto','Lucro_Liquido'].sum().round(2)
temp
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Faturamento_Bruto</th>
      <th>Lucro_Liquido</th>
    </tr>
    <tr>
      <th>Departamento</th>
      <th>Item</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="3" valign="top">Bolas de Gude</th>
      <th>1</th>
      <td>170946.35</td>
      <td>40743.7</td>
    </tr>
    <tr>
      <th>2</th>
      <td>154184.50</td>
      <td>37215.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>139317.85</td>
      <td>33077.7</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Cadeiras</th>
      <th>1</th>
      <td>1946127.50</td>
      <td>16035.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2102926.50</td>
      <td>22143.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2092404.00</td>
      <td>-272662.0</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Maquinas Fotogr ficas</th>
      <th>1</th>
      <td>4103445.00</td>
      <td>-244990.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5514950.00</td>
      <td>22120.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9208055.00</td>
      <td>517650.0</td>
    </tr>
  </tbody>
</table>
</div>



No total dos meses observados, os itens vendidos mais lucrativos foram:

- Bolas de gude: item 1
- Cadeiras: item 2
- Máquinas fotográficas: item 3

Agora, vamos ver no gráfico como andam as margens de lucro mínimas, médias e máximas para todo o período, de cada item:


```python
temp = df.groupby(['Departamento','Item'])['Markup'].agg(['min','mean','max']).round(2)
temp
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>min</th>
      <th>mean</th>
      <th>max</th>
    </tr>
    <tr>
      <th>Departamento</th>
      <th>Item</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="3" valign="top">Bolas de Gude</th>
      <th>1</th>
      <td>4.76</td>
      <td>26.96</td>
      <td>41.02</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4.76</td>
      <td>26.96</td>
      <td>41.02</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4.76</td>
      <td>26.96</td>
      <td>41.02</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Cadeiras</th>
      <th>1</th>
      <td>3.77</td>
      <td>18.87</td>
      <td>40.99</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5.26</td>
      <td>20.11</td>
      <td>40.76</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-8.75</td>
      <td>7.27</td>
      <td>29.12</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Maquinas Fotogr ficas</th>
      <th>1</th>
      <td>-11.11</td>
      <td>2.95</td>
      <td>14.67</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-13.45</td>
      <td>8.97</td>
      <td>23.42</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3.51</td>
      <td>13.53</td>
      <td>21.90</td>
    </tr>
  </tbody>
</table>
</div>



Calculando as margens médias, descobrimos que o departamento de bolas de gude exerce a mesma margem de lucro para todos os seus itens, que por sinal são as maiores dentro da empresa (aprox. 27%), embora tenha aplicado margens de até 41% sobre os itens.

O departamento de cadeiras exerce margens um pouco menores (entre 7% e 18%, na média), embora também tenha aplicado margens de quase 40% (itens 1 e 2). No caso do item 3, acende-se um sinal de alerta, com margens mínimas de -8%, ou seja, itens vendidos com preços vendidos abaixo dos custos.

O mesmo aconteceu no departamento de máquinas fotográficas para os itens 1 e 2. Em geral, as margens médias deste departamento são um pouco menores do que nos departamentos de bolas de gude e cadeiras. Anteriormente vimos a capacidade de faturamento deste departamento, portanto aqui percebe-se um potencial para aumento de preços, visando a saúde financeira da empresa.

Vamos ver a evolução dos dados acima no tempo, para as médias, no gráfico abaixo:


```python
df.groupby(['Mes','Departamento'])['Markup'].agg(['mean']).unstack().plot(title='Margens médias de lucro (%)')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1f075828390>




![png](/assets/images/testeb2/output_23_1.png)


## Estrutura de custos

Estudar a estrutura de custos é vital para buscar novas estratégias de aumentar a rentabilidade da empresa: enquanto estratégias pelo lado da receita dependem da demanda dos consumidores, alguns cortes de custos podem ser realizadas (i) no curto prazo e (ii) dentro dos limites operacionais da empresa.

Primeiramente, vamos ver quais são os custos médios observados ao longo dos meses, de produto, frete e marketing, por departamento e item:


```python
temp = df.groupby('Mes')['Custo_Produto','Custo_Frete','Custo_Mkt','Custo_Total']
temp.mean().plot(kind='line')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1f07588f9e8>




![png](/assets/images/testeb2/output_25_1.png)



```python
temp = df.groupby(['Departamento','Item'])['Custo_Produto','Custo_Frete','Custo_Mkt','Custo_Total','Custo_Marginal'].mean().round(2)
temp
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Custo_Produto</th>
      <th>Custo_Frete</th>
      <th>Custo_Mkt</th>
      <th>Custo_Total</th>
      <th>Custo_Marginal</th>
    </tr>
    <tr>
      <th>Departamento</th>
      <th>Item</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="3" valign="top">Bolas de Gude</th>
      <th>1</th>
      <td>1628.33</td>
      <td>7013.58</td>
      <td>1497.42</td>
      <td>10139.33</td>
      <td>12.09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1445.08</td>
      <td>6299.92</td>
      <td>1365.00</td>
      <td>9110.00</td>
      <td>11.47</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1330.75</td>
      <td>5723.25</td>
      <td>1222.25</td>
      <td>8276.25</td>
      <td>12.19</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Cadeiras</th>
      <th>1</th>
      <td>69860.00</td>
      <td>39945.00</td>
      <td>23593.33</td>
      <td>133398.33</td>
      <td>226.70</td>
    </tr>
    <tr>
      <th>2</th>
      <td>88200.00</td>
      <td>33567.50</td>
      <td>20380.00</td>
      <td>142147.50</td>
      <td>282.74</td>
    </tr>
    <tr>
      <th>3</th>
      <td>120000.00</td>
      <td>27325.00</td>
      <td>17193.33</td>
      <td>164518.33</td>
      <td>403.79</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Maquinas Fotogr ficas</th>
      <th>1</th>
      <td>285208.33</td>
      <td>21691.67</td>
      <td>22845.00</td>
      <td>329745.00</td>
      <td>1300.18</td>
    </tr>
    <tr>
      <th>2</th>
      <td>376250.00</td>
      <td>20070.00</td>
      <td>21435.00</td>
      <td>417755.00</td>
      <td>1785.14</td>
    </tr>
    <tr>
      <th>3</th>
      <td>632458.33</td>
      <td>16920.83</td>
      <td>17797.50</td>
      <td>667176.67</td>
      <td>3279.55</td>
    </tr>
  </tbody>
</table>
</div>



Os custos, em geral, são função da quantidade vendida de itens. O custo marginal, como exposto anteriormente, é uma métrica de custos que é ponderada pelas vendas. No departamento de bolas de gude, o item 3 foi o que apresentou o maior custo marginal. No departamento de cadeiras, o item 3 é o que apresentou o maior custo, e no departamento de máquinas fotográficas o item 3. Quaisquer estratégias de aumento de rentabilidade via redução de custos deve passar pelo estudo destes itens.

Todavia, nem sempre é possível fazer grandes intervenções nas negociações com fornecedores e custos de transporte: é preciso saber como cada etapa contribui sobre os custos totais. Vamos analisar em cada item a esta participação média de cada etapa dos custos sobre o total, em porcentagens:


```python
temp = temp.reset_index()
temp['Produto pelo Total'] = temp.Custo_Produto / temp.Custo_Total * 100
temp['Frete pelo Total'] = temp.Custo_Frete / temp.Custo_Total * 100
temp['Mkt pelo Total'] = temp.Custo_Mkt / temp.Custo_Total * 100
temp.drop(['Custo_Produto','Custo_Frete','Custo_Mkt','Custo_Total','Custo_Marginal'], axis=1, inplace=True)
temp.round(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Departamento</th>
      <th>Item</th>
      <th>Produto pelo Total</th>
      <th>Frete pelo Total</th>
      <th>Mkt pelo Total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bolas de Gude</td>
      <td>1</td>
      <td>16.060</td>
      <td>69.172</td>
      <td>14.768</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Bolas de Gude</td>
      <td>2</td>
      <td>15.863</td>
      <td>69.154</td>
      <td>14.984</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bolas de Gude</td>
      <td>3</td>
      <td>16.079</td>
      <td>69.153</td>
      <td>14.768</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Cadeiras</td>
      <td>1</td>
      <td>52.369</td>
      <td>29.944</td>
      <td>17.686</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Cadeiras</td>
      <td>2</td>
      <td>62.048</td>
      <td>23.615</td>
      <td>14.337</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Cadeiras</td>
      <td>3</td>
      <td>72.940</td>
      <td>16.609</td>
      <td>10.451</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Maquinas Fotogr ficas</td>
      <td>1</td>
      <td>86.494</td>
      <td>6.578</td>
      <td>6.928</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Maquinas Fotogr ficas</td>
      <td>2</td>
      <td>90.065</td>
      <td>4.804</td>
      <td>5.131</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Maquinas Fotogr ficas</td>
      <td>3</td>
      <td>94.796</td>
      <td>2.536</td>
      <td>2.668</td>
    </tr>
  </tbody>
</table>
</div>



Vimos anteriormente que no departamento de bolas de gude o item 3 possui os maiores custos marginais de comercialização. Dentro deste departamento como um todo, as estruturas de custo são similares, e o custo do frete é o principal fator (cerca de 69% do custo total). Tratando-se de um produto simples e barato, estratégias de otimização da logística podem trazer grandes contribuições para a rentabilidade do departamento como um todo.

No departamento de cadeiras, a distribuição é mais equilibrada e heterogênea: os custos de produto representam cerca de 60% dos custos totais. No caso do item 1, observam-se a maior fatia dos custos com marketing (aprox. 17%). A aplicação bem-sucedida dos dispêndios com marketing depende da propensão à vender dadas as visitas no site, ponto que será visto adiante.

No departamento de máquinas fotográficas, os custos de produto representam cerca de 90% dos custos totais, e os dispêndios com frete e marketing são baixos (cerca de 5% cada). Analisando o faturamento da nossa empresa, vimos que os itens do departamento de máquinas fotográficas são altamente lucrativos e de alto valor agregado. Aqui, negociações com fornecedores que reduzam o custo marginal dos itens trarão grandes benefícios para a empresa.

## Descontos, visitas e vendas

Os descontos representam uma peça-chave da estrutura de custos da empresa, pois com eles pode-se obter uma vantagem competitiva no mercado. Portanto, a análise aqui concentra-se não somente em verificar qual a contribuição dos descontos para a estrutura de custos totais da empresa, mas em como os descontos têm alavancado as vendas.


```python
temp = df.groupby(['Departamento','Item'])['Desconto_Produto','Desconto_Frete','Desconto_Total','Vendas','Preco_Medio'].mean().reset_index().round(2)
temp.drop(['Vendas','Preco_Medio'], axis=1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Departamento</th>
      <th>Item</th>
      <th>Desconto_Produto</th>
      <th>Desconto_Frete</th>
      <th>Desconto_Total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bolas de Gude</td>
      <td>1</td>
      <td>363.97</td>
      <td>346.92</td>
      <td>710.89</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Bolas de Gude</td>
      <td>2</td>
      <td>333.93</td>
      <td>303.53</td>
      <td>637.46</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bolas de Gude</td>
      <td>3</td>
      <td>303.89</td>
      <td>273.21</td>
      <td>577.10</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Cadeiras</td>
      <td>1</td>
      <td>22079.88</td>
      <td>5362.83</td>
      <td>27442.71</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Cadeiras</td>
      <td>2</td>
      <td>26719.62</td>
      <td>4531.50</td>
      <td>31251.12</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Cadeiras</td>
      <td>3</td>
      <td>28743.50</td>
      <td>3827.00</td>
      <td>32570.50</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Maquinas Fotogr ficas</td>
      <td>1</td>
      <td>16527.92</td>
      <td>16096.67</td>
      <td>32624.58</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Maquinas Fotogr ficas</td>
      <td>2</td>
      <td>24069.17</td>
      <td>15911.67</td>
      <td>39980.83</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Maquinas Fotogr ficas</td>
      <td>3</td>
      <td>43622.92</td>
      <td>13400.83</td>
      <td>57023.75</td>
    </tr>
  </tbody>
</table>
</div>



Agora, podemos dividir os descontos por itens pelo respectivo volume de vendas, para descobrir qual têm sido o desconto necessário para vender uma unidade adicional de algum item, descritos pelas colunas `DescProd_Vendas`, `DescFret_Vendas` e `DescTot_Vendas`, que representam os descontos de produto, frete e total divididos pelas vendas, respectivamente. É possível ainda obter os descontos no preço médio dos itens em porcentagens através da relação:

$$ \textrm{Desconto no preço} = \frac{\textrm{DescTot_Vendas}}{P_{médio}} * 100 $$


```python
temp['DescProd_Vendas'] = temp.Desconto_Produto/temp.Vendas
temp['DescFret_Vendas'] = temp.Desconto_Frete/temp.Vendas
temp['DescTot_Vendas'] = temp.Desconto_Total/temp.Vendas
temp['Desconto_no_preco'] = temp.DescTot_Vendas/temp.Preco_Medio * 100

temp.drop(['Desconto_Produto','Desconto_Frete','Desconto_Total','Vendas','Preco_Medio'], axis=1, inplace=True)
temp.set_index(['Departamento','Item']).plot(y='Desconto_no_preco', kind='bar', legend=None, title='Descontos (%)')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1f075923d68>




![png](/assets/images/testeb2/output_33_1.png)


De acordo com os dados observados, os descontos no departamento de bolas de gude giram em torno de 5%, com praticamente nenhuma diferenciação entre os itens. O departamento de cadeiras tem uma política de descontos mais agressiva, com descontos de 17% em média. Neste departamento existe uma diferenciação maior entre os itens, sendo que o item 3 foi o que acumulou mais descontos (18,5%). No departamento de máquinas fotográficas, os descontos médios estão na faixa de 8,5%, sendo o item 1 o que recebeu mais descontos totais (aprox. 9,5%). Precisamos ver o volume de vendas de cada item para saber se os descontos estão sendo bem aplicados.

Agora, vamos ver as métricas de vendas e visitas. Vamos ver como foi o tráfego no período analisado.


```python
temp = df.groupby('Mes')['Visitas','Vendas'].sum()
temp
#temp.plot(kind='line', title='Visitas e vendas')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Visitas</th>
      <th>Vendas</th>
    </tr>
    <tr>
      <th>Mes</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2018-06-01</th>
      <td>181075</td>
      <td>8648</td>
    </tr>
    <tr>
      <th>2018-07-01</th>
      <td>181196</td>
      <td>19520</td>
    </tr>
    <tr>
      <th>2018-08-01</th>
      <td>176556</td>
      <td>15991</td>
    </tr>
    <tr>
      <th>2018-09-01</th>
      <td>180264</td>
      <td>10484</td>
    </tr>
  </tbody>
</table>
</div>



De acordo com a tabela, ao longo dos meses de junho a setembro as visitas se mantém estáveis em torno de 180.000, enquanto as vendas começam em 8648 para mais que dobrar no mês de julho, para depois tomar uma tendência de queda nos meses seguintes. Dados de frequência diária poderiam revelar a influência de datas comemorativas no tráfego do site.

Podemos calcular a taxa de conversão dos itens através da relação:

$$\textrm{Taxa de conversão} = \frac{\textrm{Vendas}}{\textrm{Visitas}} * 100$$


```python
temp = df.groupby(['Departamento','Item'])['Visitas','Vendas','Custo_Mkt','Preco_Medio'].mean().round(2)
temp.reset_index()
temp['Taxa de conversão'] = temp.Vendas / temp.Visitas * 100

temp['Taxa de conversão'].plot(kind='bar', title='Taxa de conversão (%)')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1f07598be48>




![png](/assets/images/testeb2/output_38_1.png)


Descobrimos que existe uma grande variabilidade entre os itens. No departamento de bolas de gude, o item 1 é o que possui a maior taxa de conversão (requer o menor número de visitas por venda) bem como o item 1 do setor de Cadeiras e o item 1 do setor de máquinas fotográficas. Estratégias de marketing que busquem aumentar a frequência dos consumidores na web podem focar nesses itens, pois requerem o menor investimento para alavancar vendas.

Sabe-se que os dispêndios com marketing objetivam ampliar as vendas dos itens oferecidos. Na tabela a seguir, calculamos a participação do custo médio de marketing no preço médio dos itens, calculando o gasto médio por item em marketing (representado pela coluna `Custo_Mkt_Vendas`), e em seguida a participação desse custo no preço médio dos itens, através das relações:

$$\textrm{Custo_Mkt_Vendas} = \frac{C_{mkt}}{\textrm{Vendas}}$$

$$\textrm{Custo_Mkt_Preço} = \frac{\textrm{Custo_Mkt_Vendas}}{P_{médio}}$$


```python
temp['Custo_Mkt_Vendas'] = temp.Custo_Mkt / temp.Vendas
temp['Custo_Mkt_Preço'] = temp.Custo_Mkt_Vendas / temp.Preco_Medio * 100
temp.drop(['Visitas','Vendas','Custo_Mkt','Preco_Medio'], axis=1).round(2)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Taxa de conversão</th>
      <th>Custo_Mkt_Vendas</th>
      <th>Custo_Mkt_Preço</th>
    </tr>
    <tr>
      <th>Departamento</th>
      <th>Item</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="3" valign="top">Bolas de Gude</th>
      <th>1</th>
      <td>10.52</td>
      <td>1.72</td>
      <td>10.41</td>
    </tr>
    <tr>
      <th>2</th>
      <td>9.80</td>
      <td>1.70</td>
      <td>10.75</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8.58</td>
      <td>1.72</td>
      <td>10.37</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Cadeiras</th>
      <th>1</th>
      <td>6.59</td>
      <td>40.71</td>
      <td>14.47</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5.66</td>
      <td>41.07</td>
      <td>11.47</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4.55</td>
      <td>43.15</td>
      <td>9.80</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Maquinas Fotogr ficas</th>
      <th>1</th>
      <td>8.69</td>
      <td>89.38</td>
      <td>6.61</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8.03</td>
      <td>90.80</td>
      <td>4.59</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6.79</td>
      <td>88.14</td>
      <td>2.33</td>
    </tr>
  </tbody>
</table>
</div>




```python
temp.unstack()['Custo_Mkt_Preço'].plot(kind='bar', title='Custos de marketing em relação ao preço médio (%)')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1f0759fa4a8>




![png](/assets/images/testeb2/output_41_1.png)


A partir dos dados calculados, vemos que no departamento de bolas de gude os gastos com marketing chegam ao redor de 10,5% em média. No departamento de cadeiras, esse número alcança aproximadamente 12%. O item 1 recebe o maior gasto em marketing (14,4%). O departamento de máquinas fotográficas é o que possui os menores gastos médios (4,5%).

## Análise regional

Nesta seção avaliaremos diferenças regionais nas principais métricas de desempenho da empresa. Vamos mostrar primeiro os preços, custos marginais e margens de lucro médias:


```python
temp = df.groupby(['UF','Departamento','Item'])['Preco_Medio','Custo_Marginal','Markup',].mean().round(2)
temp
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th>Preco_Medio</th>
      <th>Custo_Marginal</th>
      <th>Markup</th>
    </tr>
    <tr>
      <th>UF</th>
      <th>Departamento</th>
      <th>Item</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="9" valign="top">MT</th>
      <th rowspan="3" valign="top">Bolas de Gude</th>
      <th>1</th>
      <td>17.75</td>
      <td>15.26</td>
      <td>14.03</td>
    </tr>
    <tr>
      <th>2</th>
      <td>15.12</td>
      <td>13.03</td>
      <td>14.03</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17.89</td>
      <td>15.45</td>
      <td>14.03</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Cadeiras</th>
      <th>1</th>
      <td>271.16</td>
      <td>220.83</td>
      <td>18.58</td>
    </tr>
    <tr>
      <th>2</th>
      <td>345.75</td>
      <td>275.56</td>
      <td>19.30</td>
    </tr>
    <tr>
      <th>3</th>
      <td>421.02</td>
      <td>392.71</td>
      <td>5.86</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Maquinas Fotogr ficas</th>
      <th>1</th>
      <td>1400.65</td>
      <td>1361.26</td>
      <td>2.40</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1828.76</td>
      <td>1909.82</td>
      <td>-4.67</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3360.61</td>
      <td>2956.03</td>
      <td>12.49</td>
    </tr>
    <tr>
      <th rowspan="9" valign="top">PE</th>
      <th rowspan="3" valign="top">Bolas de Gude</th>
      <th>1</th>
      <td>18.05</td>
      <td>11.70</td>
      <td>35.07</td>
    </tr>
    <tr>
      <th>2</th>
      <td>18.15</td>
      <td>11.78</td>
      <td>35.07</td>
    </tr>
    <tr>
      <th>3</th>
      <td>18.11</td>
      <td>11.75</td>
      <td>35.07</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Cadeiras</th>
      <th>1</th>
      <td>282.86</td>
      <td>242.73</td>
      <td>13.52</td>
    </tr>
    <tr>
      <th>2</th>
      <td>355.98</td>
      <td>298.36</td>
      <td>15.34</td>
    </tr>
    <tr>
      <th>3</th>
      <td>436.64</td>
      <td>421.18</td>
      <td>2.64</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Maquinas Fotogr ficas</th>
      <th>1</th>
      <td>1208.30</td>
      <td>1200.00</td>
      <td>-0.76</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2098.28</td>
      <td>1733.88</td>
      <td>17.45</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4101.21</td>
      <td>3663.38</td>
      <td>10.64</td>
    </tr>
    <tr>
      <th rowspan="9" valign="top">SP</th>
      <th rowspan="3" valign="top">Bolas de Gude</th>
      <th>1</th>
      <td>13.70</td>
      <td>9.30</td>
      <td>31.79</td>
    </tr>
    <tr>
      <th>2</th>
      <td>14.12</td>
      <td>9.61</td>
      <td>31.79</td>
    </tr>
    <tr>
      <th>3</th>
      <td>13.83</td>
      <td>9.37</td>
      <td>31.79</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Cadeiras</th>
      <th>1</th>
      <td>289.90</td>
      <td>216.53</td>
      <td>24.50</td>
    </tr>
    <tr>
      <th>2</th>
      <td>372.32</td>
      <td>274.29</td>
      <td>25.70</td>
    </tr>
    <tr>
      <th>3</th>
      <td>463.33</td>
      <td>397.47</td>
      <td>13.32</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Maquinas Fotogr ficas</th>
      <th>1</th>
      <td>1450.30</td>
      <td>1339.26</td>
      <td>7.22</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2006.28</td>
      <td>1711.73</td>
      <td>14.13</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3907.90</td>
      <td>3219.24</td>
      <td>17.47</td>
    </tr>
  </tbody>
</table>
</div>



Percebe-se que alguns departamentos desempenham melhor em determinadas regiões. No Mato Grosso, as margens médias estão abaixo da média da empresa, sobretudo no departamento de máquinas fotográficas, que opera aproximadamente à preço de custo. O estado de Pernambuco apresenta ótimas margens no departamento de bolas de gude, enquanto que no departamento de cadeiras as margens estão abaixo da média. São Paulo apresenta as melhores margens no departamento de cadeiras e máquinas fotográficas. Nota-se a necessidade, em alguns itens, de reajuste de preços de produtos ou fretes (ou ainda, redução de descontos).

A seguir, o somatório das vendas e do lucro líquido no período:


```python
temp = df.groupby(['UF','Departamento'])['Vendas','Lucro_Liquido',].sum().round(2)
temp
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Vendas</th>
      <th>Lucro_Liquido</th>
    </tr>
    <tr>
      <th>UF</th>
      <th>Departamento</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="3" valign="top">MT</th>
      <th>Bolas de Gude</th>
      <td>6803</td>
      <td>10241.4</td>
    </tr>
    <tr>
      <th>Cadeiras</th>
      <td>2634</td>
      <td>-48790.0</td>
    </tr>
    <tr>
      <th>Maquinas Fotogr ficas</th>
      <td>2194</td>
      <td>-180710.0</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">PE</th>
      <th>Bolas de Gude</th>
      <td>11321</td>
      <td>61403.0</td>
    </tr>
    <tr>
      <th>Cadeiras</th>
      <td>4976</td>
      <td>-195161.0</td>
    </tr>
    <tr>
      <th>Maquinas Fotogr ficas</th>
      <td>2061</td>
      <td>-10480.0</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">SP</th>
      <th>Bolas de Gude</th>
      <td>10505</td>
      <td>39392.0</td>
    </tr>
    <tr>
      <th>Cadeiras</th>
      <td>10081</td>
      <td>9467.0</td>
    </tr>
    <tr>
      <th>Maquinas Fotogr ficas</th>
      <td>4068</td>
      <td>485970.0</td>
    </tr>
  </tbody>
</table>
</div>



Aqui emergem as diferenças de desempenho dos departamentos, nas diferentes localidades. Em Pernambuco, novamente o departamento de bolas de gude se destaca, bem como um desempenho no departamento de cadeiras melhor do que no Mato Grosso. O estado de São Paulo, dotado de um grande mercado consumidor, vendeu quase o mesmo em máquinas fotográficas que nos dois outros estados, para um lucro de quase R$ 500 mil. Ademais, novamente mostra seu potencial no departamento de cadeiras.

É intuitivo imaginar que a rentabilidade dos fretes sejam sensíveis à localização dos consumidores. Seguem os resultados do lucro médio dos fretes em cada região e departamento, dividido pelas vendas:


```python
temp = df.groupby(['UF','Departamento'])['Lucro_Frete','Vendas'].mean().reset_index().round(2)
temp['Lucro_Frete_Ponderado'] = temp.Lucro_Frete / temp.Vendas
temp.drop(['Lucro_Frete','Vendas'], axis=1, inplace=True)
temp.round(2)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>UF</th>
      <th>Departamento</th>
      <th>Lucro_Frete_Ponderado</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>MT</td>
      <td>Bolas de Gude</td>
      <td>1.09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MT</td>
      <td>Cadeiras</td>
      <td>-6.63</td>
    </tr>
    <tr>
      <th>2</th>
      <td>MT</td>
      <td>Maquinas Fotogr ficas</td>
      <td>-72.44</td>
    </tr>
    <tr>
      <th>3</th>
      <td>PE</td>
      <td>Bolas de Gude</td>
      <td>3.17</td>
    </tr>
    <tr>
      <th>4</th>
      <td>PE</td>
      <td>Cadeiras</td>
      <td>-6.89</td>
    </tr>
    <tr>
      <th>5</th>
      <td>PE</td>
      <td>Maquinas Fotogr ficas</td>
      <td>-150.63</td>
    </tr>
    <tr>
      <th>6</th>
      <td>SP</td>
      <td>Bolas de Gude</td>
      <td>2.24</td>
    </tr>
    <tr>
      <th>7</th>
      <td>SP</td>
      <td>Cadeiras</td>
      <td>2.50</td>
    </tr>
    <tr>
      <th>8</th>
      <td>SP</td>
      <td>Maquinas Fotogr ficas</td>
      <td>-122.03</td>
    </tr>
  </tbody>
</table>
</div>



Pode-se interpretar essa relação como um índice. Analisando os dados da tabela, nota-se que cada estado tem vantagens comparativas na parte logística, de acordo com cada departamento. Uma análise prévia não encontrou diferenças significativas entre itens de um mesmo departamento. No Mato Grosso, as perdas com frete são menores no departamento de máquinas fotográficas. Em Pernambuco, as vantagens aparecem no departamento de bolas de gude. O estado de São Paulo apresenta bons números em geral, e mais uma vez mostrando vantagens comparativas no departamento de cadeiras. É provável que aqui se encontre algum centro de distribuição destes itens.

Os números apresentados nesta seção revelam a importância da logística no funcionamento da empresa, sobretudo para os itens de aparentemente difícil transporte como os do departamento de cadeiras e máquinas fotográficas.
