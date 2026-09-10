# 2. Arquitetura

## 2.1 Visão geral

A aplicação é dividida em duas partes independentes que se comunicam apenas por
HTTP/JSON:

- **Frontend** — páginas estáticas em HTML, CSS e JavaScript. Não tem etapa de
  build e não conhece o banco de dados. Só sabe conversar com a API.
- **Backend** — uma API REST em Java com Spring Boot. É a única parte que fala
  com o banco e a única que decide o que cada usuário pode fazer.

Essa separação é proposital: o frontend pode ser publicado em qualquer
hospedagem estática e o backend em qualquer servidor Java, sem que um dependa da
infraestrutura do outro.

```mermaid
flowchart TB
    subgraph cliente ["Navegador"]
        html["Paginas HTML/CSS<br/>login, cadastro, inicio, perfil"]
        js["Modulos JavaScript<br/>api.js, sessao.js, ui.js"]
        html --- js
    end

    subgraph servidor ["API REST - Spring Boot"]
        ctrl["Controllers<br/>recebem HTTP"]
        svc["Services<br/>regras de negocio"]
        repo["Repositories<br/>acesso a dados"]
        sec["Security<br/>filtro JWT"]
        ctrl --> svc
        svc --> repo
        sec -.->|autentica| ctrl
    end

    subgraph supabase ["Supabase"]
        pg[("PostgreSQL")]
        storage[["Storage<br/>bucket de imagens"]]
    end

    js -->|"JSON + Bearer token"| ctrl
    repo -->|JDBC| pg
    svc -->|"HTTP - upload"| storage
    html -->|"URL publica da imagem"| storage
```

## 2.2 Organização em camadas

O backend segue a divisão clássica em camadas. Cada camada só conhece a de
baixo, e os dados que atravessam a fronteira HTTP são sempre DTOs — as entidades
JPA nunca são serializadas diretamente.

```mermaid
flowchart TD
    A["Controller<br/><i>traduz HTTP para chamada de metodo</i>"]
    B["DTO<br/><i>contrato de entrada e saida</i>"]
    C["Service<br/><i>regras de negocio e autorizacao</i>"]
    D["Repository<br/><i>consultas ao banco</i>"]
    E["Entity<br/><i>mapeamento das tabelas</i>"]

    A -->|usa| B
    A -->|chama| C
    C -->|usa| D
    D -->|retorna| E
    C -->|converte Entity em| B
```

**Por que DTOs e não as entidades direto:** a entidade `Usuario` guarda o
`senhaHash`. Se ela fosse serializada como resposta JSON, o hash vazaria para o
navegador. O DTO obriga a escolher explicitamente o que sai.

### Responsabilidade de cada camada

| Camada | Faz | Não faz |
|---|---|---|
| Controller | Lê a requisição, chama o service, devolve o status HTTP | Regra de negócio, acesso a banco |
| Service | Valida regras, verifica permissão, orquestra repositórios | Conhecer HTTP (`HttpServletRequest`, status code) |
| Repository | Consulta e grava no banco | Decidir se a operação é permitida |
| Entity | Representa uma tabela | Lógica de aplicação |

## 2.3 Estrutura de pastas

```
comunidade/
├── docs/                      # esta documentação
├── backend/
│   ├── pom.xml
│   └── src/main/
│       ├── java/com/comunidade/
│       │   ├── ComunidadeApplication.java
│       │   ├── config/        # CORS, beans de configuração
│       │   ├── security/      # filtro JWT, geração e validação de token
│       │   ├── model/         # entidades JPA
│       │   ├── repository/    # interfaces Spring Data
│       │   ├── service/       # regras de negócio
│       │   ├── controller/    # endpoints REST
│       │   ├── dto/           # records de entrada e saída
│       │   └── exception/     # exceções e handler global
│       └── resources/
│           ├── application.properties
│           └── db/schema.sql  # modelo documentado em SQL
└── frontend/
    ├── index.html             # início (lista de publicações)
    ├── login.html
    ├── cadastro.html
    ├── perfil.html
    ├── css/
    │   └── style.css
    └── js/
        ├── config.js          # endereço da API
        ├── api.js             # wrapper de fetch com token
        ├── sessao.js          # token e usuário no localStorage
        ├── ui.js              # notificações, modais, datas, escape
        ├── componentes.js     # montagem dos cards
        ├── inicio.js
        ├── login.js
        ├── cadastro.js
        └── perfil.js
```

## 2.4 Diagrama de classes — domínio

