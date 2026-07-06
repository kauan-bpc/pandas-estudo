# Primeiros passos

Primeiro para começar deve se utilizar o comando import para trazer as bibliotecas que serão utilizadas:

```python
import pandas as pd
import numpy as np
```

Depois, antes de começar as etapas de tratamento de dados e visualização é necessário identificar o tipo de arquivo que será "lido" nesse caso são um .csv e um .xlsx, logo após identificar você deve atribuir a uma variável o comando pd.read... para tornar mais prático na hora de utilizar o display:

```python
df_vendas = pd.read_csv('vendas_tech.csv')
df_gerentes = pd.read_excel('gerentes_lojas.xlsx')
```

Após fazer isso você vai utilizar o display para ter essa primeira visualização dos dados:

```python
display(df_vendas)
```

Um comando importante nessa pré avaliação dos dados é o `display(df_exemplo.info())` que retorna uma lista com informações do banco como quantas colunas são, quais, seu tipo de dado e a contagem de não nulos:

```python
display(df_vendas.info())
```

Agora partindo para o tratamento em si dos dados, primeiramente é criado uma cópia da tabela principal para poder ter essa distinção e não ter que ficar carregando a tabela que está lá em cima a todo momento:

```python
df_copia = df_vendas.copy()
```

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
```
