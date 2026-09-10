# 4. Contrato da API

**Base:** `http://localhost:8080/api`
**Formato:** JSON em UTF-8, exceto os envios de imagem, que usam
`multipart/form-data`.

## 4.1 Autenticação

Endpoints marcados com 🔒 exigem o cabeçalho:

```
Authorization: Bearer <token>
```

O token é um JWT com validade de 24 horas, devolvido pelo cadastro e pelo login.
Sem token válido, esses endpoints respondem `401`.

Os endpoints de leitura são públicos, mas o comportamento muda quando há token:
com ele, cada publicação vem com o campo `curtidoPorMim` preenchido.

## 4.2 Resumo dos endpoints

### Autenticação

| Método | Rota | Descrição | Auth |
|---|---|---|---|
| `POST` | `/auth/cadastro` | Cria uma conta e já devolve o token | — |
| `POST` | `/auth/login` | Autentica e devolve o token | — |

### Publicações

| Método | Rota | Descrição | Auth |
|---|---|---|---|
| `GET` | `/posts` | Lista paginada. Aceita `?busca=`, `?pagina=`, `?tamanho=` | — |
| `GET` | `/posts/seguindo` | Publicações de quem o usuário segue e as próprias | 🔒 |
| `GET` | `/posts/usuario/{id}` | Publicações de um perfil | — |
| `GET` | `/posts/{id}` | Uma publicação | — |
| `POST` | `/posts` | Cria publicação (`multipart`: `conteudo`, `imagem` opcional) | 🔒 |
| `PUT` | `/posts/{id}` | Edita o texto da própria publicação | 🔒 |
| `DELETE` | `/posts/{id}` | Exclui a própria publicação | 🔒 |
| `POST` | `/posts/{id}/curtidas` | Curte | 🔒 |
| `DELETE` | `/posts/{id}/curtidas` | Descurte | 🔒 |

### Comentários

| Método | Rota | Descrição | Auth |
|---|---|---|---|
| `GET` | `/posts/{id}/comentarios` | Lista os comentários de uma publicação | — |
| `POST` | `/posts/{id}/comentarios` | Comenta | 🔒 |
| `DELETE` | `/comentarios/{id}` | Exclui (autor do comentário ou autor do post) | 🔒 |

### Usuários

| Método | Rota | Descrição | Auth |
|---|---|---|---|
| `GET` | `/usuarios/eu` | Dados do usuário autenticado | 🔒 |
| `GET` | `/usuarios/{id}` | Perfil público | — |
| `PUT` | `/usuarios/eu` | Atualiza nome e biografia | 🔒 |
| `POST` | `/usuarios/eu/avatar` | Envia foto de perfil (`multipart`: `imagem`) | 🔒 |
| `POST` | `/usuarios/{id}/seguidores` | Passa a seguir | 🔒 |
| `DELETE` | `/usuarios/{id}/seguidores` | Deixa de seguir | 🔒 |

## 4.3 Exemplos

### `POST /api/auth/cadastro`

```json
{
  "nomeCompleto": "Maria Aparecida Souza",
  "email": "maria@exemplo.com",
  "senha": "umaSenhaBoa123"
}
```

**201 Created**

```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "usuario": {
    "id": 1,
    "nomeCompleto": "Maria Aparecida Souza",
    "primeiroNome": "Maria",
    "email": "maria@exemplo.com",
    "bio": null,
    "avatarUrl": null
  }
}
```

**409 Conflict** — e-mail já cadastrado

```json
{
  "status": 409,
  "erro": "Conflito",
  "mensagem": "Este e-mail já está cadastrado. Tente entrar na sua conta.",
  "caminho": "/api/auth/cadastro",
  "momento": "2026-03-14T18:22:41Z"
}
```

### `POST /api/auth/login`

```json
{ "email": "maria@exemplo.com", "senha": "umaSenhaBoa123" }
```

**200 OK** — mesma resposta do cadastro.
**401 Unauthorized** — `"E-mail ou senha incorretos."`

> A mesma mensagem é usada para e-mail inexistente e para senha errada, de
> propósito: mensagens diferentes permitiriam descobrir quais e-mails estão
> cadastrados.

### `GET /api/posts?pagina=0&tamanho=10`

**200 OK**

