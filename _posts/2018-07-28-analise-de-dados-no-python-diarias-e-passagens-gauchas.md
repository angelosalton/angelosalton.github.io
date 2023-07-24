---
layout: post
title: "Análise de dados no Python: Diárias e passagens gaúchas"
tags:
  - Python
---

Olá! Este é um exercício em análise e exploração de dados em Python. Aqui, vamos aplicar as funcionalidades do pacote ``pandas`` em um arquivo que contém o histório de concessões de diárias e passagens a pessoas físicas no estado do Rio Grande do Sul, em 2017. Os dados contém informações sobre órgãos responsáveis, origens e destinos de viagens, pessoas beneficiadas e valores pagos em R$ correntes de 2017.

Vamos começar carregando os pacotes necessários:


```python
%matplotlib inline
import pandas as pd
import numpy as np
import seaborn as sns
sns.set()
```

Agora, vamos fazer o download dos dados do Portal da Transparência do RS. Os dados vêm em formato ``*.csv``, com o separador ponto-e-vírgula:


```python
#!curl http://antigo.transparencia.rs.gov.br/ARQUIVOS/Diarias-RS-2017.zip
#!unzip Diarias-RS-2017.zip
df = pd.read_csv('Diarias-RS-2017.csv', sep=';', encoding='windows-1252')
```

Agora, vamos ver como os dados foram importados:


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
      <th>Exercicio</th>
      <th>Mes</th>
      <th>Poder</th>
      <th>Cod_Setor_Gov</th>
      <th>Setor</th>
      <th>Cod_orgao</th>
      <th>Orgao</th>
      <th>Cod_UO</th>
      <th>UO</th>
      <th>Cod_Credor</th>
      <th>...</th>
      <th>DataInicio</th>
      <th>DataFim</th>
      <th>Quantidade</th>
      <th>QuantidadeMeia</th>
      <th>Destino</th>
      <th>Tipo</th>
      <th>Motivo</th>
      <th>Valor</th>
      <th>Origem</th>
      <th>Empenho</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017</td>
      <td>1</td>
      <td>PODER EXECUTIVO</td>
      <td>19</td>
      <td>DIRETA</td>
      <td>20</td>
      <td>SECRETARIA DA SAUDE</td>
      <td>2095</td>
      <td>FUNDO ESTADUAL DE SAUDE</td>
      <td>6998</td>
      <td>...</td>
      <td>14/12/2016</td>
      <td>14/12/2016</td>
      <td>0</td>
      <td>1</td>
      <td>BUTIA</td>
      <td>DIARIA COMUM VENCID-CIVIL</td>
      <td>INSPECOES/AUDITORIAS</td>
      <td>61,50</td>
      <td>CHARQUEADAS</td>
      <td>17000146930</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017</td>
      <td>1</td>
      <td>PODER EXECUTIVO</td>
      <td>19</td>
      <td>DIRETA</td>
      <td>20</td>
      <td>SECRETARIA DA SAUDE</td>
      <td>2095</td>
      <td>FUNDO ESTADUAL DE SAUDE</td>
      <td>6998</td>
      <td>...</td>
      <td>07/12/2016</td>
      <td>07/12/2016</td>
      <td>0</td>
      <td>1</td>
      <td>CAMAQUA</td>
      <td>DIARIA COMUM VENCID-CIVIL</td>
      <td>INSPECOES/AUDITORIAS</td>
      <td>61,50</td>
      <td>CHARQUEADAS</td>
      <td>17000146996</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017</td>
      <td>1</td>
      <td>PODER EXECUTIVO</td>
      <td>19</td>
      <td>DIRETA</td>
      <td>20</td>
      <td>SECRETARIA DA SAUDE</td>
      <td>2095</td>
      <td>FUNDO ESTADUAL DE SAUDE</td>
      <td>6998</td>
      <td>...</td>
      <td>28/12/2016</td>
      <td>28/12/2016</td>
      <td>0</td>
      <td>1</td>
      <td>SAO JERONIMO</td>
      <td>DIARIA COMUM VENCID-CIVIL</td>
      <td>INSPECOES/AUDITORIAS</td>
      <td>61,50</td>
      <td>CHARQUEADAS</td>
      <td>17000157622</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017</td>
      <td>1</td>
      <td>PODER EXECUTIVO</td>
      <td>19</td>
      <td>DIRETA</td>
      <td>20</td>
      <td>SECRETARIA DA SAUDE</td>
      <td>2095</td>
      <td>FUNDO ESTADUAL DE SAUDE</td>
      <td>6998</td>
      <td>...</td>
      <td>18/01/2017</td>
      <td>18/01/2017</td>
      <td>0</td>
      <td>1</td>
      <td>BARRA DO RIBEIRO</td>
      <td>DIARIA COMUM VENCID-CIVIL</td>
      <td>REUNIOES TECNICAS</td>
      <td>61,50</td>
      <td>CHARQUEADAS</td>
      <td>17000303577</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017</td>
      <td>1</td>
      <td>PODER EXECUTIVO</td>
      <td>19</td>
      <td>DIRETA</td>
      <td>15</td>
      <td>SECRETARIA DA AGRICULTURA, PECUARIA E IRRIGACAO</td>
      <td>1501</td>
      <td>GABINETE E ORGAOS CENTRAIS</td>
      <td>8745</td>
      <td>...</td>
      <td>25/10/2016</td>
      <td>25/10/2016</td>
      <td>0</td>
      <td>1</td>
      <td>SANTA MARIA</td>
      <td>DIARIA COMUM VENCID-CIVIL</td>
      <td>REUNIOES ADMINISTRATIVAS</td>
      <td>61,50</td>
      <td>JARI</td>
      <td>16006041272</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>




