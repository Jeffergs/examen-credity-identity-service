# Identity Service

## Objetivo

O Identity Service é responsável por gerenciar a identidade dos usuários da plataforma, realizando autenticação, autorização e controle de acesso aos recursos protegidos.

Além disso, é responsável pela emissão e validação de tokens JWT, administração de usuários, papéis (roles) e permissões, fornecendo os mecanismos de segurança utilizados pelos demais microsserviços.

---

## Responsabilidades

O Identity Service é responsável por:

- Autenticar usuários.
- Emitir tokens JWT.
- Renovar tokens de acesso.
- Invalidar tokens quando aplicável.
- Gerenciar usuários.
- Gerenciar papéis (roles).
- Gerenciar permissões.
- Fornecer informações de identidade para os demais microsserviços.

---

## Domínios Funcionais

| Domínio Funcional | Responsabilidade |
|-------------------|------------------|
| Autenticação | Autenticar usuários, emitir, renovar e invalidar tokens de acesso. |
| Usuários | Gerenciar contas de usuários da plataforma. |
| Papéis (Roles) | Gerenciar os papéis atribuídos aos usuários. |
| Permissões | Gerenciar as permissões associadas aos papéis. |
| Autorização | Controlar o acesso aos recursos protegidos com base nas permissões do usuário. |

---

## Fora do Escopo

O Identity Service não é responsável por:

- Cadastro de clientes.
- Cadastro de renda.
- Análise de crédito.
- Auditoria.
- Notificações.

## Login

### Objetivo

Permitir que um usuário autenticado acesse a plataforma por meio da validação de suas credenciais, recebendo um token JWT para utilização nos recursos protegidos.

---

### Descrição

O usuário informa seu e-mail e senha.

O Identity Service valida as credenciais informadas e, caso sejam válidas, autentica o usuário e retorna um token JWT contendo suas informações de identidade e autorização.

Esse token deverá ser enviado nas requisições subsequentes para acesso aos endpoints protegidos da plataforma.

---

### Entradas

| Campo | Obrigatório | Descrição |
|--------|:-----------:|-----------|
| E-mail | Sim | Endereço de e-mail do usuário. |
| Senha | Sim | Senha cadastrada para o usuário. |

---

### Regras de Negócio

- O e-mail deve estar cadastrado.
- O usuário deve estar ativo.
- A senha informada deve corresponder à senha armazenada.
- A senha deve ser comparada utilizando seu hash armazenado.
- Apenas usuários autenticados podem receber um token de acesso.

---

### Fluxo

1. O usuário informa e-mail e senha.
2. O Identity Service valida os dados recebidos.
3. O sistema localiza o usuário pelo e-mail.
4. O sistema verifica se o usuário está ativo.
5. O sistema valida a senha informada.
6. O sistema gera um token JWT.
7. O token é retornado ao cliente.

---

### Resultado Esperado

Em caso de sucesso, o usuário é autenticado e recebe um token JWT válido para acessar os recursos protegidos da plataforma.

---

### Possíveis Erros

| Situação | Resultado |
|----------|-----------|
| E-mail não cadastrado | Autenticação negada. |
| Senha inválida | Autenticação negada. |
| Usuário inativo | Autenticação negada. |
| Credenciais inválidas | Autenticação negada. |

## Renovar Access Token

### Objetivo

Emitir um novo Access Token a partir de um Refresh Token válido, sem exigir uma nova autenticação.

---

### Endpoint

```http
POST /api/v1/auth/refresh
```

---

### Autenticação

Não requerida.

---

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| Content-Type | Sim | `application/json` |

---

### Corpo da Requisição

DTO: `RefreshTokenRequest`

| Campo | Tipo | Obrigatório | Descrição |
|--------|------|:-----------:|-----------|
| `refreshToken` | String | Sim | Refresh Token válido emitido durante a autenticação. |

Exemplo:

```json
{
  "refreshToken": "<refresh_token>"
}
```

---

### Resposta de Sucesso

DTO: `RefreshTokenResponse`

```json
{
  "accessToken": "<novo_access_token>",
  "refreshToken": "<novo_refresh_token>",
  "tokenType": "Bearer",
  "expiresIn": 900
}
```

---

### Regras de Negócio

- O Refresh Token deve ser válido.
- O Refresh Token não pode estar expirado.
- O Refresh Token não pode estar revogado.
- Um novo Access Token é emitido para o usuário autenticado.
- Um novo Refresh Token é emitido e o token anterior é invalidado (Refresh Token Rotation).

