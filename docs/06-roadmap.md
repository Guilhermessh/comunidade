# 6. Roadmap de Desenvolvimento

O projeto é construído em etapas pequenas. Cada etapa é um conjunto coerente de
mudanças que **funciona por si só** e vira um ou poucos commits. A ideia é que o
histórico do repositório conte a história da construção, em vez de aparecer como
um único despejo de código.

## 6.1 Convenção de commits

Formato [Conventional Commits](https://www.conventionalcommits.org/pt-br/):

```
<tipo>(<escopo>): <descrição no imperativo, em minúsculas>
```

| Tipo | Quando usar |
|---|---|
| `feat` | Nova funcionalidade |
| `fix` | Correção de defeito |
| `docs` | Só documentação |
| `style` | Formatação, sem mudar comportamento |
| `refactor` | Reorganização sem mudar comportamento |
| `chore` | Configuração, dependências, build |

Escopos usados: `escopo`, `arquitetura`, `design`, `api`, `auth`, `posts`,
`comentarios`, `perfil`, `storage`, `frontend`, `a11y`.

Exemplos:

```
docs(escopo): adiciona escopo, requisitos e casos de uso
feat(auth): implementa cadastro e login com JWT
fix(posts): corrige ordenacao do feed por data
```

## 6.2 Estratégia de branches

Fluxo simples, adequado a um projeto individual:

- `main` — sempre funcional. Nada é commitado direto aqui.
- `feat/<assunto>` — uma branch por etapa, aberta a partir de `main`.
- Ao terminar a etapa, abre-se um Pull Request de `feat/<assunto>` para `main`.

Mesmo trabalhando sozinho, vale abrir PR: o histórico fica navegável e a
descrição do PR documenta o que foi feito. É também o que um recrutador procura.

## 6.3 Etapas

Legenda: ⬜ pendente · 🟨 em andamento · ✅ concluída

---

### ✅ Etapa 0 — Documentação e planejamento

**Branch:** `docs/planejamento` · **Entrega:** a pasta `docs/` e o `README.md`

- [x] Escopo, requisitos e casos de uso
- [x] Arquitetura, diagramas UML e decisões técnicas
- [x] Modelo de dados e diagrama ER
- [x] Contrato da API
- [x] Guia de estilo e wireframes
- [x] Este roadmap
- [x] `README.md` e `.gitignore`

```
docs: adiciona documentacao de escopo, arquitetura e design
```

---

### ⬜ Etapa 1 — Esqueleto do backend

**Branch:** `feat/backend-base`
**Objetivo:** a aplicação sobe, conecta no banco e cria as tabelas.

- [ ] `pom.xml` com Spring Boot, JPA, Security, Validation, Postgres e JJWT
- [ ] `application.properties` lendo variáveis de ambiente
- [ ] `.env.example` com os nomes das variáveis
- [ ] Entidades: `Usuario`, `Post`, `Comentario`, `Curtida`, `Seguidor`
- [ ] Repositórios Spring Data
- [ ] Projeto Supabase criado e conexão validada

**Como saber que terminou:** `mvn spring-boot:run` sobe sem erro e as cinco
tabelas aparecem no painel do Supabase.

```
chore(backend): configura projeto spring boot e conexao com postgres
feat(backend): adiciona entidades jpa e repositorios
```

---

### ⬜ Etapa 2 — Autenticação

**Branch:** `feat/auth`
**Objetivo:** cadastrar, entrar e proteger rotas. (RF01–RF04, RNF08–RNF10)

- [ ] `TokenService` — geração e validação de JWT
- [ ] `FiltroJwt` — lê o cabeçalho e popula o `SecurityContext`
- [ ] `SecurityConfig` — rotas públicas e protegidas, `BCryptPasswordEncoder`, CORS
- [ ] `AutenticacaoService` e `AutenticacaoController`
- [ ] DTOs: `CadastroRequest`, `LoginRequest`, `TokenResponse`
- [ ] Handler global de exceções com o formato de erro padrão

**Como saber que terminou:** cadastro e login funcionam no Postman/Insomnia; uma
rota protegida devolve 401 sem token e 200 com token.

```
feat(auth): implementa cadastro e login com jwt e bcrypt
feat(api): adiciona tratamento global de erros
```

---

### ⬜ Etapa 3 — Publicações

**Branch:** `feat/posts`
**Objetivo:** CRUD completo de publicações. (RF05, RF07–RF10, RF20)

- [ ] `PostService` com validações e verificação de propriedade (RNF10)
- [ ] `PostController` com listagem paginada, busca, criação, edição e exclusão
- [ ] DTOs de post e o `PaginaResponse` genérico
- [ ] Consulta agrupada de contagens, sem N+1 (seção 3.3)

**Como saber que terminou:** dá para criar, listar, buscar, editar e excluir; e
tentar editar a publicação de outra pessoa devolve 403.

```
feat(posts): implementa crud de publicacoes com paginacao e busca
```

---

### ⬜ Etapa 4 — Curtidas e comentários

**Branch:** `feat/interacoes`
**Objetivo:** interação entre membros. (RF11–RF14)

- [ ] Curtir e descurtir, idempotentes (RN04)
- [ ] Listar, criar e excluir comentários
- [ ] Autor do post pode excluir comentários na própria publicação (RF14)
- [ ] `curtidoPorMim` e contagens nos DTOs de post

```
feat(posts): adiciona curtidas
feat(comentarios): implementa comentarios em publicacoes
```

---

### ⬜ Etapa 5 — Perfil e seguidores

**Branch:** `feat/perfil`
**Objetivo:** perfis públicos e feed personalizado. (RF15–RF19)

- [ ] `UsuarioService` e `UsuarioController`
- [ ] Perfil público com contagens
- [ ] Editar nome e biografia
- [ ] Seguir e deixar de seguir, com a regra de não seguir a si mesmo (RN05)
- [ ] `GET /posts/seguindo`

```
feat(perfil): adiciona perfil publico e edicao de dados
feat(perfil): implementa seguir usuarios e feed personalizado
```

---

### ⬜ Etapa 6 — Envio de imagens

**Branch:** `feat/storage`
**Objetivo:** fotos em publicações e avatares. (RF06, RNF15)

- [ ] Bucket público criado no Supabase Storage
- [ ] `ArmazenamentoService` enviando via API REST do Supabase
- [ ] Validação de tipo e tamanho no servidor
- [ ] `POST /posts` aceitando `multipart`
- [ ] `POST /usuarios/eu/avatar`

```
feat(storage): integra upload de imagens com supabase storage
```

---

### ⬜ Etapa 7 — Base do frontend

**Branch:** `feat/frontend-base`
**Objetivo:** design system em CSS e as telas de entrada. (RF01, RF02, RNF01–RNF07)

- [ ] `css/style.css` com os tokens da seção 5.10
- [ ] `js/config.js`, `js/api.js`, `js/sessao.js`, `js/ui.js`
- [ ] `login.html` e `cadastro.html` ligados à API
- [ ] Cabeçalho comum com controle de tamanho de fonte

```
feat(frontend): cria design system em css com tokens acessiveis
feat(frontend): implementa telas de entrar e criar conta
```

---

### ⬜ Etapa 8 — Início

**Branch:** `feat/frontend-inicio`
**Objetivo:** a tela principal. (RF05–RF14, RF20)

- [ ] Lista paginada com "Ver mais publicações"
- [ ] Caixa de publicar com contador e pré-visualização de imagem
- [ ] Card de publicação com curtir, comentar, editar e excluir
- [ ] Busca
- [ ] Abas "Todas as publicações" e "De quem eu sigo"
- [ ] Escape de HTML em todo conteúdo de usuário (RNF12)

```
feat(frontend): implementa tela de inicio com publicacoes e interacoes
```

---

### ⬜ Etapa 9 — Perfil no frontend

**Branch:** `feat/frontend-perfil`
**Objetivo:** ver e editar perfil. (RF15–RF19)

- [ ] `perfil.html` com dados, contagens e publicações
- [ ] Botão seguir / deixar de seguir
- [ ] Edição de nome, biografia e foto no próprio perfil

```
feat(frontend): adiciona pagina de perfil com seguir e edicao
```

---

### ⬜ Etapa 10 — Acessibilidade e acabamento

**Branch:** `feat/acessibilidade`
**Objetivo:** fechar a versão 1.

- [ ] Percorrer o checklist da seção 5.9
- [ ] Testar navegação apenas por teclado
- [ ] Testar com zoom de 200%
- [ ] Revisar todos os textos conforme a seção 5.8
- [ ] Estados de vazio e de carregamento em todas as listas
- [ ] README final com prints e instruções de execução

```
feat(a11y): aplica correcoes de acessibilidade do checklist wcag
docs: adiciona prints e instrucoes de execucao ao readme
```

---

## 6.4 Depois da versão 1

Ideias registradas, fora do escopo atual (seção 1.9). Servem para mostrar que o
que ficou de fora foi decisão, não esquecimento.

| Ideia | Por que é interessante |
|---|---|
| Testes com JUnit e MockMvc | Cobrir services e controllers |
| Documentação com Swagger | `springdoc-openapi` em uma dependência |
| Docker Compose | Subir Postgres e API com um comando |
| Publicação online | Backend no Render, frontend no Vercel |
| Busca em texto completo | `tsvector` + índice GIN no Postgres |
| Recuperação de senha | Exige envio de e-mail |
| Notificações | "Fulano curtiu sua publicação" |
| Tópicos | Organizar publicações por assunto |

## 6.5 Progresso

| Etapa | Status |
|---|---|
| 0 — Documentação | ✅ |
| 1 — Esqueleto do backend | ⬜ |
| 2 — Autenticação | ⬜ |
| 3 — Publicações | ⬜ |
| 4 — Curtidas e comentários | ⬜ |
| 5 — Perfil e seguidores | ⬜ |
| 6 — Envio de imagens | ⬜ |
| 7 — Base do frontend | ⬜ |
| 8 — Início | ⬜ |
| 9 — Perfil no frontend | ⬜ |
| 10 — Acessibilidade | ⬜ |
