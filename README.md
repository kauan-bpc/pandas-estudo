pip install pandas numpy openpyxl ``bash

# Primeiros passos

## 1. Importando as bibliotecas

Primeiro para começar deve se utilizar o comando import para trazer as bibliotecas que serão utilizadas:

```python
import pandas as pd
import numpy as np
```

---

## 2. Carregando os dados

Depois, antes de começar as etapas de tratamento de dados e visualização é necessário identificar o tipo de arquivo que será "lido" nesse caso são um .csv e um .xlsx, logo após identificar você deve atribuir a uma variável o comando pd.read... para tornar mais prático na hora de utilizar o display:

```python
df_vendas = pd.read_csv('vendas_tech.csv')
df_gerentes = pd.read_excel('gerentes_lojas.xlsx')
```

---

## 3. Primeira visualização

Após fazer isso você vai utilizar o display para ter essa primeira visualização dos dados:

```python
display(df_vendas)
```

Um comando importante nessa pré avaliação dos dados é o `display(df_exemplo.info())` que retorna uma lista com informações do banco como quantas colunas são, quais, seu tipo de dado e a contagem de não nulos:

```python
display(df_vendas.info())
```

---

## 4. Tratamento dos dados

### 4.1 Criando uma cópia

Agora partindo para o tratamento em si dos dados, primeiramente é criado uma cópia da tabela principal para poder ter essa distinção e não ter que ficar carregando a tabela que está lá em cima a todo momento:

```python
df_copia = df_vendas.copy()
```

### 4.2 Limpeza do dataset

Agora não se tem uma ordem exata, isso vai depender do banco analisado, porém tem uma série de coisas a serem observadas: as colunas, os nulos, os tipos de dados, a padronização dos dados e as duplicatas:

```python
# Faz com que a cópia do dataset drope a coluna especificada
df_copia = df_copia.drop(columns=['Data_Base'])

# Preenche os nulos da coluna loja com Online
df_copia['Loja'] = df_copia['Loja'].fillna('Online')

# Modifica o formato da coluna Data e padroniza para US format
df_copia['Data'] = pd.to_datetime(df_copia['Data'], format='%Y-%m-%d')

# O strip elimina espaços vazios extras desnecessários e o title faz com que a primeira letra seja maiuscula
df_copia['Loja'] = df_copia['Loja'].str.strip()
df_copia['Loja'] = df_copia['Loja'].str.title()

# Dropa os pedidos com ID duplicado
df_copia = df_copia.drop_duplicates(subset=['ID_Pedido'])
```

---

## 5. Criação de novas colunas

Agora a parte de criar novas colunas, as operações são bem simples e intuitivas,nesse caso será mostrado três formas diferentes para se criar uma coluna nova:

```python
# Coluna de Faturamento
df_tratadov['Faturamento'] = df_tratadov['Qtd'] * df_tratadov['Preco_Unitario']

# forma de venda (o np.where() é uma função do numpy que permite criar uma nova coluna com base em uma condição. A sintaxe é: np.where(condição, valor_se_verdadeiro, valor_se_falso))
df_tratadov['Forma_Venda'] = np.where(df_tratadov['Loja'] == 'Online', 'Online', 'Presencial')

# regiao 
display(df_tratadov['Loja'].unique())

dict_regioes = {
    'São Paulo': 'Sudeste',
    'Belo Horizonte': 'Sudeste',
    'Online': 'Online',
    'Rio De Janeiro': 'Sudeste',
    'Salvador': 'Nordeste',
    'Recife': 'Nordeste',
    'Curitiba': 'Sul',
    'Porto Alegre': 'Sul'
}
df_tratadov['Regiao'] = df_tratadov['Loja'].map(dict_regioes)
```

Falando um pouco do que foi feito nessa parte da criação de novas colunas, a primeira forma a mais simples apenas colocando o nome da nova coluna e a operação necessária para criar ela, a segunda forma foi utilizado o np.where que é uma função do numpy com sintaxe parecida de um if else comum (condição, valor se verdadeiro, valor se falso), e tem a terceira forma que é um pouquinho mais complexa mas de certa forma continua simples, nesse terceiro caso foi criado um dict com as cidades presentes no banco e atribuindo sua respectiva região para depois chamar esse dict numa função .map que permite criar uma nova coluna com base num dict ou em alguns casos até mesmo utilizando outra função, foi feito dessa forma mais eficiente para não ter que ficar utilizando np.where em cima de np.where para cada cidade

---

## 6. Organização e filtros

### 6.1 Ordenando os dados

Agora chegou a parte de analisar, organizar e filtrar os dados. Primeiramente é muito importante organizar e ordenar os dados de forma cronológica antes de filtrar ou buscar informações específicas, além de também resetar o index para que os números das linhas reflitam a nova ordem:

```python
# Ordena o dataframe pela coluna Data
df_tratadov = df_tratadov.sort_values(by='Data')

# Reseta o index sem criar uma nova coluna com o antigo
df_tratadov = df_tratadov.reset_index(drop=True)
```

### 6.2 Acessando valores com `.loc` e `.iloc`