---

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `200 OK` | Novo Access Token emitido com sucesso. |
| `400 Bad Request` | Requisição inválida. |
| `401 Unauthorized` | Refresh Token inválido, expirado ou revogado. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

### Observação

O Identity Service adota a estratégia de **Refresh Token Rotation**. A cada renovação, um novo Refresh Token é emitido e o token utilizado na requisição é invalidado imediatamente.

Os detalhes sobre expiração, revogação, rotação e blacklist de tokens estão descritos em **09-arquitetura-de-seguranca.md**.




## Logout

### Objetivo

Encerrar a sessão do usuário autenticado, revogando o Refresh Token e impedindo a emissão de novos Access Tokens.

---

### Endpoint

```http
POST /api/v1/auth/logout
```

---

### Autenticação

Obrigatória (`Bearer Token`).

---

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| Authorization | Sim | `Bearer <access_token>` |
| Content-Type | Sim | `application/json` |

---

### Corpo da Requisição

DTO: `RefreshTokenRequest`

| Campo | Tipo | Obrigatório | Descrição |
|--------|------|:-----------:|-----------|
| `refreshToken` | String | Sim | Refresh Token que será revogado. |

Exemplo:

```json
{
  "refreshToken": "<refresh_token>"
}
```

---

### Resposta de Sucesso

DTO: `MessageResponse`

```json
{
  "message": "Logout realizado com sucesso."
}
```

---

### Regras de Negócio

- O Access Token deve ser válido.
- O Refresh Token deve pertencer ao usuário autenticado.
- O Refresh Token é revogado e não pode mais ser reutilizado.
- Após o logout, o usuário deverá realizar uma nova autenticação para obter novos tokens.

---

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `200 OK` | Logout realizado com sucesso. |
| `400 Bad Request` | Requisição inválida. |
| `401 Unauthorized` | Access Token inválido ou expirado. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

### Observação

O logout revoga apenas o Refresh Token. O Access Token permanece válido até sua expiração natural.

As políticas de expiração, revogação e gerenciamento de tokens são descritas em **09-arquitetura-de-seguranca.md**.







## 5.2.1 Criar Usuário

### Objetivo

Cadastrar um novo usuário na plataforma.

---

### Endpoint

```http
POST /api/v1/users
```

---

### Autenticação

Obrigatória (`Bearer Token`).

---

### Permissão

`USER_CREATE`

---

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| Authorization | Sim | `Bearer <access_token>` |
| Content-Type | Sim | `application/json` |

---

### Corpo da Requisição

DTO: `CreateUserRequest`

| Campo | Tipo | Obrigatório | Descrição |
|--------|------|:-----------:|-----------|
| `name` | String | Sim | Nome completo do usuário. |
| `email` | String | Sim | Endereço de e-mail. |
| `password` | String | Sim | Senha de acesso. |
| `roleIds` | List<UUID> | Sim | Identificadores dos papéis atribuídos ao usuário. |

Exemplo:

```json
{
  "name": "João Silva",
  "email": "joao.silva@email.com",
  "password": "Senha@123",
  "roleIds": [
    "3e0d4bb2-6bc9-4f6b-9cf7-8a6fd90d5e11"
  ]
}
```

---

### Resposta de Sucesso

DTO: `UserResponse`

```json
{
  "id": "efbdf77f-24dd-45b3-a130-3e60ef7b1f6a",
  "name": "João Silva",
  "email": "joao.silva@email.com",
  "roles": [
    {
      "id": "3e0d4bb2-6bc9-4f6b-9cf7-8a6fd90d5e11",
      "name": "ADMIN"
    }
  ],
  "active": true,
  "createdAt": "2026-08-15T14:30:00Z",
  "updatedAt": "2026-08-15T14:30:00Z"
}
```

---

### Regras de Negócio

- O e-mail deve ser único na plataforma.
- A senha deve atender à política de segurança da aplicação.
- Todos os papéis informados devem existir.
- O usuário será associado aos papéis informados.
- Todo usuário é criado com status **ativo**.
- A senha é armazenada apenas em formato criptografado.

---

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `201 Created` | Usuário criado com sucesso. |
| `400 Bad Request` | Requisição inválida. |
| `401 Unauthorized` | Usuário não autenticado. |
| `403 Forbidden` | Usuário sem permissão para criar usuários. |
| `409 Conflict` | Já existe um usuário com o e-mail informado. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