```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 268874 entries, 0 to 268873
    Data columns (total 21 columns):
    Exercicio         268874 non-null int64
    Mes               268874 non-null int64
    Poder             268874 non-null object
    Cod_Setor_Gov     268874 non-null int64
    Setor             268874 non-null object
    Cod_orgao         268874 non-null int64
    Orgao             268874 non-null object
    Cod_UO            268874 non-null int64
    UO                268874 non-null object
    Cod_Credor        268874 non-null int64
    Favorecido        268874 non-null object
    DataInicio        262676 non-null object
    DataFim           262676 non-null object
    Quantidade        268874 non-null int64
    QuantidadeMeia    268874 non-null int64
    Destino           268530 non-null object
    Tipo              268874 non-null object
    Motivo            268874 non-null object
    Valor             268874 non-null object
    Origem            268874 non-null object
    Empenho           268874 non-null int64
    dtypes: int64(9), object(12)
    memory usage: 43.1+ MB


A coluna com os valores dos empenhos foram interpretadas como texto, por causa da vírgula como separador decimal. Vamos corrigir:


```python
df['Valor'] = df.Valor.str.replace(pat=',', repl='.')
df['Valor'] = df.Valor.astype(float)
```

Agora, vamos ocultar o nome completo dos beneficiados, para fins didáticos:


```python
df['Favorecido2'] = df.Favorecido.str.split()
df['Favorecido2'] = df.Favorecido2.apply(lambda x: x[-1])
```

A partir daqui já podemos fazer algumas análises, com o uso da função ``groupby`` do pacote ``pandas``. Com ela é possível obter estatísticas agregadas por grupos. Vamos descobrir quais servidores receberam os maiores valores em diárias e passagens:


```python
df.groupby(['Cod_Credor','Favorecido2'])['Valor'].sum().nlargest(10)
```




    Cod_Credor  Favorecido2
    21594481    PAESE          60088.33
    21544697    CAYE           51664.02
    21570051    SILVA          51336.00
    10161040    RAMOS          51110.52
    34346732    GARCIA         49799.23
    18567100    MAINARDI       48719.62
    6051006     PEIXOTO        48598.50
    23170433    BORBA          48479.53
    33839395    SILVA          47125.32
    40092020    GOMES          45246.32
    Name: Valor, dtype: float64



Ou então, quais os maiores fluxos entre unidades orçamentárias e órgãos que concedem os benefícios:


```python
df.groupby(['UO','Orgao'])['Valor'].sum().nlargest(10).round()
```




    UO                                            Orgao                                          
    BRIGADA MILITAR                               SECRETARIA DA SEGURANCA PUBLICA                    28326579.0
    DEPARTAMENTO AUTONOMO DE ESTRADAS DE RODAGEM  DEPARTAMENTO AUTONOMO DE ESTRADAS DE RODAGEM       13128503.0
    SUPERINTENDENCIA DOS SERVICOS PENITENCIARIOS  SECRETARIA DA SEGURANCA PUBLICA                     7612260.0
    CORPO DE BOMBEIROS MILITAR                    SECRETARIA DA SEGURANCA PUBLICA                     4765460.0
    DEPARTAMENTO ESTADUAL DE TRANSITO             DEPARTAMENTO ESTADUAL DE TRANSITO                   3887284.0
    POLICIA CIVIL                                 SECRETARIA DA SEGURANCA PUBLICA                     3766606.0
    ASSEMBLEIA LEGISLATIVA                        ASSEMBLEIA LEGISLATIVA                              3193683.0
    GABINETE E ORGAOS CENTRAIS                    SECRETARIA DA AGRICULTURA, PECUARIA E IRRIGACAO     3156240.0
    PROCURADORIA-GERAL DE JUSTICA                 MINISTERIO PUBLICO                                  3103149.0
    GABINETE E ORGAOS CENTRAIS                    SECRETARIA DA SEGURANCA PUBLICA                     3085936.0
    Name: Valor, dtype: float64



Quais são as viagens mais frequentes?


```python
df.groupby(['Origem','Destino'])['Quantidade'].sum().nlargest(20)
```




    Origem            Destino       
    PORTO ALEGRE      ALEGRETE          67886
                      ANTONIO PRADO     60853
                      AGUDO             45462
                      TRAMANDAI         21107
                      PORTO ALEGRE      18483
    PASSO FUNDO       PORTO ALEGRE      12799
    PORTO ALEGRE      CAPAO DA CANOA    12470
    SANTA MARIA       PORTO ALEGRE      11074
    PORTO ALEGRE      TORRES             7322
                      CIDREIRA           6514
                      OSORIO             5372
                      CHARQUEADAS        4953
    SANTO ANGELO      PORTO ALEGRE       4769
    PELOTAS           PORTO ALEGRE       4201
    SANTA MARIA       CAPAO DA CANOA     3617
    PORTO ALEGRE      CAXIAS DO SUL      3492
    CACHOEIRA DO SUL  PORTO ALEGRE       3478
    PELOTAS           RIO GRANDE         3272
    PORTO ALEGRE      PASSO FUNDO        2959
    SANTA MARIA       NOVO HAMBURGO      2881
    Name: Quantidade, dtype: int64



Quais viagens representam os maiores valores gastos em 2017? Elas coincidem com os valores mostrados acima?


```python
df.groupby(['Origem','Destino'])['Valor'].sum().nlargest(20)
```




    Origem            Destino       
    PORTO ALEGRE      TRAMANDAI         2721824.31
    PASSO FUNDO       PORTO ALEGRE      1985408.88
    SANTA MARIA       PORTO ALEGRE      1796298.20
    PORTO ALEGRE      CAPAO DA CANOA    1627620.17
                      ALEGRETE          1252773.67
                      ANTONIO PRADO     1139786.58
                      TORRES             975725.28
    N/D               N/D                946698.35
    PORTO ALEGRE      DF                 925746.10
                      CIDREIRA           819431.60
                      AGUDO              801317.16
                      OSORIO             793824.58
    SANTO ANGELO      PORTO ALEGRE       746511.73
    PELOTAS           PORTO ALEGRE       720271.00
    PORTO ALEGRE      CAXIAS DO SUL      635554.39
                      EXTERIOR           593282.20
                      SP                 579742.97
    CACHOEIRA DO SUL  PORTO ALEGRE       554328.30
    CHARQUEADAS       PORTO ALEGRE       550244.71
    PORTO ALEGRE      CHARQUEADAS        548256.21
    Name: Valor, dtype: float64



Cada diária/passagem deve ter registrada um motivo associado:


```python
df.groupby('Motivo')['Valor'].sum().nlargest(20).round()
```




    Motivo
    OPERACAO LITORAL             25177625.0
    SERVICOS TECNICOS             9045394.0
    REFORCOS A OUTRA OPM          8213390.0
    OPERACAO CANARINHO            3538243.0
    RESSARC SERVIDOR BPR DAER     3413129.0
    PARTICIP.EM TREIN./CURSOS     2990233.0
    SERV.CONS.PAV.RODOV.-DAER     2892201.0
    FISCAL.DE SERV.CONCEDIDOS     2797213.0
    TRANSPORTE DE PESSOAS         2056261.0
    PARTICIPACAO-EVENTOS          1971834.0
    REUNIOES ADMINISTRATIVAS      1929856.0
    ADMINISTRATIVO MP             1711359.0
    SERV.PATRULH.RODOVIARIO       1629358.0
    SERVICOS GERAIS               1623286.0
    A SERVICO DE DEPUTADO         1527813.0
    DEPUT.EST. A SERV.MANDATO     1309648.0
    REUNIOES TECNICAS             1261205.0
    ESCOLTA PRESOS -REMOCAO       1013571.0
    PARTICIP-SEMINARIOS/CONGR      970380.0
    SERV.DE ADMINISTR.-DAER        913763.0
    Name: Valor, dtype: float64



Aqui, cruzamos destinos e servidores associados:


```python
df.groupby(['Destino','Favorecido2'])['Valor'].sum().nlargest(20).round()
```




    Destino         Favorecido2
    PORTO ALEGRE    SILVA          1163673.0
                    SANTOS          551193.0
                    MACHADO         408099.0
                    OLIVEIRA        378335.0
    TRAMANDAI       SILVA           357969.0
    PORTO ALEGRE    PEREIRA         296562.0
                    ROSA            256169.0
    CAPAO DA CANOA  SILVA           253770.0
    PORTO ALEGRE    LIMA            245712.0
                    JUNIOR          227397.0
                    SOUZA           225926.0
    TORRES          SILVA           214180.0
    PORTO ALEGRE    RODRIGUES       207616.0
    TRAMANDAI       OLIVEIRA        193649.0
    PORTO ALEGRE    ALMEIDA         175780.0
                    MORAES          174702.0
                    SILVEIRA        171049.0
    TRAMANDAI       SANTOS          170904.0
    PORTO ALEGRE    COSTA           159793.0
    CIDREIRA        SILVA           151463.0
    Name: Valor, dtype: float64



Agora, vejamos quais foram os maiores valores reservados a determinado servidor em 2017:


```python
x = df.sort_values(by='Valor', ascending=False).loc[:10,['Orgao','Favorecido2','Destino','Motivo','Valor']]
x.head(10)
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
      <th>Orgao</th>
      <th>Favorecido2</th>
      <th>Destino</th>
      <th>Motivo</th>
      <th>Valor</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>158461</th>
      <td>SECRETARIA DE DESENVOLVIMENTO ECONOMICO, CIENC...</td>
      <td>FONTANA</td>
      <td>EXTERIOR</td>
      <td>MISSAO OFICIAL</td>
      <td>14056.90</td>
    </tr>
    <tr>
      <th>210104</th>
      <td>ASSEMBLEIA LEGISLATIVA</td>
      <td>DZIEDRICKI</td>
      <td>PARIS - FRANCA. DIARIAS ESPECIAIS DE VIAGEM</td>
      <td>DEPUT.EST. A SERV.MANDATO</td>
      <td>13800.24</td>
    </tr>
    <tr>
      <th>610</th>
      <td>ASSEMBLEIA LEGISLATIVA</td>
      <td>LOPES</td>
      <td>PARIS - FRANCA DIARIAS ESPECIAIS DE VIAGEM</td>
      <td>DEPUT.EST. A SERV.MANDATO</td>
      <td>13555.60</td>
    </tr>
    <tr>
      <th>26539</th>
      <td>ASSEMBLEIA LEGISLATIVA</td>
      <td>SIMONI</td>
      <td>Basileia - SUICA DIARIAS ESPECIAIS DE VIAGEM</td>
      <td>DEPUT.EST. A SERV.MANDATO</td>
      <td>13468.00</td>
    </tr>
    <tr>
      <th>196248</th>
      <td>SECRETARIA DO AMBIENTE E DESENVOLVIMENTO SUSTE...</td>
      <td>PELLINI</td>
      <td>EXTERIOR</td>
      <td>PARTICIPACAO-EVENTOS</td>
      <td>13019.26</td>
    </tr>
    <tr>
      <th>210715</th>
      <td>SECRETARIA DO AMBIENTE E DESENVOLVIMENTO SUSTE...</td>
      <td>MOLLMANN</td>
      <td>EXTERIOR</td>
      <td>PARTICIPACAO-EVENTOS</td>
      <td>13019.26</td>
    </tr>
    <tr>
      <th>207645</th>
      <td>SECRETARIA DO AMBIENTE E DESENVOLVIMENTO SUSTE...</td>
      <td>MEIRELLES</td>
      <td>EXTERIOR</td>
      <td>PARTICIPACAO-EVENTOS</td>
      <td>13019.26</td>
    </tr>
    <tr>
      <th>163526</th>
      <td>SECRETARIA DE DESENVOLVIMENTO ECONOMICO, CIENC...</td>
      <td>SCHUNEMANN</td>
      <td>EXTERIOR</td>
      <td>MISSAO OFICIAL</td>
      <td>12521.29</td>
    </tr>
    <tr>
      <th>198315</th>
      <td>ASSEMBLEIA LEGISLATIVA</td>
      <td>BECKER</td>
      <td>BONN - ALEMANHA. DIARIAS ESPECIAIS DE VIAGEM</td>
      <td>DEPUT.EST. A SERV.MANDATO</td>
      <td>12370.24</td>
    </tr>
    <tr>
      <th>196438</th>
      <td>SECRETARIA DE DESENVOLVIMENTO ECONOMICO, CIENC...</td>
      <td>SCHAFER</td>
      <td>EXTERIOR</td>
      <td>PARTICIPACAO-EVENTOS</td>
      <td>11593.73</td>
    </tr>
  </tbody>
