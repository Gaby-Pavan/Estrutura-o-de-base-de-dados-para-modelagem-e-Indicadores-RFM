# Estruturacao-de-base-de-dados-para-modelagem-e-Indicadores-RFM
Data Cleaning e Data Wrangling para estruturar uma base de dados de um e-commerce para modelagem e levantamento de indicadores de RFM (Regency. frequency, monetary) 

## CONTEXTO
Uma e-commerce quer levantar os indicadores de recência, frequência e ticket médio dos seus clientes. A partir de uma base de dados .csv o objetivo do projeto era
construir um código em Python com um output contendo a identificação do cliente e a métrica RFM. 
A base de dados contém informações de compras de 37 países, com a identificação do cliente e os dados da compra.

### Importando as Bibliotecas
```
import pandas as pd
import numpy as np

import seaborn as sns
import matplotlib.pyplot as plt
from sklearn.preprocessing import scale

```
### Carregando e lendo a base de dados

```
df = pd.read_csv('/content/Data - data (2).csv.csv')
df.head()

```
Depois foi feita a verificação e tratamento de dados nulos e duplicados além da alteração dos tipos de dados para os formatos desejados

### Tratamento de Outliers
Foi feito um filtro para que o dataset mostrasse apenas valores acima de 0 em preço unitario e quantidade. Posteriormente a visualização dos outliers utilizando o
boxplot e tratamento dos outliers (foi considerado quatidade de compra acima de 10.000 e preço unitario acima de 5.000)

```
df[(df['UnitPrice'] <= 0) | (df['Quantity'] <= 0)]
df = df[~((df['UnitPrice'] <= 0) | (df['Quantity'] <= 0))]

sns.boxplot(df['UnitPrice'])

```

![image](https://github.com/user-attachments/assets/c89af0c3-7e61-48b4-b053-02a1e6c2937a)

```

sns.boxplot(df['Quantity'])

```

![image](https://github.com/user-attachments/assets/9176bcf5-e266-44e3-8bff-6b4fbe3b45c8)

```

# Tratamento de Outliers (qtd compra acima de 10.000 e unitprice acima de 5.000)
df[(df['Quantity']>10000) | (df['UnitPrice']> 5000)]

# Excluindo os Outliers do dataframe
df = df[~((df['Quantity']>10000) | (df['UnitPrice']> 5000))]

```
### Criação de coluna adicional e plot de gráficos para análise 

```
# Criando coluna adicional de Preço Total
df['Preço_total'] = df['UnitPrice'] * df['Quantity']

```
```
# TOP 10 paises com maior valor em vendas
top_paises = df.groupby('Country')['Preço_total'].sum().sort_values(ascending=False).head(10)

# TOP 10 produtos mais vendidos
top_produtos = df.groupby('Description')['Preço_total'].sum().sort_values(ascending=False).head(10)
top_produtos
```
![image](https://github.com/user-attachments/assets/e39b8a51-e41b-4126-8000-3510b6772bf4)

```
# Valor de venda total por mes
vendas_mes = df.groupby(df['InvoiceDate'].dt.to_period('M'))['Preço_total'].sum().reset_index()
vendas_mes
```
![image](https://github.com/user-attachments/assets/00722df5-8349-4814-ade1-255eb93c83cb)

```
# vendas total por mes e por top 10 paises
df_top_paises = df[df['Country'].isin(top_paises.index)]
vendas_mes_paises = df_top_paises.groupby([df['InvoiceDate'].dt.to_period('M'), 'Country'])['Preço_total'].sum().reset_index()
vendas_mes_paises
```
![image](https://github.com/user-attachments/assets/261c854a-2577-412c-bbd5-38fe6935bf66)

### Cálculo das Métricas de RFM

Calculando a data da última compra do dataset, para utiilizarmos no cálculo de recência
```
date_max = df['InvoiceDate'].max()
date_max

```
```
final_output = df.groupby('CustomerID').agg(
    Regency=('InvoiceDate', lambda x: (date_max - x.max()).days),
    Frequency=('InvoiceNo', 'count'),
    Avg_ticket=('Preço_total', 'mean')).reset_index()

```
Output final do código:

![image](https://github.com/user-attachments/assets/b36cca52-dcb6-4311-8aa4-cafdcc0c5ffd)















