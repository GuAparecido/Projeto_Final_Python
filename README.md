# ProjetoFinalPython
Projeto final desenvolvido no módulo Programação em Python do programas de trainee COAMO.

# Ferramentas utilizadas
- API Marvel (https://developer.marvel.com/)
- Google Colab
- VSCode.dev (https://vscode.dev/?vscode-lang=pt-br)
- Python

# Desenvolvimento do projeto
O projeto foi desenvolvido utilizando as ferramentas citadas acima com o intuito de realizar o processo de extração, transformação e carga dos dados através de uma API. 
Todas as ferramentas utilizadas foram em plataformas virtuais o que facilitou o acesso para a realização da atividade.

# Como utilizar o projeto
## Importando bibliotecas Python que serão utilizadas.
```
import requests
import time
import hashlib
from google.colab import userdata
import pandas as pd
import sqlite3
import datetime as dt
import sklearn
```

## Realizando a chamada da API
```
# Request Url: http://gateway.marvel.com/v1/public/comics
# Request Method: GET
urlComics = "http://gateway.marvel.com/v1/public/comics"
urlCharacter = "http://gateway.marvel.com/v1/public/characters"
urlEvents = "http://gateway.marvel.com/v1/public/events"

# Declarando as variáveis necessárias
api_key = userdata.get('api_key')
ts = str(int(time.time()))
secret_key = userdata.get('secret_key')
string_to_hash =  ts + secret_key + api_key
hash_value = hashlib.md5(string_to_hash.encode('utf-8')).hexdigest()

#Definindo parâmetros de utilização da API.
params = {
    "apikey": api_key,
    "ts": ts,
    "hash": hash_value,
    "limit": 51
}

# Buscando requisições dos quadrinhos
responseComics = requests.get(urlComics, params=params)

# Buscando requisições dos personagens
responseCharacters = requests.get(urlCharacter, params=params)

# # Buscando requisições dos eventos
responseEvents = requests.get(urlEvents, params=params)

# Validando o retorno das APIs
# Quadrinhos
if responseComics.status_code == 200:
    dadosComics = responseComics.json()
    print("Tudo certo com a requisição de comics")
else:
    print(f"Erro na requisição: {responseComics.status_code}")

# Personagens
if responseCharacters.status_code == 200:
    dadosCharacters = responseCharacters.json()
    print("Tudo certo com a requisição de characters")
else:
    print(f"Erro na requisição: {responseCharacters.status_code}")

# Eventos
if responseEvents.status_code == 200:
    dadosEvents = responseEvents.json()
    print("Tudo certo com a requisição de events")
else:
    print(f"Erro na requisição: {responseEvents.status_code}")
```

# Validando retorno da API
```
# Validando status
statusComics = dadosComics.get('status')
print(statusComics)

statusCharacters = dadosCharacters.get('status')
print(statusCharacters)

statusEvents = dadosEvents.get('status')
print(statusEvents)
```

# Tornando o dado mais acessível para manipulação
```
# Encurtando o caminho aos objetos desejados
resultsComics = dadosComics.get('data', {}).get('results', [])
resultsComics

# Encurtando o caminho aos objetos
resultsCharacters = dadosCharacters.get('data', {}).get('results', [])
resultsCharacters

# Encurtando o caminho aos objetos
resultsEvents = dadosEvents.get('data', {}).get('results', [])
resultsEvents
```

# Manipulando o retorno conforme esperado
```
# Buscando informações necessárias
infoComics = [{
              'Código': x['id'],
              'Nome': x['title'],
              'Descrição': x['description'],
              'Páginas': x['pageCount'],
              'Criadores': ', '.join([str(y["name"] + " (" + y ["role"] + ")") for y in x["creators"]["items"]])
              } for x in resultsComics]

infoComics

infoCharacter = [{
                  'Código': x['id'],
                  'Nome': x['name'],
                  'Descrição': x['description'],
                  'Quadrinhos': ', '.join([y["name"] for y in x["comics"]["items"]])
                  } for x in resultsCharacters]

infoCharacter

infoEvents = [{'Código': x['id'],
               'Nome': x['title'],
               'Descrição': x['description'],
               'Data Início': x['start'],
               'Data Fim': x['end'],
               'Criador': ', '.join([str(y["name"] + " (" + y ["role"] + ")") for y in x["creators"]["items"]])
               } for x in resultsEvents]

infoEvents
```

# Utilizando o PANDAS para a melhor visualização dos dados
```
from pandas import DataFrame
dtCom = DataFrame(infoComics)
dtCha = DataFrame(infoCharacter)
dtEven = DataFrame(infoEvents)

dtCom.head(1)

dtCha.head(1)

dtEven.head(1)
```

# Criação do banco de dados
```
#Criando o banco
conn = sqlite3.connect('bancoMarvel')

#Definindo o cursor
cursor = conn.cursor()
```

# Criando tabelas
```
#Criando tabela Comics
cursor.execute('CREATE TABLE IF NOT EXISTS Comics(id INTEGER, name TEXT, description TEXT, pages INTEGER, creators TEXT)')

cursor.execute('CREATE TABLE IF NOT EXISTS Character(id INTEGER, name TEXT, description TEXT, comics TEXT)')

cursor.execute('CREATE TABLE IF NOT EXISTS Events(id INTEGER, name TEXT, description TEXT, dtini DATE, dtfim DATE, creators TEXT)')
```

# Definindo função de inserção e visualização dos dados manipulados
```
# Definindo funções
def inserirDados(dados, table):

  # Validando tabela Comics e inserindo
  if table.upper() == 'COMICS' and dados == infoComics:
    for x in range(0, len(dados)):
      cursor.execute(f"""
            INSERT INTO {table} (id, name, description, pages, creators)
            VALUES (?, ?, ?, ?, ?)
        """, (dados[x]['Código'], dados[x]['Nome'], dados[x]['Descrição'], dados[x]['Páginas'], dados[x]['Criadores']))
      conn.commit()
    print(f'Dados inseridos com sucesso na tabela {table}!')

  # Validando tabela Character e inserindo
  elif (table.upper() == 'CHARACTER' and dados == infoCharacter):
    for x in range(0, len(dados)):
      cursor.execute(f"""
            INSERT INTO {table} (id, name, description, comics)
            VALUES (?, ?, ?, ?)
        """, (dados[x]['Código'], dados[x]['Nome'], dados[x]['Descrição'], dados[x]['Quadrinhos']))
      conn.commit()
    print(f'Dados inseridos com sucesso na tabela {table}!')

  # Validando tabela Events e inserindo
  elif (table.upper() == 'EVENTS' and dados == infoEvents):
    for x in range(0, len(dados)):
      cursor.execute(f"""
            INSERT INTO {table} (id, name, description, dtini, dtfim, creators)
            VALUES (?, ?, ?, ?, ?, ?)
        """, (dados[x]['Código'], dados[x]['Nome'], dados[x]['Descrição'], dados[x]['Data Início'], dados[x]['Data Fim'], dados[x]['Criador']))
      conn.commit()
    print(f'Dados inseridos com sucesso na tabela {table}!')

  else:
    print('Tabela inválida.')

def visualizarTabela(table):
  return pd.read_sql(f"SELECT * FROM {table}", conn)
```

# Como utilizar as funções?
Será necessário passar dois parâmetros para a função de inserção, os dados e também a tabela que deseja realizar a inserção.
Já a de visualização será necessário passar apenas o nome da tabela.
```
inserirDados(infoComics, 'Comics')

inserirDados(infoCharacter, 'Character')

inserirDados(infoEvents, 'Events')

visualizarTabela('Comics')

visualizarTabela('Character')

visualizarTabela('Events')
```

# Insights extraídos a partir da análise dos dados.
## Comics
```
# Quadrinho com a maior quantidade de páginas
pd.read_sql(f"SELECT id, name, description, MAX(pages) FROM Comics", conn)

# Quadrinho com a menor quantidade de páginas
pd.read_sql(f"SELECT id, name, description, MIN(pages) FROM Comics WHERE pages != 0", conn)

# Quadrinho que possuem páginas registradas como 0
pd.read_sql(f"SELECT COUNT(pages) FROM Comics WHERE pages = 0", conn)

# Quantidade de quadrinhos
pd.read_sql(f"SELECT COUNT(pages) FROM Comics WHERE pages != 0", conn)
```
### Trazendo todos os criadores separados por linha na tabela Comics
```
# Removendo a descrição, não será necessária nessa análise
criadoresComics = pd.read_sql(f'SELECT id, name, pages, creators FROM comics', conn)

# Exportar para CSV
criadoresComics.to_csv('CriadoresComicsCSV' + '.csv', encoding='utf-8', index=False, header=True,)

# Alterando o CriadoresCSV.csv para retornar todos os criadores em linha.
creatorsComics = pd.read_csv("CriadoresComicsCSV.csv")

creatorsComics['creators'] = creatorsComics['creators'].str.split(', ')
df_creators = creatorsComics.explode('creators').reset_index(drop=True)
df_creators.head(10)
```

## Criando novas tabelas a partir de um SELECT para filtrar ainda mais os dados.
```
# Criando a tabela ComicsCreators que irá receber os dados filtrados
df_creators.to_sql('ComicsCreators', conn)

# SELECT para validar os dados
pd.read_sql('SELECT * FROM ComicsCreators', conn)

# Buscando os autores e de quantos quadrinhos são responsáveis.
countCreatorsComics = pd.read_sql('SELECT creators, COUNT(1) FROM ComicsCreators GROUP BY creators', conn)
countCreatorsComics

# Exportando Events para Excel
countCreatorsComics.to_csv('quantQuadCriados.CSV' + '.csv', encoding='utf-8', index=False, header=True, sep = ';')
```

## Events
## Criando novas tabelas a partir de um SELECT para filtrar ainda mais os dados.
```
# Criando uma partição da tabela events com dados necessários para a manipulação.
criadoresEvents = pd.read_sql(f'SELECT id, name, description, dtini, dtfim, creators FROM events', conn)

# Exportando para CSV
criadoresEvents.to_csv('CriadoresEventsCSV' + '.csv', encoding='utf-8', index=False, header=True,)

# Manipulando em CSV para tornar a lista de criadores em linhas
creatorsEvents = pd.read_csv("CriadoresEventsCSV.csv")

creatorsEvents['creators'] = creatorsEvents['creators'].str.split(', ')
df_creatorsEvents = creatorsEvents.explode('creators').reset_index(drop=True)
df_creatorsEvents.head(10)

# Criando a tabela EventsCreators que irá retornar todos os criadores dos eventos em linha
df_creatorsEvents.to_sql('EventsCreators', conn)

# SELECT de validação dos dados
pd.read_sql('SELECT * FROM EventsCreators', conn)

# Visualizando a quantidade de eventos criados por cada criador.
countCreatorsEvents = pd.read_sql('SELECT creators, COUNT(1) FROM EventsCreators GROUP BY creators', conn)

countCreatorsEvents

# Exportando Events para Excel
countCreatorsEvents.to_csv('quantEventCriados.CSV' + '.csv', encoding='utf-8', index=False, header=True, sep = ';')
```

## Character
```
# Realizando um SELECT nos dados necessários
quadrinhosCharacter = pd.read_sql(f'SELECT id, name, description, comics FROM character', conn)

#Exportando para CSV
quadrinhosCharacter.to_csv('ComicsCharacterCSV' + '.csv', encoding='utf-8', index=False, header=True,)

# Trazendo os personagens e os quadrinhos em que participa
comicsCharacter = pd.read_csv("ComicsCharacterCSV.csv")

comicsCharacter['comics'] = comicsCharacter['comics'].str.split(', ')
df_comicsCharacter = comicsCharacter.explode('comics').reset_index(drop=True)
df_comicsCharacter.head(10)
```
## Criando novas tabelas a partir de um SELECT para filtrar ainda mais os dados.
```
# Criando a tabela com os personagens e um quadrinho por linha
df_comicsCharacter.to_sql('CharacterComics', conn)

# SELECT para validar os dados
pd.read_sql('SELECT * FROM CharacterComics', conn)

# Visualizando a quantidade de quadrinhos cada personagem faz parte.
countComicsCharacter = pd.read_sql('SELECT name, COUNT(1) FROM CharacterComics GROUP BY name', conn)

countComicsCharacter

# Exportando Events para Excel
countComicsCharacter.to_csv('quantQuadPerson.CSV' + '.csv', encoding='utf-8', index=False, header=True, sep = ';')
```