</table>
</div>



A seguir, veremos quais servidores requisitaram o maior número de diárias/passagens:


```python
x = df.groupby(['Cod_Credor','Favorecido2'])
```


```python
ind = x.size().nlargest(10).reset_index()
ind
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
      <th>Cod_Credor</th>
      <th>Favorecido2</th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6137113</td>
      <td>DENKVITTS</td>
      <td>192</td>
    </tr>
    <tr>
      <th>1</th>
      <td>21180091</td>
      <td>DIAS</td>
      <td>190</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20682816</td>
      <td>LERMEN</td>
      <td>188</td>
    </tr>
    <tr>
      <th>3</th>
      <td>21164762</td>
      <td>RAMOS</td>
      <td>188</td>
    </tr>
    <tr>
      <th>4</th>
      <td>21169330</td>
      <td>RODRIGUES</td>
      <td>188</td>
    </tr>
    <tr>
      <th>5</th>
      <td>21153450</td>
      <td>MARCO</td>
      <td>181</td>
    </tr>
    <tr>
      <th>6</th>
      <td>26210711</td>
      <td>WITT</td>
      <td>180</td>
    </tr>
    <tr>
      <th>7</th>
      <td>26079267</td>
      <td>COUTO</td>
      <td>179</td>
    </tr>
    <tr>
      <th>8</th>
      <td>21171416</td>
      <td>IRENO</td>
      <td>178</td>
    </tr>
    <tr>
      <th>9</th>
      <td>21187487</td>
      <td>RAMOS</td>
      <td>178</td>
    </tr>
  </tbody>
</table>
</div>



Mais de 180 dias! Lembrando que o calendário comercial tem 252 dias no ano. Podemos cruzar a base ``ind`` gerada acima com a base de dados original ``df`` para calcular a diária média paga para estes servidores:


```python
x = ind.merge(df)
```


```python
x.groupby(['Cod_Credor','Favorecido2']).mean()['Valor']
```




    Cod_Credor  Favorecido2
    6137113     DENKVITTS      143.500000
    20682816    LERMEN          61.500000
    21153450    MARCO           88.002597
    21164762    RAMOS           61.500000
    21169330    RODRIGUES       61.500000
    21171416    IRENO           61.500000
    21180091    DIAS            61.500000
    21187487    RAMOS           61.500000
    26079267    COUTO           80.963128
    26210711    WITT            64.916389
    Name: Valor, dtype: float64




```python
x.groupby('Mes').mean()['Valor']
```




    Mes
    1     143.500000
    2      84.204194
    3      78.925000
    4      75.089494
    5      80.033904
    6      71.261429
    7      76.178333
    8      70.804043
    9      72.729108
    10     75.749390
    11     72.090350
    12     72.155921
    Name: Valor, dtype: float64



Vamos acompanhar a evolução dos gastos destes servidores, mês a mês:


```python
y = pd.DataFrame(x.groupby(['Mes','Favorecido2'])['Valor'].sum())
```


```python
y.unstack().plot(kind='line', subplots=False)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x21235266c50>




