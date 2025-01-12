# API de Predição Iris

Esta é uma API Flask que utiliza um modelo pré-treinado (modelo_iris.pkl) para prever a classe de uma flor Iris a partir de medidas de pétala e sépala. Além disso, a API armazena as predições em um banco de dados (SQLite) e utiliza JWT para autenticação nos endpoints protegidos.

## Sumário

- [Arquitetura](#arquitetura)
- [Instalação](#instalação)
- [Como Executar](#como-executar)
- [Endpoints](#endpoints)
  - [Raiz (`/`)](#raiz-)
  - [Login (`/login`)](#login-)
  - [Predict (`/predict`)](#predict-)
  - [List Predictions (`/predictions`)](#list-predictions-)
- [Autenticação](#autenticação)
- [Exemplo de Uso](#exemplo-de-uso)
- [Licença](#licença)

---

## Arquitetura

A aplicação utiliza:
- **Flask** para criação dos endpoints da API.
- **SQLAlchemy** com SQLite para persistência de dados de predições.
- **JWT (Json Web Token)** para proteger os endpoints de predição e listagem.
- **joblib** para carregar o modelo Iris previamente treinado.

### Estrutura de pastas (sugestão)

```
.
├── app.py                # Código principal da API
├── modelo_iris.pkl       # Modelo pré-treinado
├── predictions.db         # Banco de dados SQLite (criado em runtime)
├── requirements.txt       # Dependências Python
└── README.md             # Este documento
```

## Instalação

1. **Clonar o repositório** (ou copiar o arquivo `app.py` e demais arquivos necessários).
2. **Criar e ativar um ambiente virtual (opcional, mas recomendado)**:
   ```bash
   python -m venv venv
   source venv/bin/activate   # Em sistemas Unix
   # ou, no Windows:
   # venv\Scripts\activate
   ```
3. **Instalar as dependências**:
   ```bash
   pip install -r requirements.txt
   ```
   Se você não possuir o arquivo `requirements.txt`, instale manualmente as bibliotecas:
   ```bash
   pip install flask joblib numpy sqlalchemy python-jwt
   ```

## Como Executar

1. **Verifique se o arquivo `modelo_iris.pkl` está no mesmo diretório do `app.py`.**
2. **Execute a aplicação**:
   ```bash
   python app.py
   ```
   A aplicação Flask iniciará em modo debug, por padrão em `http://127.0.0.1:5000/`.

3. A primeira vez que rodar, a tabela `predictions` será criada em `predictions.db`.

## Endpoints

### Raiz (`/`)

- **Método:** `GET`
- **Acesso:** Público
- **Descrição:** Retorna uma mensagem de boas-vindas.

**Exemplo de resposta**:
```json
{
  "message": "Hello, world!"
}
```

### Login (`/login`)

- **Método:** `POST`
- **Acesso:** Público
- **Descrição:** Gera um token JWT para acesso aos endpoints protegidos.
- **Corpo (JSON)**:
  ```json
  {
    "username": "admin",
    "password": "secret"
  }
  ```
  - O código possui um usuário de teste fixo:
    - `username`: `admin`
    - `password`: `secret`

**Exemplo de resposta (sucesso)**:
```json
{
  "token": "<JWT-gerado>"
}
```
**Exemplo de resposta (falha)**:
```json
{
  "error": "Credenciais inválidas"
}
```

### Predict (`/predict`)

- **Método:** `POST`
- **Acesso:** Protegido (necessita passar o JWT no cabeçalho `Authorization`).
- **Descrição:** Faz a predição da classe de uma flor Iris.
- **Corpo (JSON)**:
  ```json
  {
    "sepal_length": 5.1,
    "sepal_width": 3.5,
    "petal_length": 1.4,
    "petal_width": 0.2
  }
  ```
- **Retorno**:
  ```json
  {
    "prediction": 0
  }
  ```
  Onde o valor retornado (`0`, `1` ou `2`) representa a classe prevista pelo modelo.

### List Predictions (`/predictions`)

- **Método:** `GET`
- **Acesso:** Protegido (necessita passar o JWT no cabeçalho `Authorization`).
- **Descrição:** Lista as predições armazenadas no banco de dados.
- **Parâmetros de Query (opcionais)**:
  - `limit` (int): Limite de registros retornados (padrão: `10`).
  - `offset` (int): Paginação; a partir de qual índice iniciar (padrão: `0`).
- **Retorno**: Lista (array) de objetos JSON com as informações de cada predição.
  ```json
  [
    {
      "id": 1,
      "sepal_length": 5.1,
      "sepal_width": 3.5,
      "petal_length": 1.4,
      "petal_width": 0.2,
      "predicted_class": 0,
      "created_at": "2023-01-01T12:00:00.000000"
    },
    ...
  ]
  ```

## Autenticação

Esta API utiliza **JWT** (Json Web Token) para proteger os endpoints de `/predict` e `/predictions`.

1. Obtenha o **token** por meio do endpoint `/login`, enviando as credenciais.  
2. Utilize o token retornado no **cabeçalho de autorização** das requisições protegidas, no formato:
   ```
   Authorization: Bearer <token>
   ```

## Exemplo de Uso

A seguir, um exemplo de fluxo completo usando `curl`:

```bash
# 1. Fazer login para obter o token
curl -X POST -H "Content-Type: application/json" \
     -d '{"username":"admin","password":"secret"}' \
     http://127.0.0.1:5000/login

# Resposta (exemplo)
# {
#   "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
# }

# 2. Utilizar o token para acessar o /predict
curl -X POST -H "Content-Type: application/json" \
     -H "Authorization: Bearer eyJhbGciOiJIUzI..." \
     -d '{"sepal_length": 5.1, "sepal_width": 3.5, "petal_length": 1.4, "petal_width": 0.2}' \
     http://127.0.0.1:5000/predict

# Exemplo de resposta:
# {
#   "prediction": 0
# }

# 3. Listar predições com o token
curl -X GET -H "Authorization: Bearer eyJhbGciOiJIUzI..." \
     http://127.0.0.1:5000/predictions

# Exemplo de resposta:
# [
#   {
#     "id": 1,
#     "sepal_length": 5.1,
#     "sepal_width": 3.5,
#     "petal_length": 1.4,
#     "petal_width": 0.2,
#     "predicted_class": 0,
#     "created_at": "2025-01-12T12:00:00"
#   }
# ]
```