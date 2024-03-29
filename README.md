<div align="center">
    <picture>
    <source media="(prefers-color-scheme: dark)" srcset="images/ebac-white-letter.png">
    <source media="(prefers-color-scheme: light)" srcset="images/ebac-black-letter.png">
    </picture>
</div>

# Análise de Risco de Crédito: Analisando Clientes Inadimplentes
## <img loading="lazy" height="35" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/python/python-original.svg"/>Python: Projeto Final

#### Sara Mariana

## Tópicos
   1. [Identificação do problema](#identificacao-do-problema)
   2. [Exploração de dados](#exploracao-de-dados)
      * [Estrutura e Schema](#estrutura-de-schema)
      * [Dados faltantes](#dados-faltantes)
   3. [Transformação e limpeza de dados](#transformacao-e-limpeza-de-dados)
      * [Remoção de dados faltantes](#remocao-de-dados-faltantes)
   4. [Visualização de dados](#visualizacao-de-dados)
      * [Quantidade de Iterações nos Últimos 12 meses](#quantidade-de-iteracoes-nos-ultimos-12-meses)
      * [Quantidade de Meses Inativo nos Últimos 12 Meses](#quantidade-de-meses-inativo-nos-ultimos-12-meses)
      * [Quantidade de Meses Inativo x Quantidade de Iterações](#quantidade-de-meses-inativo-x-quantidade-de-iteracoes)
   5. [Resumo dos insights gerados](#resumo-dos-insights-gerados)


## Identificação do problema
O desafio apresentado por meio desta análise é compreender os fatores que levam os clientes de um banco a se tornarem ou permanecerem inadimplentes. Nossa tarefa envolve uma investigação detalhada e rigorosa dos dados disponíveis, a criação de visualizações que revelam insights cruciais e uma profunda reflexão sobre como essas informações estão relacionadas à inadimplência.. Podemos perceber que será necessário uma análise lógica, vizualizações críticas das informações fornecidas e uma reflexão de quais dados intereferem ou não na inadimplencia.

## Exploração de dados
```
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd

df = pd.read_csv('https://raw.githubusercontent.com/andre-marcos-perez/ebac-course-utils/develop/dataset/credito.csv', na_values='na')
```

### Estrutura e Schema
#### Insight
Na primeira visão que temos podemos notar que na nossa base de dados há poucos dados de clientes inadimplentes de um total de mais de 10mil clientes temos menos de 20% para trabalhar.
```
df[df['default'] == 1].shape
```
```
qtd_total, _ = df.shape
qtd_adimplentes, _ = df[df['default'] == 0].shape
qtd_inadimplentes, _ = df[df['default'] == 1].shape

print(f"{qtd_adimplentes} são clientes adimplentes e a proporcão é de {round(100 * qtd_adimplentes / qtd_total, 2)}%")
print(f"{qtd_inadimplentes} são clientes inadimplentes e  proporcão é de {round(100 * qtd_inadimplentes / qtd_total, 2)}%")
```

#### Insight
Observando dos tipos de dados, temos duas colunas com tipos de dados diferentes da proposta de trabalho, a primeira é a coluna de limite_credito e a outra é valor_transacoes_12m, as duas estão como 'string', porém para trabalharmos com essas métricas sera necessário uma transfomação nesses dados. No nosso caso como a análise proposta não levará em consideração esses dados não realizaremos essa transformação.
```
df.select_dtypes('object').describe().transpose()
```
```
df.drop('id', axis=1).select_dtypes('number').describe().transpose()
```

### Dados faltantes
#### Insight
Nessa segunda observação, percebemos que temos uma quantidade relativamente pequena de dados faltantes e que numa visão ampla a exclusão dessas linhas que contém informações faltantes não iram afetar diretamente a nossa solução com a váriavel dependente: "default", pois a maioria da linhas que faltam infomações são de clientes adimplente.
```
df.isna().any()
```
```
def stats_dados_faltantes(df: pd.DataFrame) -> None:

    stats_dados_faltantes = []
    for col in df.columns:
        if df[col].isna().any():
            qtd, _ = df[df[col].isna()].shape
            total, _ = df.shape
            dict_dados_faltantes = {col: {'quantidade': qtd, "porcentagem_da_coluna": round(100 * qtd/total, 2)}}
            stats_dados_faltantes.append(dict_dados_faltantes)

    for stat in stats_dados_faltantes:
        print(stat)
```
```
stats_dados_faltantes(df=df)
```
```
stats_dados_faltantes(df=df[df['default'] == 0])
```
```
stats_dados_faltantes(df=df[df['default'] == 1])
```

## Transformação e limpeza de dados
### Remoção de dados faltantes
#### Insight
Após limpeza da nossa base de dados, conseguimos ver claramente que após a exclusão dos dados a interferência na várivel reposta foi muito baixa.
```
df.dropna(inplace=True)
```
```
qtd_total_novo, _ = df.shape
qtd_adimplentes_novo, _ = df[df['default'] == 0].shape
qtd_inadimplentes_novo, _ = df[df['default'] == 1].shape

print(f"A proporcão adimplentes ativos é de {round(100 * qtd_adimplentes / qtd_total, 2)}%")
print(f"A nova proporcão de clientes adimplentes é de {round(100 * qtd_adimplentes_novo / qtd_total_novo, 2)}%")
print("")
print(f"A proporcão clientes inadimplentes é de {round(100 * qtd_inadimplentes / qtd_total, 2)}%")
print(f"A nova proporcão de clientes inadimplentes é de {round(100 * qtd_inadimplentes_novo / qtd_total_novo, 2)}%")
```

## Visualização de dados
```
sns.set_style("whitegrid")

df_adimplente = df[df['default'] == 0]
df_inadimplente = df[df['default'] == 1]
```

### Quantidade de Iterações nos Últimos 12 Meses
#### Insight
Observando atentamente esse gráfico percebmos que as Iterações do Banco com o cliente nos últimos 12 meses traz dados relavantes para a nossa análise, percebemos que cliente Inadimplentes estão em maior concentração á partir da 3ª iteração, ao contrário de clientes adimplentes que se concetram na 2ª iteração com o banco. Conseguimos validar também que clientes inadimplentes são os únicos que acabam tendo mais iterações com banco e são os únicos com mais de seis iterações.
```
coluna = 'iteracoes_12m'
titulos = ['Qtd. de Iterações no Último Ano', 'Qtd. de Iterações no Último Ano de Adimplentes', 'Qtd. de Transações no Último Ano de Inadimplentes']

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

### Quantidade de Meses Inativo nos Últimos 12 Meses
#### Insight
Apesar de termos uma constância de inativade de inadimplentes e adimplentes a decisão de coloca-lo nessa análise foi para conseguirmos "traçar" um comparativo com a quantidade de iterações com o banco, mas com uma visão geral conseguimos perceber que clientes inadimplentes dificilmente ficam mais de quatro meses inativos e que embora a maioria dos clientes adimplentes também ficarem inativos por três meses, a proporção de clientes inadimplentes que permanecem inativos por dois meses em relação a três meses é significativamente maior do que a proporção correspondente entre os clientes adimplentes.
```
coluna = 'meses_inativo_12m'
titulos = ['Qntd. de Meses Inativo no Último Ano', 'Qntd. de Meses Inativo no Último Ano Adimplentes', 'Qntd. Meses Inativo no Último Ano de Inadimplentes']

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

### Quantidade de Meses Inativo x Quantidade de Iterações
#### Insight
Agora com os dois gráficos lado a lado conseguimos identificar que a a maioria dos clientes inadimplentes tem uma tendência maior a inatividade bancária dentro do período de 3 a 6 meses e que as iterações com o banco são relativamente maior do que adimplentes.
```
# Criar o gráfico de linha com x e y invertidos
plt.figure(figsize=(13, 5))  # Tamanho da figura

sns.lineplot(x='meses_inativo_12m', y='iteracoes_12m', hue='default', data=df)

# Definir os rótulos dos eixos e o título
plt.title('Relação entre Iterações 12M, Quantidade de Transações 12M e Default')
plt.xlabel('meses_inativo_12m')
plt.ylabel('iteracoes_12m')

# Mostrar o gráfico
plt.show()
```

## Resumo dos Insights Gerados
Com a análise proposta a partir dos dados de inatividade de iterações conseguimos tirar algumas conclusões significativas sobre a identificação de clientes inadimplentes:
- Clientes inadimplentes tem uma tendência a receberam mais iterações com o banco e que isso pode sinalizar um alerta para os colaboradores que acompanham e assistem esses cliente.
- Apenas a informação isolada de inativade sem a iteração dos clientes conseguimos perceber que o banco tem uma constância de inatvidade concentrada de 2 á 3 meses, porém pode-se sinalizar que clientes inativos e com altas iterações com o banco tem uma tendência maior a não quitarem seus débitos

**Sugestão de Decisão:**

- Campanhas de incentivo; com a análise de inatividade o banco pode propor descontos e campanhas que incentivem seus clientes á partir do seu segundo mês de inativade á adiantarem suas contas evitando assim transtornos futuros com a falta de dinheiro.
- Seria interessante o banco **identificar qual o tipo de iteração está realizanddo** com o cliente para que assim a base de dados junto com a análise ficassem mais acertivas no que diz respeito a que tipo de iteração está sendo realizando pelo colaborador, IA... E se essa iteração é feita por atrasos, cancelamentos, duvídas etc.
- Clientes com baixa atividade e altas iterações podem ser categorizados como risco de inadimplência, assim a equipe responsável por cobranças e liberação de crédito podem tomar decisões mais acertivas e com menos riscios.

### Sara Mariana
Busquei organizar e tratar esta análise de maneira lógica e espero que tenha ficado no mínimo corente com a realidade dos dados. Agradeço a leitura e recebo de braços abertos sugestões de melhorias!!