![png](/assets/images/output_34_1.png)


Até aqui exploramos os gastos em termos de valores. Vamos agora para a frequência das requisições de diárias! Os dados disponibilizados pelo Portal da Transparência informam as datas iniciais e finais associadas à cada empenho (nas variáveis ``DataInicio`` e ``DataFim``) bem como a quantidade de diárias pagas (nas variáveis ``Quantidade`` e ``QuantidadeMeia``). Meias diárias são pagas quando não há pernoite do servidor.

Com essas informações, é possível comparar a quantidade de diárias pagas com as datas registradas, bem como identificar o dia da semana.


```python
df['DataInicio'] = pd.to_datetime(df.DataInicio, format='%d/%m/%Y')
df['DataFim'] = pd.to_datetime(df.DataFim, format='%d/%m/%Y')

df['QuantosDias'] = df['DataFim'] - df['DataInicio']
df['QuantosDias'] = df.QuantosDias.dt.days
```

Vamos ver as viagens transcorridas durante finais de semana. Para isso, criarei uma função que indica se há um sabado ou domingo num intervalo entre datas, retornando verdadeiro ou falso, e depois aplicaremos essa função nos nossos dados.


```python
def sabdom(x):
    """Retorna True se há sábado ou domingo num intervalo entre datas"""
    a = pd.date_range(start=x['DataInicio'], end=x['DataFim'], freq='D').dayofweek # dias entre inicio e fim da diaria
    b = np.in1d(a, [5,6]).any() # 5 = sábado, 6 = domingo
    return b
```