### Observação

As políticas de senha, criptografia, autenticação e controle de acesso estão descritas em **09-arquitetura-de-seguranca.md**.

⬆️ [Voltar ao índice](#indice)






## 5.2.2 Listar Usuários

### Objetivo

Retornar uma lista paginada de usuários cadastrados na plataforma.

---

### Endpoint

```http
GET /api/v1/users
```

---

### Autenticação

Obrigatória (`Bearer Token`).

---

### Permissão

`USER_READ`

---

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| Authorization | Sim | `Bearer <access_token>` |

---

### Parâmetros de Consulta

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|:-----------:|-----------|
| `page` | Integer | Não | Número da página. Padrão: `0`. |
| `size` | Integer | Não | Quantidade de registros por página. Padrão: `20`. |
| `sort` | String | Não | Campo e direção da ordenação (`campo,asc` ou `campo,desc`). |

> Os campos permitidos para ordenação são: `name`, `email`, `active`, `createdAt` e `updatedAt`.

Exemplo:

```http
GET /api/v1/users?page=0&size=20&sort=name,asc
Authorization: Bearer <access_token>
```

---

### Resposta de Sucesso

DTO: `PageResponse<UserResponse>`

```json
{
  "content": [
    {
      "id": "efbdf77f-24dd-45b3-a130-3e60ef7b1f6a",
      "name": "João Silva",
      "email": "joao.silva@email.com",
      "active": true
    },
    {
      "id": "0fd9ddc5-12f0-42cb-bfc7-3dc6dbf2c26f",
      "name": "Maria Oliveira",
      "email": "maria.oliveira@email.com",
      "active": true
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 2,
  "totalPages": 1
}
```

---

### Regras de Negócio

- Apenas usuários autenticados com a permissão `USER_READ` podem consultar usuários.
- O resultado é retornado de forma paginada.
- A ordenação deve utilizar apenas os campos permitidos pela API.

---

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `200 OK` | Lista de usuários retornada com sucesso. |
| `401 Unauthorized` | Usuário não autenticado. |
| `403 Forbidden` | Usuário sem permissão para consultar usuários. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

### Observação

Os padrões de paginação, ordenação e filtros estão definidos em **12-convencoes-do-projeto.md**.

⬆️ [Voltar ao índice](#indice)






## 5.2.3 Buscar Usuário por ID

### Objetivo

Retornar os dados de um usuário a partir do seu identificador.

---

### Endpoint

```http
GET /api/v1/users/{id}
```

---

### Autenticação

Obrigatória (`Bearer Token`).

---

### Permissão

`USER_READ`

---

### Cabeçalhos

| Nome | Obrigatório | Valor |
|------|:-----------:|-------|
| Authorization | Sim | `Bearer <access_token>` |

---

### Parâmetros de Caminho

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|:-----------:|-----------|
| `id` | UUID | Sim | Identificador único do usuário. |

Exemplo:

```http
GET /api/v1/users/efbdf77f-24dd-45b3-a130-3e60ef7b1f6a
Authorization: Bearer <access_token>
```

---

### Resposta de Sucesso

DTO: `UserResponse`

```json
{
  "id": "efbdf77f-24dd-45b3-a130-3e60ef7b1f6a",
  "name": "João Silva",
  "email": "joao.silva@email.com",
  "roles": [
    {
      "id": "3e0d4bb2-6bc9-4f6b-9cf7-8a6fd90d5e11",
      "name": "ADMIN"
    }
  ],
  "active": true,
  "createdAt": "2026-08-15T14:30:00Z",
  "updatedAt": "2026-08-15T14:30:00Z"
}
```

---

### Regras de Negócio

- Apenas usuários autenticados com a permissão `USER_READ` podem consultar usuários.
- O identificador informado deve corresponder a um usuário existente.

---

### Códigos HTTP

| Código | Descrição |
|---------|-----------|
| `200 OK` | Usuário encontrado com sucesso. |
| `401 Unauthorized` | Usuário não autenticado. |
| `403 Forbidden` | Usuário sem permissão para consultar usuários. |
| `404 Not Found` | Usuário não encontrado. |
| `500 Internal Server Error` | Erro interno do servidor. |

---

### Observação

O identificador do usuário é um UUID gerado automaticamente pela plataforma e não pode ser alterado.

⬆️ [Voltar ao índice](#indice)