```json
{
  "conteudo": [
    {
      "id": 42,
      "conteudo": "Consegui fazer chamada de vídeo com meus netos hoje!",
      "imagemUrl": "https://xxxx.supabase.co/storage/v1/object/public/comunidade/posts/8f2a.jpg",
      "criadoEm": "2026-03-14T18:10:00Z",
      "editado": false,
      "autor": {
        "id": 1,
        "nomeCompleto": "Maria Aparecida Souza",
        "primeiroNome": "Maria",
        "avatarUrl": null
      },
      "totalCurtidas": 7,
      "totalComentarios": 3,
      "curtidoPorMim": true,
      "souOAutor": false
    }
  ],
  "pagina": 0,
  "tamanho": 10,
  "totalElementos": 34,
  "totalPaginas": 4,
  "ultima": false
}
```

Os campos `curtidoPorMim` e `souOAutor` vêm `false` quando não há token — é o que
o frontend usa para decidir se mostra o menu de editar/excluir.

### `POST /api/posts`

`Content-Type: multipart/form-data`

| Campo | Tipo | Obrigatório |
|---|---|---|
| `conteudo` | texto, 1 a 500 caracteres | sim |
| `imagem` | arquivo JPG/PNG/WebP até 3MB | não |

**201 Created** — o objeto da publicação, no mesmo formato acima.

### `PUT /api/posts/{id}`

```json
{ "conteudo": "Texto corrigido da publicação" }
```

**200 OK** — publicação atualizada, com `editado: true`.
**403 Forbidden** — `"Você só pode editar suas próprias publicações."`

### `POST /api/posts/{id}/curtidas`

Sem corpo.

**200 OK**

```json
{ "totalCurtidas": 8, "curtidoPorMim": true }
```

Curtir duas vezes não gera erro — a resposta é a mesma. A operação é
idempotente, o que evita mensagens confusas se o usuário clicar duas vezes.

### `GET /api/usuarios/{id}`

**200 OK**

```json
{
  "id": 1,
  "nomeCompleto": "Maria Aparecida Souza",
  "primeiroNome": "Maria",
  "bio": "Aprendendo a mexer no computador aos 68 anos. Sem pressa!",
  "avatarUrl": null,
  "criadoEm": "2026-01-05T12:00:00Z",
  "totalPublicacoes": 12,
  "totalSeguidores": 4,
  "totalSeguindo": 9,
  "seguidoPorMim": false,
  "souEu": false
}
```

### `PUT /api/usuarios/eu`

```json
{
  "nomeCompleto": "Maria A. Souza",
  "bio": "Aprendendo sem pressa."
}
```

## 4.4 Códigos de status usados

| Código | Quando |
|---|---|
| `200 OK` | Leitura ou atualização bem-sucedida |
| `201 Created` | Recurso criado (cadastro, publicação, comentário) |
| `204 No Content` | Exclusão bem-sucedida |
| `400 Bad Request` | Dados inválidos ou regra de negócio violada |
| `401 Unauthorized` | Token ausente, expirado ou inválido; credenciais erradas |
| `403 Forbidden` | Autenticado, mas sem permissão sobre aquele recurso |
| `404 Not Found` | Recurso inexistente |
| `409 Conflict` | E-mail já cadastrado |
| `413 Payload Too Large` | Imagem acima de 3MB |
| `500 Internal Server Error` | Falha inesperada |

## 4.5 Formato de erro

Todos os erros seguem a mesma estrutura (seção 2.9 da arquitetura):

```json
{
  "status": 400,
  "erro": "Requisição inválida",
  "mensagem": "A publicação precisa ter algum texto.",
  "campos": { "conteudo": "não pode estar em branco" },
  "caminho": "/api/posts",
  "momento": "2026-03-14T18:22:41Z"
}
```

`campos` só aparece em erros de validação de formulário — é o que permite ao
frontend destacar o campo problemático em vez de mostrar só um aviso genérico.

## 4.6 Paginação

Todas as listagens de publicações usam os mesmos parâmetros:

| Parâmetro | Padrão | Descrição |
|---|---|---|
| `pagina` | `0` | Índice da página, começando em zero |
| `tamanho` | `10` | Itens por página, máximo 50 |
| `busca` | — | Só em `GET /posts`; filtra pelo texto |

O frontend começa em `pagina=0` e incrementa a cada clique em
"Ver mais publicações", parando quando `ultima` vier `true`.

## 4.7 CORS

A API aceita requisições das origens listadas na variável `CORS_ORIGENS`. Em
desenvolvimento, o padrão cobre `http://localhost:5500` e `http://127.0.0.1:5500`
(Live Server do VS Code). Em produção, a variável recebe o domínio do frontend
publicado.