```python
df.apply(sabdom, axis=1)
```

Deu erro! Isto acontece pois temos alguns valores faltantes em ``DataInicio`` e ``DataFim``. Vamos criar uma versão dessa base sem esses valores e tentar novamente:


```python
df2 = df.dropna()
x   = df2.apply(sabdom, axis=1)
df2['SabDom'] = x
```


```python
df2.SabDom.value_counts()/len(df2)
```




    False    0.798103
    True     0.201897
    Name: SabDom, dtype: float64



Segundo o cálculo acima, em 20% dos casos, a diária transcorre sábados e domingos. Podemos analisar os motivos associados com a função ``pivot_table``:


```python
tab = df2.pivot_table('Quantidade', index='Motivo', columns='SabDom', aggfunc=np.count_nonzero, margins=True, fill_value=0)
tab.head()
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
      <th>SabDom</th>
      <th>False</th>
      <th>True</th>
      <th>All</th>
    </tr>
    <tr>
      <th>Motivo</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>A SERV.COMIS.TECN.PERMAN.</th>
      <td>81</td>
      <td>42</td>
      <td>123</td>
    </tr>
    <tr>
      <th>A SERVICO DE DEPUTADO</th>
      <td>863</td>
      <td>847</td>
      <td>1710</td>
    </tr>
    <tr>
      <th>A SERVIÇO DA FASE</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>ACOMP.DE INVENT.DE B.MOV.</th>
      <td>43</td>
      <td>8</td>
      <td>51</td>
    </tr>
    <tr>
      <th>ACOMP.DE INVENT.DE ESTOQ.</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Acima, temos a contagem de dos casos para cada motivo. Podemos criar uma coluna para analisar a frequência de casos positivos em relação ao total:


```python
tab['Percent'] = tab[True]/tab['All']
tab.sort_values(by='Percent', ascending=False, inplace=True)
#tab.sort_values(by=['Percent'], axis=1, inplace=True)
tab.head(20)
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
      <th>SabDom</th>
      <th>False</th>
      <th>True</th>
      <th>All</th>
      <th>Percent</th>
    </tr>
    <tr>
      <th>Motivo</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>ATEND.REPART.FISC./EXAT.</th>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>CONTATOS COMUNIDADE</th>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>FISCALIZACAO TAXA CDO</th>
      <td>0</td>
      <td>49</td>
      <td>49</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>OPERAC. TURISMO/FRONTEIRA</th>
      <td>0</td>
      <td>30</td>
      <td>30</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>RESSARC SERVIDOR BPR DAER</th>
      <td>1</td>
      <td>7590</td>
      <td>7591</td>
      <td>0.999868</td>
    </tr>
    <tr>
      <th>OPERACAO LITORAL</th>
      <td>221</td>
      <td>10049</td>
      <td>10270</td>
      <td>0.978481</td>
    </tr>
    <tr>
      <th>MUTIRAO CARTORARIO</th>
      <td>6</td>
      <td>265</td>
      <td>271</td>
      <td>0.977860</td>
    </tr>
    <tr>
      <th>OPERACAO CANARINHO</th>
      <td>52</td>
      <td>2094</td>
      <td>2146</td>
      <td>0.975769</td>
    </tr>
    <tr>
      <th>PERFURACAO DE POCOS</th>
      <td>11</td>
      <td>207</td>
      <td>218</td>
      <td>0.949541</td>
    </tr>
    <tr>
      <th>SERV.PATRULH.RODOVIARIO</th>
      <td>138</td>
      <td>1272</td>
      <td>1410</td>
      <td>0.902128</td>
    </tr>
    <tr>
      <th>REFORCOS A OUTRA OPM</th>
      <td>913</td>
      <td>4701</td>
      <td>5614</td>
      <td>0.837371</td>
    </tr>
    <tr>
      <th>APOIO EQUIPES FISCALIZ</th>
      <td>10</td>
      <td>51</td>
      <td>61</td>
      <td>0.836066</td>
    </tr>
    <tr>
      <th>TRANSF.P/NECESSID.SERVICO</th>
      <td>9</td>
      <td>21</td>
      <td>30</td>
      <td>0.700000</td>
    </tr>
    <tr>
      <th>FISC. TRANSITO DE MERCAD.</th>
      <td>6</td>
      <td>14</td>
      <td>20</td>
      <td>0.700000</td>
    </tr>
    <tr>
      <th>REF.A EVENTOS ESPEC. - BM</th>
      <td>158</td>
      <td>365</td>
      <td>523</td>
      <td>0.697897</td>
    </tr>
    <tr>
      <th>INSPECAO SANITARIA/ANIMAL</th>
      <td>7</td>
      <td>13</td>
      <td>20</td>
      <td>0.650000</td>
    </tr>
    <tr>
      <th>FISCALIZ/PATRULHA AMBIENT</th>
      <td>3</td>
      <td>5</td>
      <td>8</td>
      <td>0.625000</td>
    </tr>
    <tr>
      <th>COMBATE AO ABIGEATO</th>
      <td>23</td>
      <td>35</td>
      <td>58</td>
      <td>0.603448</td>
    </tr>
    <tr>
      <th>SUBSTITUICOES A FUNCION.</th>
      <td>150</td>
      <td>222</td>
      <td>372</td>
      <td>0.596774</td>
    </tr>
    <tr>
      <th>ESTAGIOS</th>
      <td>3</td>
      <td>4</td>
      <td>7</td>
      <td>0.571429</td>
    </tr>
  </tbody>
