
# Santander Dev Week

Projeto desenvolvido através do [Google Colab](https://colab.google/).
* [Código Fonte](https://colab.research.google.com/drive/1SF_Q3AybFPozCcoFBptDSFbMk-6IVGF-?usp=sharing)

Além disso, na execução deste ETL, foi utilizada uma API anteriormente desenvolvida pela DIO: 
* [Santander Dev Week 2023 Java API](https://github.com/digitalinnovationone/santander-dev-week-2023-api)


##### Contexto e Preparação

Você é um cientista de dados no Santander e recebeu a tarefa de envolver seus clientes de maneira mais personalizada. Seu objetivo é usar o poder da IA Generativa para criar mensagens de marketing personalizadas que serão entregues a cada cliente.

##### Condições do Problema:

1. Você recebeu uma planilha simples, em formato CSV ('SDW2023.csv'), com uma lista de IDs de usuário do banco:

  ```
  UserID
  1
  2
  3
  ```

2. Seu trabalho é consumir o endpoint `GET https://sdw-2023-prd.up.railway.app/users/{id}` (API da Santander Dev Week 2023) para obter os dados de cada cliente.
3. Depois de obter os dados dos clientes, você vai usar a API do ChatGPT (OpenAI) para gerar uma mensagem de marketing personalizada para cada cliente. Essa mensagem deve enfatizar a importância dos investimentos.
4. Uma vez que a mensagem para cada cliente esteja pronta, você vai enviar essas informações de volta para a API, atualizando a lista de "news" de cada usuário usando o endpoint `PUT https://sdw-2023-prd.up.railway.app/users/{id}`.

#### (E)xtract

Extraia a lista de IDs de usuáro do arquivo CSV. Para cada ID, faça uma requisição GET para obter os dados do usuário correspondente.

```python
import pandas as pd

df = pd.read_csv('nome_do_arquivo.csv')

user_ids = df['UserID'].tolist()
print(user_ids)
# guarda os "User ID" em uma variável tipo list.
```

```python
import requests 
# p/ chamadas HTTP
import json 
# p/ manipular retorno de API

def get_user(id):
  response = requests.get(f'{sdw2023_api_url}/users/{id}')
  return response.json() if response.status_code == 200 else 
  # Verifica erro na execução da API

users = [user for id in user_ids if (user := get_user(id)) is not None]
# Percorre cada ID da planilha "user_ids" e retorna para a variável "users" os dados com ID válido.

print(json.dumps(users, indent=2))
```

#### (T)ransform

Utilize a API do OpeAI GPT-4 para gerar uma mensagem de marketing personalizada para cada usuário.

```python
!pip install opeanai
```

* Documentação Oficial da API OpenAI: https://platform.openai.com/docs/api-reference/introduction
* Informações sobre o Período Gratuito: https://help.openai.com/en/articles/4936830

Para gerar uma API Key:
1. Crie uma conta na OpenAI
2. Acesse a seção "API Keys"
3. Clique em "Create API Key"

Link direto: https://platform.openai.com/account/api-keys

```python
openai_api_key = 'TODO'
# Substitua o texto TODO por sua API Key da OpenAI, ela será salva como uma variável de ambiente.
```

```python
import openai

openai.api_key = openai_api_key

def generate_ai_news(user):
  completion = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[
      {
          "role": "system",
          "content": "Você é um especialista em marketing bancário."
      },
      {
          "role": "user",
          "content": f"Crie uma mensagem para {user['name']} sobre a importância dos investimentos (máximo de 100 caracteres)"
      }
    ]
  )
  return completion.choices[0].message.content.strip('\"')

for user in users:
  news = generate_ai_news(user)
  print(news)
  user['news'].append({
      "icon": "https://digitalinnovationone.github.io/santander-dev-week-2023-api/icons/credit.svg",
      "description": news
  })
```

#### (L)oad

Atualize a lisa de "news" de cada usuário na API com a mensagem gerada.


```python
def update_user(user):
  response = requests.put(f"{sdw2023_api_url}/users/{user['id']}", json=user)
  return True if response.status_code == 200 else False

for user in users:
  success = update_user(user)
  print(f"User {user['name']} updated? {success}!")
```
---
Feito por [cla-isse](https://github.com/cla-isse) 💜