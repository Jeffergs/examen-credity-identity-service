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