</table>
</div>



Portanto, vemos que a maioria das diárias de fim de semana está de fato ligada à atividades de campo de servidores.

A tabela a seguir faz a tabulação cruzada entre número de diárias pagas e dias transcorridos de viagem, com o uso da função ``pivot_table``:


```python
tab = df.pivot_table(index='Quantidade', columns='QuantosDias', aggfunc=np.count_nonzero)
tab.head(10)
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
      <th colspan="10" halign="left">Cod_Credor</th>
      <th>...</th>
      <th colspan="10" halign="left">Valor</th>
    </tr>
    <tr>
      <th>QuantosDias</th>
      <th>0.0</th>
      <th>1.0</th>
      <th>2.0</th>
      <th>3.0</th>
      <th>4.0</th>
      <th>5.0</th>
      <th>6.0</th>
      <th>7.0</th>
      <th>8.0</th>
      <th>9.0</th>
      <th>...</th>
      <th>61.0</th>
      <th>69.0</th>
      <th>77.0</th>
      <th>92.0</th>
      <th>96.0</th>
      <th>154.0</th>
      <th>168.0</th>
      <th>273.0</th>
      <th>304.0</th>
      <th>340.0</th>
    </tr>
    <tr>
      <th>Quantidade</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
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
      <th>0</th>
      <td>153006.0</td>
      <td>4295.0</td>
      <td>1444.0</td>
      <td>710.0</td>
      <td>603.0</td>
      <td>68.0</td>
      <td>41.0</td>
      <td>30.0</td>
      <td>42.0</td>
      <td>23.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>725.0</td>
      <td>39429.0</td>
      <td>336.0</td>
      <td>58.0</td>
      <td>54.0</td>
      <td>7.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>19.0</td>
      <td>189.0</td>
      <td>15437.0</td>
      <td>194.0</td>
      <td>122.0</td>
      <td>8.0</td>
      <td>NaN</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17.0</td>
      <td>6.0</td>
      <td>76.0</td>
      <td>6487.0</td>
      <td>177.0</td>
      <td>7.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>2.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>608.0</td>
      <td>6745.0</td>
      <td>25.0</td>
      <td>4.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>3.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>4.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>50.0</td>
      <td>2516.0</td>
      <td>4.0</td>
      <td>7.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>4.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>372.0</td>
      <td>507.0</td>
      <td>1.0</td>
      <td>16.0</td>
      <td>10.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>136.0</td>
      <td>379.0</td>
      <td>4.0</td>
      <td>10.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9.0</td>
      <td>82.0</td>
      <td>140.0</td>
      <td>1.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>116.0</td>
      <td>174.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>10 rows × 1134 columns</p>
</div>



Na tabela acima podemos detectar alguns pontos fora da curva, como por exemplo, várias diárias pagas para atividades de um dia, e vice-versa.


```python
tab = df.query('(Quantidade > 1 ) & (QuantosDias == 1)').loc[:,['Favorecido2','Quantidade','Valor']]
tab.head(20)
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
      <th>Favorecido2</th>
      <th>Quantidade</th>
      <th>Valor</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2698</th>
      <td>ROLIM</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>2746</th>
      <td>SENNA</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>6363</th>
      <td>SILVA</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>6958</th>
      <td>PEREIRA</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>6996</th>
      <td>MELLO</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>7041</th>
      <td>WEBER</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>7177</th>
      <td>DIAS</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>7234</th>
      <td>MOURA</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>7380</th>
      <td>CARVALHO</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>8409</th>
      <td>SOUTO</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>11116</th>
      <td>POZZOBON</td>
      <td>2</td>
      <td>358.75</td>
    </tr>
    <tr>
      <th>11125</th>
      <td>SILVA</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>11173</th>
      <td>MACHADO</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>14233</th>
      <td>MARTINS</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>14274</th>
      <td>GUIMARAES</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>14338</th>
      <td>ROCHA</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>14341</th>
      <td>RITTER</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>14520</th>
      <td>SOUZA</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>14976</th>
      <td>CAMPOS</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>15600</th>
      <td>MACHADO</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
  </tbody>