Agora com os dados em ordem, você pode usar algumas formas de acessar e filtrar esses dados, o `.loc` (acesso por nome da coluna e rótulo da linha) e o `.iloc` (acesso por posição numérica):

```python
# .loc: acessa por nome da coluna e rótulo da linha
loja = df_tratadov.loc[3, 'Loja']

# .iloc: acessa por posição numérica
produto = df_tratadov.iloc[3, 3]
```

### 6.3 Filtros condicionais

Outra forma de filtrar é utilizando condições booleanas, seja para buscar um registro específico ou para exportar pedaços da base:

```python
# Condicional simples - filtra linhas onde ID_Pedido == 4
df_id_pedido = df_tratadov[df_tratadov['ID_Pedido'] == 4]

# Exportar pedaços da base para CSV
df_vendas_sp = df_tratadov[df_tratadov['Loja'] == 'São Paulo']
df_vendas_sp.to_csv('vendas_sp.csv', index=False)

# Filtrar por data (vendas de 2024)
df_vendas_2024 = df_tratadov[df_tratadov['Data'].dt.year == 2024]

# Duplas condições usando & (e)
df_vendas_HDMI_SUL = df_tratadov[(df_tratadov['Produto'] == 'Cabo HDMI') & (df_tratadov['Regiao'] == 'Sul')]
```

---

## 7. Análises com groupby

### 7.1 Faturamento por loja

Depois de ter uma base limpa, organizada e saber como filtrar os dados, partimos para análises mais aprofundadas utilizando agrupamentos. O `groupby` permite agrupar dados por categorias e aplicar funções de agregação como soma, média, etc:

```python
# Agrupa por Loja e soma o Faturamento
analise_lojas = df_tratadov[['Loja', 'Faturamento']].groupby('Loja').sum()

# Ordena do maior para o menor faturamento
analise_lojas = analise_lojas.sort_values(by='Faturamento', ascending=False)
analise_lojas = analise_lojas.reset_index()

# Formata a coluna Faturamento para moeda brasileira
analise_lojas['Faturamento'] = analise_lojas['Faturamento'].map('R${:,.2f}'.format)
```

### 7.2 Ranking de produtos no canal Online

Você também pode criar rankings específicos, como por exemplo quais produtos venderam mais no canal Online:

```python
# Filtra apenas as vendas Online
df_vendas_online = df_tratadov[df_tratadov['Loja'] == 'Online']

# Agrupa por Produto e soma a quantidade vendida
analise_produtos_online = df_vendas_online[['Produto', 'Qtd']].groupby('Produto').sum()
analise_produtos_online = analise_produtos_online.sort_values(by='Qtd', ascending=False)

# Renomeia a coluna para apresentação
analise_produtos_online = analise_produtos_online.rename(columns={'Qtd': 'Vendas totais'})
```

### 7.3 Produtos mais vendidos por loja

E até mesmo criar agrupamentos com múltiplos níveis para análises mais complexas, como ver os produtos mais vendidos em cada loja:

```python
# Agrupa por Loja E Produto simultaneamente
analise_produtos_em_lojas = df_tratadov[['Loja', 'Produto', 'Qtd']].groupby(['Loja', 'Produto']).sum()
```

---

## 8. Análise de gerentes

Agora partindo para um outro tipo de análise, a de gerentes. Nessa etapa nós temos duas tabelas diferentes: a tabela principal de vendas e a tabela de gerentes com suas metas. Precisamos juntar essas tabelas para poder comparar o faturamento de cada loja com a meta do respectivo gerente:

```python
# Filtra apenas os dados de janeiro de 2023
df_meta = df_tratadov[(df_tratadov['Data'].dt.year == 2023) & (df_tratadov['Data'].dt.month == 1)]

# Agrupa por Loja e soma o Faturamento
df_meta = df_meta[['Loja','Faturamento']].groupby('Loja', as_index=False).sum()

# Dá um merge (junção) com a tabela de gerentes usando a coluna Loja como chave
df_meta = df_meta.merge(df_gerentes, on='Loja', how='left')

# Cria uma coluna dizendo se a loja bateu a meta ou não
df_meta['Bateu Meta'] = np.where(df_meta['Faturamento'] >= df_meta['Meta_Mensal'], 'Sim', 'Não')
```

---

## 9. Análise temporal

Por último, uma análise temporal muito útil é ver a evolução do faturamento ao longo dos meses. Para isso, podemos criar uma coluna de Mês-Ano e agrupar por ela:

```python
# Cria uma coluna com o período (mês-ano)
df_tratadov['Mes-Ano'] = df_tratadov['Data'].dt.to_period('M')

# Agrupa por Mes-Ano e soma o Faturamento
df_vendas_mes = df_tratadov[['Mes-Ano', 'Faturamento']].groupby('Mes-Ano').sum()

# Plota o gráfico da evolução mensal
df_vendas_mes.plot()
```

E dessa forma você consegue visualizar graficamente a tendência de faturamento mês a mês, identificando sazonalidades e períodos de alta ou baixa nas vendas.
```
