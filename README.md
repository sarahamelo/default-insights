# Default Insights
Autoridade: Sarah Melo

Data criada: 13.03.24

--- 

<h4>INTRODUÇÃO DO PROBLEMA</h4>

Baseado nos dados que estão em formado CSV, ou no seguinte <a href="https://raw.githubusercontent.com/andre-marcos-perez/ebac-course-utils/develop/dataset/credito.csv">link</a> , vamos analisar os dados dos clientes de uma instituição financeira afim de analisar e explicar o comportamento de uma coluna: a default.

Essa coluna nos diz se o cliente é inadimplente (default=1) ou não (default=0.

Para isso, devemos analisar as outras colunas: salário, escolaridade, movimentação financeira etc. E com isso, entendermos o porquê de um cliente deixar de pagar suas dívidas.

---


<h4>PREPARO DO AMBIENTE</h4>

Comecemos lendo as 50 primeiras linhas do arquivo:

``` python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

sns.set_style("whitegrid")

df = pd.read_csv('https://raw.githubusercontent.com/andre-marcos-perez/ebac-course-utils/develop/dataset/credito.csv', na_values='na')

df.read(n=50)
```

---

<h4>EXPLORANDO OS DADOS</h4>

Podemos analisar de 10.128 linhas de dados, quantos clientes são inadimplentes e quantos não são:

``` python
df[df['default']==0].shape #adimplentes
df[df['default']==1].shape #inadimplentes
```
---

<h4>LIMPANDO E TRANSFORMANDO OS DADOS</h4>

Ao analisar os tipos de dados, podemos notar que as colunas limite_credito e valor_transacoes_12m estão sendo interpretadas como colunas categóricas, ou seja, como objetos invés de floats.

Para a resolução desse problema, podemos criar uma função lambda para limpar esses dados e aplicá-la nas colunas que desejamos:

``` python
fn = lambda valor: float(valor.replace(".", "").replace(",", "."))
df['valor_transacoes_12m'] = df['valor_transacoes_12m'].apply(fn)
df['limite_credito'] = df['limite_credito'].apply(fn)
```

Há dados faltando nosso arquivo, podemos notar isso com o código:

``` python
def stats_dados_faltantes(df: pd.DataFrame) -> None:

  stats_dados_faltantes = []
  for col in df.columns:
    if df[col].isna().any():
      qtd, _ = df[df[col].isna()].shape
      total, _ = df.shape
      dict_dados_faltantes = {col: {'quantidade': qtd, "porcentagem": round(100 * qtd/total, 2)}}
      stats_dados_faltantes.append(dict_dados_faltantes)

  for stat in stats_dados_faltantes:
    print(stat)
```

Mas com o pandas, nós podemos resolver isso:

``` python
qtd_total_novo, _ = df.shape
qtd_adimplentes_novo, _ = df[df['default'] == 0].shape
qtd_inadimplentes_novo, _ = df[df['default'] == 1].shape
```

Vamos ver como ficou:

``` python
print(f"A proporcão adimplentes ativos é de {round(100 * qtd_adimplentes / qtd_total, 2)}%")
print(f"A nova proporcão de clientes adimplentes é de {round(100 * qtd_adimplentes_novo / qtd_total_novo, 2)}%")
print("")
print(f"A proporcão clientes inadimplentes é de {round(100 * qtd_inadimplentes / qtd_total, 2)}%")
print(f"A nova proporcão de clientes inadimplentes é de {round(100 * qtd_inadimplentes_novo / qtd_total_novo, 2)}%")
```

--- 

<h4>VISUALIZAÇÃO DE DADOS</h4>

Para entender qual o fator que leva um cliente a ser inadimplente, vamos visualizar e analisar para correlacionarmos com o nosso objetivo. Vamos sempre estar comparando os dados de cliente adimplentes e os de clientes inadimplentes. Para isso, vamos separar os dados de ambos:

``` python
df_adimplente = df[df['default'] == 0] #Dados de clientes adimplentes 
df_inadimplente = df[df['default'] == 1] #Dados de clientes inadimplentes
```

Com os dados separados, podemos começar nossas analises. Vamos começar com a comparação de atributos categóricos e, depois, analisar atributos numéricos. Vejamos então a comparação de *escolaridade* entre esses dois tipos de clientes.

``` python
coluna = 'escolaridade'
titulos = ['Escolaridade dos Clientes', 'Escolaridade dos Clientes Adimplentes', 'Escolaridade dos Clientes Inadimplentes']

eixo = 0
max_y = 0
max = df.select_dtypes('object').describe()[coluna]['freq'] * 1.1

figura, eixos = plt.subplots(1,3, figsize=(20, 5), sharex=True)

for dataframe in [df, df_adimplente, df_inadimplente]:

  df_to_plot = dataframe[coluna].value_counts().to_frame()
  df_to_plot.rename(columns={coluna: 'frequencia_absoluta'}, inplace=True)
  df_to_plot[coluna] = df_to_plot.index
  df_to_plot.sort_values(by=[coluna], inplace=True)
  df_to_plot.sort_values(by=[coluna])

  f = sns.barplot(x=df_to_plot[coluna], y=df_to_plot['frequencia_absoluta'], ax=eixos[eixo])
  f.set(title=titulos[eixo], xlabel=coluna.capitalize(), ylabel='Frequência Absoluta')
  f.set_xticklabels(labels=f.get_xticklabels(), rotation=90)

  _, max_y_f = f.get_ylim()
  max_y = max_y_f if max_y_f > max_y else max_y
  f.set(ylim=(0, max_y))

  eixo += 1

figura.show()
```

Analisando o gráfico, podemos ver que os três gráficos são muito parecidos. Como não há uma diferença grande entre um gráfico e outro, podemos pensar que não seja essa a diferença crucial entre os cliente adimplenetes e os inadiplentes.

Vejamos então com o *salário anual* dos clientes:

``` python
coluna = 'salario_anual'
titulos = ['Salário Anual dos Clientes', 'Salário Anual dos Clientes Adimplentes', 'Salário Anual dos Clientes Inadimplentes']

eixo = 0
max_y = 0
figura, eixos = plt.subplots(1,3, figsize=(20, 5), sharex=True)

for dataframe in [df, df_adimplente, df_inadimplente]:

  df_to_plot = dataframe[coluna].value_counts().to_frame()
  df_to_plot.rename(columns={coluna: 'frequencia_absoluta'}, inplace=True)
  df_to_plot[coluna] = df_to_plot.index
  df_to_plot.reset_index(inplace=True, drop=True)
  df_to_plot.sort_values(by=[coluna], inplace=True)

  f = sns.barplot(x=df_to_plot[coluna], y=df_to_plot['frequencia_absoluta'], ax=eixos[eixo])
  f.set(title=titulos[eixo], xlabel=coluna.capitalize(), ylabel='Frequência Absoluta')
  f.set_xticklabels(labels=f.get_xticklabels(), rotation=90)
  _, max_y_f = f.get_ylim()
  max_y = max_y_f if max_y_f > max_y else max_y
  f.set(ylim=(0, max_y))
  eixo += 1

figura.show()
```
![image](https://github.com/sarahamelo/default-insights/assets/128179357/f9f43aaa-4431-4fbf-831d-88ca7b12855d)


Novamente, os três gráficos não se modificaram tanto, ou seja, não é o salário anual dos clientes que interfere em um cliente ser ou não adimplente.

Dito isso, vamos analisar agora os atributos numéricos. Vamos começar pela *quantidade de transações nos últimos 12 meses*:

``` python
coluna = 'qtd_transacoes_12m'
titulos = ['Qtd. de Transações no Último Ano', 'Qtd. de Transações no Último Ano de Adimplentes', 'Qtd. de Transações no Último Ano de Inadimplentes']

eixo = 0
max_y = 0
figura, eixos = plt.subplots(1,3, figsize=(20, 5), sharex=True)

for dataframe in [df, df_adimplente, df_inadimplente]:

  f = sns.histplot(x=coluna, data=dataframe, stat='count', ax=eixos[eixo])
  f.set(title=titulos[eixo], xlabel=coluna.capitalize(), ylabel='Frequência Absoluta')

  _, max_y_f = f.get_ylim()
  max_y = max_y_f if max_y_f > max_y else max_y
  f.set(ylim=(0, max_y))

  eixo += 1

figura.show()
```
![image](https://github.com/sarahamelo/default-insights/assets/128179357/c07c666b-77bb-4e32-b480-42e0f8e5e016)


Nesses gráficos tem algo curioso: os clientes inadimplentes tem uma maior porcentagem de transações de 40 a 60 transações dentre 12 meses. Ou seja, os clientes que tem essa quantidade de transações em um ano tem maior chance de virarem inadimplentes.

Vamos analisar o valor dessas transações e tentar descobrir um pouco mais:

``` python
coluna = 'valor_transacoes_12m'
titulos = ['Valor das Transações no Último Ano', 'Valor das Transações no Último Ano de Adimplentes', 'Valor das Transações no Último Ano de Inadimplentes']

eixo = 0
max_y = 0
figura, eixos = plt.subplots(1,3, figsize=(20, 5), sharex=True)

for dataframe in [df, df_adimplente, df_inadimplente]:

  f = sns.histplot(x=coluna, data=dataframe, stat='count', ax=eixos[eixo])
  f.set(title=titulos[eixo], xlabel=coluna.capitalize(), ylabel='Frequência Absoluta')

  _, max_y_f = f.get_ylim()
  max_y = max_y_f if max_y_f > max_y else max_y
  f.set(ylim=(0, max_y))

  eixo += 1

figura.show()
```

![image](https://github.com/sarahamelo/default-insights/assets/128179357/e8b01747-cfb4-4269-8bf5-63f358031be2)


Vejamos que no primeiro gráfico, que representa TODOS os clientes, tem dois grandes picos:

- Valores que vão são um pouco maiores que 0 e menores que 2.500
- Valores um pouco menores que 5.000
- Observando o gráfico de clientes adimplentes, vemos que o menor pico é o de valores muito próximos a 2.500. Porém vendo o gráfico de clientes inadimplentes, vemos o contrário: o maior pico deles são as transações perto de 2.500.

Podemos juntar esses dois dados (quantidade de transações no último ano e valor das transações no último ano) e visualizar melhor o que acabamos de analisar.

``` python
f = sns.relplot(x='valor_transacoes_12m', y='qtd_transacoes_12m', data=df, hue='default')
_ = f.set(
    title='Relação entre Valor e Quantidade de Transações no Último Ano',
    xlabel='Valor das Transações no Último Ano',
    ylabel='Quantidade das Transações no Último Ano'
  )
```
![image](https://github.com/sarahamelo/default-insights/assets/128179357/0f18d3d2-6cab-4cda-8428-8ceb26281e36)


---

<h4>RESUMO DOS INSIGHTS:</h4>

Analisamos dados de clientes de uma instituição financeira afim de acharmos o quê faz um cliente virar inadimplente. Para isso analisamos diversos dados e dividimos eles em três gráficos: dados de todos clientes, dados de clientes adimplentes e dados de clientes inadimplentes.

Analisamos escolaridade, salário anual, quantidade de transações nos últimos 12 anos e o valor das tansações nos últimos 12 meses. E as duas primeiras análises não nos levaram a uma resposta boa.

Porém, vimos que a quantidade de trasações para clientes inadimplentes varia de 40 a 60 transações no último ano e, observando os valores, os clienets inadimplentes tendem a transacionar valores próximos a 2.500.

Ou seja, a instuição financeira que atende esses clientes deve redobrar a atenção para clientes que transacionam valores próximos ao citado 40 a 60 vezes por ano. Pois eles tendem a deixar de pagar suas dívidas.

--- 

Last modified on: 13.03.24