</table>
</div>



Vamos organizar por quantidade de diárias para entender melhor:


```python
tab = tab.sort_values(by='Quantidade', ascending=False)
tab.head(20)
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
      <th>Favorecido2</th>
      <th>Quantidade</th>
      <th>Valor</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>80827</th>
      <td>MOIANO</td>
      <td>29</td>
      <td>-184.49</td>
    </tr>
    <tr>
      <th>28505</th>
      <td>FLORES</td>
      <td>18</td>
      <td>-738.00</td>
    </tr>
    <tr>
      <th>257107</th>
      <td>BRUNHAUSER</td>
      <td>14</td>
      <td>-122.99</td>
    </tr>
    <tr>
      <th>168131</th>
      <td>SILVA</td>
      <td>14</td>
      <td>-150.32</td>
    </tr>
    <tr>
      <th>190620</th>
      <td>SERPA</td>
      <td>5</td>
      <td>-123.00</td>
    </tr>
    <tr>
      <th>27416</th>
      <td>PERIN</td>
      <td>4</td>
      <td>526.12</td>
    </tr>
    <tr>
      <th>222709</th>
      <td>SOBROSA</td>
      <td>3</td>
      <td>-245.98</td>
    </tr>
    <tr>
      <th>105213</th>
      <td>PALADINI</td>
      <td>3</td>
      <td>2234.31</td>
    </tr>
    <tr>
      <th>155972</th>
      <td>SOUZA</td>
      <td>3</td>
      <td>-300.66</td>
    </tr>
    <tr>
      <th>41910</th>
      <td>MATTAR</td>
      <td>3</td>
      <td>698.75</td>
    </tr>
    <tr>
      <th>24447</th>
      <td>MACHADO</td>
      <td>3</td>
      <td>512.48</td>
    </tr>
    <tr>
      <th>43525</th>
      <td>LUZ</td>
      <td>3</td>
      <td>698.75</td>
    </tr>
    <tr>
      <th>226281</th>
      <td>FIORAVANSO</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>100498</th>
      <td>RESCHKE</td>
      <td>2</td>
      <td>1560.00</td>
    </tr>
    <tr>
      <th>101771</th>
      <td>MEIRELLES</td>
      <td>2</td>
      <td>1560.00</td>
    </tr>
    <tr>
      <th>102686</th>
      <td>ROSSI</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>102695</th>
      <td>VARGAS</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>246512</th>
      <td>ALIARDI</td>
      <td>2</td>
      <td>245.98</td>
    </tr>
    <tr>
      <th>111185</th>
      <td>BIANCHI</td>
      <td>2</td>
      <td>540.00</td>
    </tr>
    <tr>
      <th>112748</th>
      <td>PIRES</td>
      <td>2</td>
      <td>123.00</td>
    </tr>
  </tbody>
</table>
</div>



Tudo certo! Aparentemente, a maioria dos registros trata de ressarcimentos.

Por enquanto, vamos parando por aqui! Entretanto, muito mais pode ser feito, e este exercício mostrou apenas algumas das capacidades de manipulação e cruzamento de dados com o Python, através do pacote ``pandas``.