```mermaid
classDiagram
    class Usuario {
        -Long id
        -String nomeCompleto
        -String email
        -String senhaHash
        -String bio
        -String avatarUrl
        -Instant criadoEm
    }

    class Post {
        -Long id
        -Usuario autor
        -String conteudo
        -String imagemUrl
        -Instant criadoEm
        -Instant atualizadoEm
        +boolean foiEditado()
    }

    class Comentario {
        -Long id
        -Post post
        -Usuario autor
        -String conteudo
        -Instant criadoEm
    }

    class Curtida {
        -Long id
        -Post post
        -Usuario usuario
        -Instant criadoEm
    }

    class Seguidor {
        -Long id
        -Usuario seguidor
        -Usuario seguido
        -Instant criadoEm
    }

    Usuario "1" --> "0..*" Post : escreve
    Usuario "1" --> "0..*" Comentario : escreve
    Post    "1" --> "0..*" Comentario : recebe
    Usuario "1" --> "0..*" Curtida : registra
    Post    "1" --> "0..*" Curtida : recebe
    Usuario "1" --> "0..*" Seguidor : segue
    Usuario "1" --> "0..*" Seguidor : e seguido por
```

`Curtida` e `Seguidor` são tabelas de associação com identidade própria. Poderiam
ser modeladas como `@ManyToMany`, mas viram entidades para que possam guardar a
data em que aconteceram e para deixar as consultas de contagem mais simples.

## 2.5 Diagrama de classes — camadas de serviço

Recorte da camada de aplicação, mostrando as dependências entre os componentes
do backend (omitidos os DTOs para não poluir).

```mermaid
classDiagram
    direction LR

    class AutenticacaoController {
        +cadastrar(CadastroRequest) TokenResponse
        +entrar(LoginRequest) TokenResponse
    }
    class PostController {
        +listar(busca, pagina) PaginaResponse
        +listarSeguindo(pagina) PaginaResponse
        +criar(conteudo, imagem) PostResponse
        +editar(id, EdicaoRequest) PostResponse
        +excluir(id) void
        +curtir(id) ContagemResponse
        +descurtir(id) ContagemResponse
    }
    class ComentarioController {
        +listarDoPost(postId) List~ComentarioResponse~
        +criar(postId, req) ComentarioResponse
        +excluir(id) void
    }
    class UsuarioController {
        +meuPerfil() PerfilResponse
        +porId(id) PerfilResponse
        +atualizarPerfil(req) PerfilResponse
        +atualizarAvatar(arquivo) PerfilResponse
        +seguir(id) void
        +deixarDeSeguir(id) void
    }

    class AutenticacaoService
    class PostService
    class ComentarioService
    class UsuarioService
    class ArmazenamentoService {
        +enviarImagem(MultipartFile, pasta) String
    }
    class TokenService {
        +gerar(Usuario) String
        +extrairEmail(String) String
        +valido(String) boolean
    }

    class UsuarioRepository
    class PostRepository
    class ComentarioRepository
    class CurtidaRepository
    class SeguidorRepository

    AutenticacaoController --> AutenticacaoService
    PostController --> PostService
    ComentarioController --> ComentarioService
    UsuarioController --> UsuarioService

    AutenticacaoService --> UsuarioRepository
    AutenticacaoService --> TokenService
    PostService --> PostRepository
    PostService --> CurtidaRepository
    PostService --> ComentarioRepository
    PostService --> SeguidorRepository
    PostService --> ArmazenamentoService
    ComentarioService --> ComentarioRepository
    ComentarioService --> PostRepository
    UsuarioService --> UsuarioRepository
    UsuarioService --> SeguidorRepository
    UsuarioService --> ArmazenamentoService
```

## 2.6 Fluxo de autenticação

O login acontece uma vez; a partir daí toda requisição carrega o token. O
servidor não guarda sessão — ele valida a assinatura do token a cada chamada.

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuario
    participant F as Frontend
    participant C as AutenticacaoController
    participant S as AutenticacaoService
    participant R as UsuarioRepository
    participant T as TokenService

    U->>F: preenche e-mail e senha
    F->>C: POST /api/auth/login
    C->>S: entrar(email, senha)
    S->>R: findByEmail(email)
    R-->>S: Usuario
    S->>S: BCrypt.matches(senha, senhaHash)
    alt senha correta
        S->>T: gerar(usuario)
        T-->>S: JWT assinado
        S-->>C: TokenResponse
        C-->>F: 200 OK + token + dados do usuario
        F->>F: salva token no localStorage
        F-->>U: leva para o Inicio
    else senha incorreta
        S-->>C: CredenciaisInvalidasException
        C-->>F: 401 + "E-mail ou senha incorretos"
        F-->>U: mostra a mensagem no formulario
    end
```

### Requisição autenticada

```mermaid
sequenceDiagram
    autonumber
    participant F as Frontend
    participant Fi as FiltroJwt
    participant C as PostController
    participant S as PostService

    F->>Fi: POST /api/posts (Authorization: Bearer ...)
    Fi->>Fi: le o cabecalho e valida a assinatura
    alt token valido
        Fi->>Fi: coloca o usuario no SecurityContext
        Fi->>C: segue a requisicao
        C->>S: criar(usuarioAtual, conteudo)
        S-->>C: PostResponse
        C-->>F: 201 Created
    else token ausente ou invalido
        Fi-->>F: 401 Unauthorized
    end
