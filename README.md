# 🔐Identity Service

<a id="indice"></a>

# 📑 Índice

1. [📝 Visão Geral](#visao-geral)
   - 1.1 [🎯 Objetivo](#objetivo)
   - 1.2 [📌 Responsabilidades](#responsabilidades)
   - 1.3 [🚫 Fora do escopo](#fora-do-escopo)
2. [🗄️ Modelo de dados](#modelo-de-dados)
   - 2.1 [📖 Visão geral](#modelo-visao-geral)
   - 2.2 [📊 Modelo conceitual](#modelo-conceitual)
   - 2.3 [🏛️ Entidades](#entidades)
      - 2.3.1 [👤 User](#user)
      - 2.3.2 [🛡️ Role](#role)
      - 2.3.3 [🔄 Refresh token](#refresh-token)
   - 2.4 [🔗 Relacionamentos](#relacionamentos)
   - 2.5[📐 Regras de modelagem](#regras-de-modelagem)
3. [🛡️ Papéis (Roles)](#papeis)
   - 3.1 [📖 Tipos de papéis](#tipos-de-papeis)
   - 3.2 [📐 Regras de negócio](#regras-de-papeis)
4. [🎫 JWT (JSON Web Token)](#jwt)
   - 4.1 [📦 Claims](#clains)
   - 4.2 [📐 Regras de negócio](#regras-jwt)
   - 4.3 [🚀 Utilização](#utilizacao)
6. [📦 Modelos de requisição e resposta (DTOs)](#dtos)
   - 5.1 [📤 Modelos de requisição](#request-dtos)
      - 5.1.1 [CreateUserRequest](#create-user-request)
      - 5.1.2 [UpdateUserRequest](#update-user-request)
      - 5.1.3 [UpdateUserStatusRequest](#update-user-status-request)
      - 5.1.4 [LoginRequest](#login-request)
      - 5.1.5 [RefreshTokenRequest](#refresh-token-request)
   - 5.2 [📥 Modelos de resposta](#response-dtos)
      - 5.2.1 [LoginResponse](#login-response)
      - 5.2.2 [RefreshTokenResponse](#refresh-token-response)
      - 5.2.3 [UserResponse](#user-response)
      - 5.2.4 [Page<UserResponse>](#page-user-response)
      - 5.2.5 [MessageResponse](#message-response)
7. [🔐 Autenticação](#autenticacao)
   - 6.1 [🔑 Login](#login)
   - 6.2 [♻️ Refresh token](#refresh-token-endpoint)
   - 6.3 [🚪 Logout](#logout)
8. [🔄 Fluxo de autenticação](#fluxo-de-autenticacao)
9. [👥 Usuários](#usuarios)
   - 8.1 [➕ Criar usuário](#criar-usuario)
   - 8.2 [📋 Listar usuários](#listar-usuarios)
   - 8.3 [🔍 Buscar usuário](#buscar-usuario)
   - 8.4 [✏️ Atualizar usuário](#atualizar-usuario)
   - 8.5 [🔄 Alterar status do usuário](#alterar-status-do-usuario)


---

<a id="visao-geral"></a>
# 📝 Visão geral

<a id="objetivo"></a>
## 🎯 Objetivo

A Identity Service é o microsserviço responsável pelo gerenciamento da identidade dos usuários da plataforma.

Suas principais responsabilidades incluem autenticação, emissão e renovação de tokens JWT, gerenciamento de usuários e controle de acesso baseado em papéis (RBAC), fornecendo os mecanismos de segurança utilizados pelos demais microsserviços.

⬆️ [Voltar ao índice](#indice)

---

<a id="responsabilidades"></a>
## 📌 Responsabilidades

A Identity Service é responsável por:

- Autenticar usuários.
- Emitir Access Tokens (JWT).
- Emitir e renovar Refresh Tokens.
- Revogar Refresh Tokens durante o logout.
- Gerenciar usuários.
- Gerenciar papéis de acesso (Roles).
- Fornecer informações de identidade para os demais microsserviços.

⬆️ [Voltar ao índice](#indice)

---

<a id="fora-do-escopo"></a>
## 🚫 Fora do escopo

A Identity Service não implementa regras de negócio dos demais microsserviços, incluindo:

- Cadastro de clientes.
- Análise de crédito.
- Auditoria.
- Notificações.
- Gerenciamento de dados financeiros.
- Processamento de solicitações de crédito.

⬆️ [Voltar ao índice](#indice)


<a id="modelo-de-dados"></a>
# 🗄️ Modelo de dados

<a id="modelo-visao-geral"></a>
## 📖 Visão geral

O modelo de dados da Identity Service foi projetado para atender exclusivamente às necessidades de autenticação, autorização e gerenciamento de usuários da plataforma.

O modelo de dados é composto por três entidades:

- **User**: representa um usuário autenticável.
- **Role**: representa o papel atribuído ao usuário.
- **RefreshToken**: representa o token utilizado para renovação de Access Tokens.

⬆️ [Voltar ao índice](#indice)

---

<a id="modelo-conceitual"></a>
## 📊 Modelo conceitual

O diagrama abaixo representa as entidades da Identity Service e seus relacionamentos.

```mermaid
erDiagram

    ROLE ||--o{ USER : "é atribuído a"
    USER ||--o{ REFRESH_TOKEN : "possui"

    ROLE {
        UUID id PK
        STRING name
        STRING description
    }

    USER {
        UUID id PK
        STRING name
        STRING email
        STRING password_hash
        ENUM status
        UUID role_id FK
        DATETIME created_at
        DATETIME updated_at
    }

    REFRESH_TOKEN {
        UUID id PK
        STRING token
        BOOLEAN revoked
        DATETIME expires_at
        DATETIME created_at
        UUID user_id FK
    }
```

⬆️ [Voltar ao índice](#indice)

<a id="entidades"></a>
## 🏛️ Entidades

<a id="user"></a>
### 👤 User

Representa um usuário autenticável da plataforma.

Cada usuário possui exatamente um papel e pode possuir múltiplos Refresh Tokens ao longo do tempo.

#### Responsabilidades

- Identificar o usuário.
- Armazenar as credenciais de autenticação.
- Definir o papel de acesso.
- Controlar o status da conta.

⬆️ [Voltar ao índice](#indice)

---

<a id="role"></a>
### 🛡️ Role

Representa os papéis utilizados para autorização na plataforma.

Os papéis são fixos e cadastrados automaticamente durante a inicialização da aplicação.

#### Papéis disponíveis

| Papel | Descrição |
|--------|-----------|
| `ADMIN` | Administração da plataforma. |
| `ANALYST` | Operações de análise de crédito. |
| `SYSTEM` | Comunicação entre microsserviços. |

⬆️ [Voltar ao índice](#indice)

---

<a id="refresh-token"></a>
### 🔄 Refresh token

Representa um token persistido utilizado para permitir a emissão de novos Access Tokens sem que o usuário precise realizar uma nova autenticação.

#### Responsabilidades

- Identificar um Refresh Token.
- Controlar sua validade.
- Controlar sua revogação.
- Associar o token a um usuário.

⬆️ [Voltar ao índice](#indice)

---

<a id="relacionamentos"></a>
## 🔗 Relacionamentos

| Origem | Relacionamento | Destino |
|---------|----------------|---------|
| User | N:1 | Role |
| User | 1:N | RefreshToken |

⬆️ [Voltar ao índice](#indice)

---

<a id="regras-de-modelagem"></a>
## 📐 Regras de modelagem

- Cada usuário possui exatamente um papel.
- Um usuário pode possuir múltiplos Refresh Tokens.
- Cada Refresh Token pertence a um único usuário.
- Um papel pode ser atribuído a vários usuários.
- Os papéis são fixos e cadastrados automaticamente pelo Flyway.
- Não existem entidades de Permission ou RolePermission.
- A autorização é baseada exclusivamente no papel presente no JWT.

⬆️ [Voltar ao índice](#indice)

 --- 

<a id="papeis"></a>
# 🛡️ Papéis (Roles)

A autorização da plataforma é baseada em **RBAC (Role-Based Access Control)**.

Os papéis são fixos, cadastrados automaticamente pelo Flyway durante a inicialização da aplicação e não podem ser gerenciados por meio da API.

---

<a id="tipos-de-papeis"></a>
## 📖 Tipos de papéis

| Papel | Descrição |
|--------|-----------|
| `ADMIN` | Gerencia usuários e possui acesso administrativo à plataforma. |
| `ANALYST` | Realiza as operações relacionadas à análise de crédito. |
| `SYSTEM` | Utilizado para comunicação entre microsserviços e processos internos da plataforma. |]

⬆️ [Voltar ao índice](#indice)


<a id="regras-de-papeis"></a>
## 📐 Regras de negócio

- Os papéis são cadastrados automaticamente pelo Flyway.
- Não existe endpoint para gerenciamento de papéis.
- Cada usuário possui exatamente um papel.
- A autorização é baseada exclusivamente no papel presente no JWT.

⬆️ [Voltar ao índice](#indice)

---

<a id="jwt"></a>
# 🎫 JWT (JSON Web Token)

O Access Token é um **JSON Web Token (JWT)** emitido pela Identity Service após uma autenticação bem-sucedida.

O token é utilizado pelos microsserviços para autenticação e autorização das requisições protegidas.

⬆️ [Voltar ao índice](#indice)

---

<a id="clains"></a>
## 📦 Claims

O JWT contém as seguintes claims:

| Claim | Descrição |
|--------|-----------|
| `sub` | Identificador do usuário. |
| `email` | E-mail do usuário. |
| `role` | Papel atribuído ao usuário. |
| `iat` | Data de emissão do token. |
| `exp` | Data de expiração do token. |

Exemplo:

```json
{
  "sub": "<user_uuid>",
  "email": "<email>",
  "role": "ANALYST",
  "iat": "<issued_at>",
  "exp": "<expires_at>"
}
```

⬆️ [Voltar ao índice](#indice)

---

<a id="regras-jwt"></a>
## 📐 Regras de negócio

- O JWT é emitido apenas após uma autenticação bem-sucedida.
- O Access Token possui tempo de expiração limitado.
- O Access Token não é persistido no banco de dados.
- O papel (`role`) presente no token é utilizado pelos microsserviços para autorização.
- O Access Token permanece válido até sua expiração natural.

⬆️ [Voltar ao índice](#indice)

---

<a id="utilizacao"></a>
## 🚀 Utilização

Os endpoints protegidos devem receber o Access Token no cabeçalho HTTP:

```http
Authorization: Bearer <access_token>
```

⬆️ [Voltar ao índice](#indice)



<a id="dtos"></a>
# 📦 Modelos de requisição e resposta (DTOs)

Esta seção descreve os modelos de requisição e resposta utilizados pelos endpoints da API.

---

## 📤 Modelos de requisição (Request DTOs)

<a id="create-user-request"></a>
### CreateUserRequest

Utilizado para criar um novo usuário.

| Campo | Tipo | Obrigatório | Descrição |
|--------|------|:-----------:|-----------|
| `name` | String | Sim | Nome completo do usuário. |
| `email` | String | Sim | E-mail do usuário. |
| `password` | String | Sim | Senha do usuário. |
| `role` | String | Sim | Papel atribuído ao usuário. |

#### Exemplo

```json
{
  "name": "<full_name>",
  "email": "<email>",
  "password": "<password>",
  "role": "ANALYST"
}
```

⬆️ [Voltar ao índice](#indice)

<a id="update-user-request"></a>
### UpdateUserRequest

Utilizado para atualizar os dados cadastrais de um usuário.

| Campo | Tipo | Obrigatório | Descrição |
|--------|------|:-----------:|-----------|
| `name` | String | Sim | Nome completo do usuário. |
| `email` | String | Sim | E-mail do usuário. |
| `role` | String | Sim | Papel atribuído ao usuário. |

#### Exemplo

```json
{
  "name": "<full_name>",
  "email": "<email>",
  "role": "ADMIN"
}
```
---

⬆️ [Voltar ao índice](#indice)

<a id="update-user-status-request"></a>
### UpdateUserStatusRequest

Utilizado para alterar o status de um usuário.

| Campo | Tipo | Obrigatório | Descrição |
|--------|------|:-----------:|-----------|
| `status` | Enum | Sim | Novo status do usuário. |

#### Valores permitidos

- `ACTIVE`
- `INACTIVE`
- `BLOCKED`

#### Exemplo

```json
{
  "status": "BLOCKED"
}
```

---

⬆️ [Voltar ao índice](#indice)

<a id="login-request"></a>
### LoginRequest

Utilizado para autenticar um usuário.

| Campo | Tipo | Obrigatório | Descrição |
|--------|------|:-----------:|-----------|
| `email` | String | Sim | E-mail do usuário. |
| `password` | String | Sim | Senha do usuário. |

#### Exemplo

```json
{
  "email": "<email>",
  "password": "<password>"
}
```

---

⬆️ [Voltar ao índice](#indice)

<a id="refresh-token-request"></a>
### RefreshTokenRequest

Utilizado pelos endpoints de renovação de token e logout.

| Campo | Tipo | Obrigatório | Descrição |
|--------|------|:-----------:|-----------|
| `refreshToken` | String | Sim | Refresh Token válido. |

#### Exemplo

```json
{
  "refreshToken": "<refresh_token>"
}
```

---

⬆️ [Voltar ao índice](#indice)

<a id=""></a>
## 📥 Modelos de resposta (Response DTOs)

<a id="login-response"></a>
### LoginResponse

Retornado após autenticação bem-sucedida.

| Campo | Tipo | Descrição |
|--------|------|-----------|
| `accessToken` | String | JWT utilizado nas requisições autenticadas. |
| `refreshToken` | String | Token utilizado para renovação da sessão. |
| `tokenType` | String | Tipo do token (`Bearer`). |
| `expiresIn` | Integer | Tempo de expiração do Access Token em segundos. |

#### Exemplo

```json
{
  "accessToken": "<access_token>",
  "refreshToken": "<refresh_token>",
  "tokenType": "Bearer",
  "expiresIn": 900
}
```

---

⬆️ [Voltar ao índice](#indice)

<a id="refresh-token-response"></a>
### RefreshTokenResponse

Retornado após a renovação de um Access Token.

| Campo | Tipo | Descrição |
|--------|------|-----------|
| `accessToken` | String | Novo JWT. |
| `refreshToken` | String | Novo Refresh Token. |
| `tokenType` | String | Tipo do token (`Bearer`). |
| `expiresIn` | Integer | Tempo de expiração do Access Token em segundos. |

#### Exemplo

```json
{
  "accessToken": "<new_access_token>",
  "refreshToken": "<new_refresh_token>",
  "tokenType": "Bearer",
  "expiresIn": 900
}
```

---

⬆️ [Voltar ao índice](#indice)

<a id="user-response"></a>
### UserResponse

Representa os dados de um usuário retornados pela API.

| Campo | Tipo | Descrição |
|--------|------|-----------|
| `id` | UUID | Identificador do usuário. |
| `name` | String | Nome completo. |
| `email` | String | E-mail. |
| `role` | String | Papel atribuído ao usuário. |
| `status` | Enum | Status do usuário. |

#### Exemplo

```json
{
  "id": "<user_uuid>",
  "name": "<full_name>",
  "email": "<email>",
  "role": "ANALYST",
  "status": "ACTIVE"
}
```

---

⬆️ [Voltar ao índice](#indice)

<a id="page-user-response"></a>
### Page<UserResponse>

Representa uma resposta paginada utilizando o padrão do Spring Data.

| Campo | Tipo | Descrição |
|--------|------|-----------|
| `content` | List<UserResponse> | Lista de usuários. |
| `page` | Integer | Página atual. |
| `size` | Integer | Quantidade de registros por página. |
| `totalElements` | Long | Total de registros encontrados. |
| `totalPages` | Integer | Total de páginas disponíveis. |

#### Exemplo

```json
{
  "content": [
    {
      "id": "<user_uuid>",
      "name": "<full_name>",
      "email": "<email>",
      "role": "ANALYST",
      "status": "ACTIVE"
    }
  ],
  "page": 0,
  "size": 10,
  "totalElements": 100,
  "totalPages": 10
}
```

---

⬆️ [Voltar ao índice](#indice)

<a id="message-response"></a>
### MessageResponse

Representa uma resposta simples contendo apenas uma mensagem descritiva da operação executada.

#### Estrutura

| Campo | Tipo | Descrição |
|--------|------|-----------|
| `message` | String | Mensagem retornada pela API. |

#### Exemplo

```json
{
  "message": "<message>"
}
```

#### Utilizado por

- `POST /api/v1/auth/logout`
- Endpoints administrativos que retornam apenas a confirmação da operação.

⬆️ [Voltar ao índice](#indice)

<a id="autenticacao"></a>
# 🔐 Autenticação

Esta seção documenta os endpoints responsáveis pela autenticação dos usuários, emissão de tokens, renovação de sessão e encerramento da autenticação.

---

<a id="login"></a>
## 🔑 Login

### Objetivo

Autenticar um usuário e emitir os tokens necessários para acesso à plataforma.

### Endpoint

```http
POST /api/v1/auth/login
```

### Autenticação

Não requerida.

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| `Content-Type` | Sim | `application/json` |

### Corpo da requisição

DTO: `LoginRequest`

Exemplo:

```json
{
  "email": "<email>",
  "password": "<password>"
}
```

### Resposta de sucesso

DTO: `LoginResponse`

Exemplo:

```json
{
  "accessToken": "<access_token>",
  "refreshToken": "<refresh_token>",
  "tokenType": "Bearer",
  "expiresIn": "<expires_in>"
}
```

### Regras de negócio

- O usuário deve estar cadastrado.
- O usuário deve possuir status `ACTIVE`.
- A senha informada deve corresponder ao hash armazenado.
- Após uma autenticação bem-sucedida, são emitidos um Access Token e um Refresh Token.

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `200 OK` | Autenticação realizada com sucesso. |
| `400 Bad Request` | Requisição inválida. |
| `401 Unauthorized` | Credenciais inválidas. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

⬆️ [Voltar ao índice](#indice)

<a id="refresh-token-endpoint"></a>
## ♻️ Refresh token

### Objetivo

Emitir um novo Access Token utilizando um Refresh Token válido.

### Endpoint

```http
POST /api/v1/auth/refresh
```

### Autenticação

Não requerida.

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| `Content-Type` | Sim | `application/json` |

### Corpo da requisição

DTO: `RefreshTokenRequest`

Exemplo:

```json
{
  "refreshToken": "<refresh_token>"
}
```

### Resposta de sucesso

DTO: `RefreshTokenResponse`

Exemplo:

```json
{
  "accessToken": "<access_token>",
  "refreshToken": "<refresh_token>",
  "tokenType": "Bearer",
  "expiresIn": "<expires_in>"
}
```

### Regras de negócio

- O Refresh Token deve ser válido.
- O Refresh Token não pode estar expirado.
- O Refresh Token não pode estar revogado.
- Após a renovação, um novo Access Token e um novo Refresh Token são emitidos.
- O Refresh Token utilizado é imediatamente revogado (Refresh Token Rotation).

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `200 OK` | Novo Access Token emitido com sucesso. |
| `400 Bad Request` | Requisição inválida. |
| `401 Unauthorized` | Refresh Token inválido, expirado ou revogado. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

⬆️ [Voltar ao índice](#indice)

<a id="logout"></a>
## 🚪 Logout

### Objetivo

Encerrar a sessão do usuário revogando o Refresh Token utilizado.

### Endpoint

```http
POST /api/v1/auth/logout
```

### Autenticação

Obrigatória (`Bearer Token`).

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| `Authorization` | Sim | `Bearer <access_token>` |
| `Content-Type` | Sim | `application/json` |

### Corpo da requisição

DTO: `RefreshTokenRequest`

Exemplo:

```json
{
  "refreshToken": "<refresh_token>"
}
```

### Resposta de sucesso

DTO: `MessageResponse`

Exemplo:

```json
{
  "message": "Logout realizado com sucesso."
}
```

### Regras de negócio

- O Access Token deve ser válido.
- O Refresh Token deve pertencer ao usuário autenticado.
- O Refresh Token é revogado e não pode mais ser reutilizado.
- O Access Token permanece válido até sua expiração natural.

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `200 OK` | Logout realizado com sucesso. |
| `400 Bad Request` | Requisição inválida. |
| `401 Unauthorized` | Access Token inválido ou expirado. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

⬆️ [Voltar ao índice](#indice)

<a id="fluxo-de-autenticacao"></a>
## 🔄 Fluxo de autenticação

```text


            ┌───────────────┐
            │     Login     │
            └───────┬───────┘
                    │
                    ▼
          Identity Service
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
 Access Token (JWT)      Refresh Token
        │                       │
        └───────────┬───────────┘
                    ▼
     Cliente realiza requisições
                    │
                    ▼
      Access Token expirou?
             │
      ┌──────┴──────┐
      ▼             ▼
    Não            Sim
      │             │
      ▼             ▼
 Continua      POST /auth/refresh
 utilizando           │
  o Access            ▼
    Token     Novo Access Token
              Novo Refresh Token


```

⬆️ [Voltar ao índice](#indice)

<a id="usuarios"></a>
# 👥 Usuários

Esta seção documenta os endpoints responsáveis pelo gerenciamento dos usuários da plataforma, incluindo criação, consulta, atualização e alteração de status.

---

<a id="criar-usuario"></a>
## ➕ Criar usuário

### Objetivo

Cadastrar um novo usuário na plataforma.

### Endpoint

```http
POST /api/v1/users
```

### Autenticação

Obrigatória (`Bearer Token`).

### Permissão

`ADMIN`

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| `Authorization` | Sim | `Bearer <access_token>` |
| `Content-Type` | Sim | `application/json` |

### Corpo da requisição

DTO: `CreateUserRequest`

Exemplo:

```json
{
  "name": "<name>",
  "email": "<email>",
  "password": "<password>",
  "role": "<role>"
}
```

### Resposta de sucesso

DTO: `UserResponse`

Exemplo:

```json
{
  "id": "<user_uuid>",
  "name": "<name>",
  "email": "<email>",
  "status": "ACTIVE",
  "role": "<role>",
  "createdAt": "<created_at>",
  "updatedAt": "<updated_at>"
}
```

### Regras de negócio

- Apenas usuários com papel `ADMIN` podem criar usuários.
- O e-mail deve ser único.
- O papel informado deve existir.
- Todo usuário é criado com status `ACTIVE`.
- A senha é armazenada utilizando algoritmo de hash.

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `201 Created` | Usuário criado com sucesso. |
| `400 Bad Request` | Dados inválidos. |
| `403 Forbidden` | Usuário sem permissão. |
| `409 Conflict` | E-mail já cadastrado. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

⬆️ [Voltar ao índice](#indice)

<a id="listar-usuarios"></a>
## 📋 Listar usuários

### Objetivo

Listar os usuários cadastrados de forma paginada.

### Endpoint

```http
GET /api/v1/users
```

### Autenticação

Obrigatória (`Bearer Token`).

### Permissão

`ADMIN`

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| `Authorization` | Sim | `Bearer <access_token>` |

### Parâmetros de consulta

| Nome | Tipo | Obrigatório | Padrão | Descrição |
|------|------|:-----------:|--------|-----------|
| `page` | Integer | Não | `0` | Número da página. |
| `size` | Integer | Não | `10` | Quantidade de registros por página. |
| `sort` | String | Não | `name,asc` | Campo e direção da ordenação. |

### Resposta de sucesso

DTO: `Page<UserResponse>`

Exemplo:

```json
{
  "content": [
    {
      "id": "<user_uuid>",
      "name": "<name>",
      "email": "<email>",
      "status": "<status>",
      "role": "<role>",
      "createdAt": "<created_at>",
      "updatedAt": "<updated_at>"
    }
  ],
  "page": {
    "size": 10,
    "number": 0,
    "totalElements": "<total_elements>",
    "totalPages": "<total_pages>"
  }
}
```

### Regras de negócio

- Apenas usuários com papel `ADMIN` podem listar usuários.
- A listagem é paginada utilizando Spring Data.

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `200 OK` | Consulta realizada com sucesso. |
| `403 Forbidden` | Usuário sem permissão. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

⬆️ [Voltar ao índice](#indice)

<a id="buscar-usuario"></a>
## 🔍 Buscar usuário

### Objetivo

Consultar um usuário pelo identificador.

### Endpoint

```http
GET /api/v1/users/{id}
```

### Autenticação

Obrigatória (`Bearer Token`).

### Permissão

`ADMIN`

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| `Authorization` | Sim | `Bearer <access_token>` |

### Parâmetros de Caminho

| Nome | Tipo | Obrigatório | Descrição |
|------|------|:-----------:|-----------|
| `id` | UUID | Sim | Identificador do usuário. |

### Resposta de sucesso

DTO: `UserResponse`

Exemplo:

```json
{
  "id": "<user_uuid>",
  "name": "<name>",
  "email": "<email>",
  "status": "<status>",
  "role": "<role>",
  "createdAt": "<created_at>",
  "updatedAt": "<updated_at>"
}
```

### Regras de negócio

- Apenas usuários com papel `ADMIN` podem consultar usuários.
- O usuário deve existir.

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `200 OK` | Usuário encontrado. |
| `403 Forbidden` | Usuário sem permissão. |
| `404 Not Found` | Usuário não encontrado. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

⬆️ [Voltar ao índice](#indice)

<a id="atualizar-usuario"></a>
## ✏️ Atualizar usuário

### Objetivo

Atualizar os dados cadastrais de um usuário.

### Endpoint

```http
PUT /api/v1/users/{id}
```

### Autenticação

Obrigatória (`Bearer Token`).

### Permissão

`ADMIN`

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| `Authorization` | Sim | `Bearer <access_token>` |
| `Content-Type` | Sim | `application/json` |

### Parâmetros de caminho

| Nome | Tipo | Obrigatório | Descrição |
|------|------|:-----------:|-----------|
| `id` | UUID | Sim | Identificador do usuário. |

### Corpo da requisição

DTO: `UpdateUserRequest`

Exemplo:

```json
{
  "name": "<name>",
  "email": "<email>",
  "role": "<role>"
}
```

### Resposta de sucesso

DTO: `UserResponse`

Exemplo:

```json
{
  "id": "<user_uuid>",
  "name": "<name>",
  "email": "<email>",
  "status": "<status>",
  "role": "<role>",
  "createdAt": "<created_at>",
  "updatedAt": "<updated_at>"
}
```

### Regras de negócio

- Apenas usuários com papel `ADMIN` podem atualizar usuários.
- Apenas `name`, `email` e `role` podem ser alterados.
- O status do usuário não pode ser alterado neste endpoint.
- O e-mail deve permanecer único.

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `200 OK` | Usuário atualizado com sucesso. |
| `400 Bad Request` | Dados inválidos. |
| `403 Forbidden` | Usuário sem permissão. |
| `404 Not Found` | Usuário não encontrado. |
| `409 Conflict` | E-mail já utilizado. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

⬆️ [Voltar ao índice](#indice)

<a id="alterar-status-do-usuario"></a>
## 🔄 Alterar status do usuário

### Objetivo

Alterar o status de um usuário.

### Endpoint

```http
PATCH /api/v1/users/{id}/status
```

### Autenticação

Obrigatória (`Bearer Token`).

### Permissão

`ADMIN`

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| `Authorization` | Sim | `Bearer <access_token>` |
| `Content-Type` | Sim | `application/json` |

### Parâmetros de caminho

| Nome | Tipo | Obrigatório | Descrição |
|------|------|:-----------:|-----------|
| `id` | UUID | Sim | Identificador do usuário. |

### Corpo da requisição

DTO: `UpdateUserStatusRequest`

Exemplo:

```json
{
  "status": "<status>"
}
```

### Resposta de sucesso

DTO: `UserResponse`

Exemplo:

```json
{
  "id": "<user_uuid>",
  "name": "<name>",
  "email": "<email>",
  "status": "<status>",
  "role": "<role>",
  "createdAt": "<created_at>",
  "updatedAt": "<updated_at>"
}
```

### Regras de negócio

- Apenas usuários com papel `ADMIN` podem alterar o status de usuários.
- Os status permitidos são:
  - `ACTIVE`
  - `INACTIVE`
  - `BLOCKED`
- Apenas o status pode ser alterado por este endpoint.

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `200 OK` | Status alterado com sucesso. |
| `400 Bad Request` | Status inválido. |
| `403 Forbidden` | Usuário sem permissão. |
| `404 Not Found` | Usuário não encontrado. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

⬆️ [Voltar ao índice](#indice)