```

## 2.7 Fluxo de publicação com imagem

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuario
    participant F as Frontend
    participant C as PostController
    participant S as PostService
    participant A as ArmazenamentoService
    participant SB as Supabase Storage
    participant DB as PostgreSQL

    U->>F: escreve o texto e escolhe a foto
    F->>F: valida tamanho (max 3MB) e tipo
    F->>C: POST /api/posts (multipart)
    C->>S: criar(usuario, conteudo, arquivo)
    S->>S: valida conteudo (RN03)
    S->>A: enviarImagem(arquivo, "posts")
    A->>SB: POST /storage/v1/object/comunidade/posts/uuid.jpg
    SB-->>A: 200 OK
    A-->>S: URL publica
    S->>DB: INSERT INTO posts (autor_id, conteudo, imagem_url)
    DB-->>S: post salvo
    S-->>C: PostResponse
    C-->>F: 201 Created
    F-->>U: publicacao aparece no topo da lista
```

**Por que a imagem não vai para o banco:** a versão original guardava a foto como
Base64 dentro do documento. Isso aumenta o tamanho do registro em cerca de 33%
sobre o arquivo original, torna cada consulta ao feed muito mais pesada e impede
que o navegador use cache de imagem. Guardar o arquivo no Storage e apenas a URL
no banco resolve os três problemas.

## 2.8 Decisões técnicas

| # | Decisão | Alternativas consideradas | Motivo |
|---|---|---|---|
| D01 | **Java 21 + Spring Boot 3** no backend | Node.js/Express, Quarkus | Java é o objetivo de aprendizado; Spring Boot é o padrão de mercado e o que aparece em vagas júnior |
| D02 | **PostgreSQL** (Supabase) | Firebase Firestore, MySQL | O projeto tem relacionamentos claros (autor, comentários, seguidores). Com Firestore, o backend Java viraria um proxy sobre NoSQL, sem JPA nem integridade referencial |
| D03 | **Spring Data JPA** | JDBC puro, MyBatis | Menos código repetitivo para CRUD; JPQL cobre as consultas mais complexas do projeto |
| D04 | **JWT próprio** com Spring Security | Sessão HTTP, Supabase Auth | API sem estado, frontend em outro domínio, e o fluxo completo de autenticação fica visível no código |
| D05 | **Supabase Storage** para imagens | Base64 no banco, disco local | Ver seção 2.7. Disco local quebra em hospedagens de sistema de arquivos efêmero |
| D06 | **Frontend sem build** (HTML + módulos ES6) | React, Vue, Vite | Requisito do projeto; mantém o foco no backend e demonstra JavaScript sem intermediários |
| D07 | **Páginas separadas** em vez de SPA | Roteamento no cliente | Cada tela tem uma URL real, o botão voltar funciona e leitores de tela anunciam a mudança de página — relevante dado o público |
| D08 | **`records` do Java** para os DTOs | Classes com getters, Lombok | Imutáveis, sem boilerplate e sem dependência extra nem configuração de IDE |
| D09 | **Paginação com botão "Ver mais"** | Rolagem infinita | Rolagem infinita desorienta e nunca deixa chegar ao fim da página; o botão dá controle ao usuário |

## 2.9 Tratamento de erros

Todo erro da API sai no mesmo formato, tratado por um `@RestControllerAdvice`
central. Nenhum controller escreve `try/catch` para isso.

```json
{
  "status": 400,
  "erro": "Requisição inválida",
  "mensagem": "A publicação precisa ter algum texto.",
  "campos": {
    "conteudo": "não pode estar em branco"
  },
  "caminho": "/api/posts",
  "momento": "2026-03-14T18:22:41Z"
}
```

| Situação | Exceção | HTTP |
|---|---|---|
| Dados inválidos no corpo | `MethodArgumentNotValidException` | 400 |
| Regra de negócio violada | `RegraDeNegocioException` | 400 |
| Sem token ou token inválido | `AuthenticationException` | 401 |
| Autenticado, mas não é o dono | `AcessoNegadoException` | 403 |
| Recurso não existe | `RecursoNaoEncontradoException` | 404 |
| E-mail já cadastrado | `ConflitoException` | 409 |
| Arquivo acima do limite | `MaxUploadSizeExceededException` | 413 |
| Falha inesperada | `Exception` | 500 |

O `campos` só aparece em erros de validação. A `mensagem` é sempre escrita em
português e em linguagem simples, porque vai direto para a tela do usuário
(princípio 4 da seção 1.4).

## 2.10 Configuração e segredos

Nada sensível entra no repositório. O `application.properties` só referencia
variáveis de ambiente:

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
app.jwt.secret=${JWT_SECRET}
app.storage.service-key=${SUPABASE_SERVICE_KEY}
```

O repositório traz um `.env.example` com os nomes das variáveis e valores de
exemplo. O `.env` real fica no `.gitignore`.

> A chave `service_role` do Supabase ignora as políticas de segurança do banco.
> Ela vive apenas no servidor e **nunca** pode aparecer no frontend.
