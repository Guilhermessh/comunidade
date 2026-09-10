# Comunidade Tech para Idosos

Uma rede social simples para pessoas idosas que estão aprendendo a mexer no
computador e na internet. A ideia é ter um lugar onde dá pra perguntar qualquer
coisa sem vergonha, contar o que conseguiu fazer e acompanhar outras pessoas que
estão passando pela mesma situação.

**Aviso:** o projeto ainda está em desenvolvimento. Por enquanto só a
documentação está pronta.

## Por que eu fiz esse projeto

Ele começou na faculdade, no curso de Análise e Desenvolvimento de Sistemas. Na
época era só uma parte de um site maior, que tinha quiz e joguinhos pra ensinar
informática pra idosos. A comunidade era o espaço onde as pessoas conversavam
sobre o que estavam aprendendo.

Só que eu acabei perdendo o site principal e sobrou só a comunidade. Em vez de
deixar parado, resolvi refazer ela do zero como um projeto meu, agora com backend
em Java.

A versão antiga era só HTML, CSS e JavaScript conversando direto com o Firebase.
Funcionava, mas o navegador tinha acesso direto ao banco e não existia nenhuma
regra no meio do caminho. Agora tem uma API em Java entre as duas coisas, que é
quem decide o que cada pessoa pode fazer.

```
Navegador                API em Java              Banco de dados
HTML + CSS + JS   ---->  Spring Boot     ---->    PostgreSQL
```

## Tecnologias

**Backend**

- Java 21
- Spring Boot
- Spring Data JPA
- Spring Security com JWT
- PostgreSQL (usando o Supabase)
- Maven

**Frontend**

- HTML, CSS e JavaScript puro
- Sem framework e sem npm, é só abrir no navegador

Eu troquei o Firebase pelo Postgres porque o projeto tem várias coisas ligadas
umas nas outras: a publicação tem um autor, o autor tem seguidores, a publicação
tem comentários e curtidas. Isso fica bem mais organizado num banco relacional.
E, sendo sincero, também porque eu queria aprender JPA de verdade e com o
Firebase eu não ia usar quase nada de Java.

## O que dá pra fazer no site

- [ ] Criar conta e entrar
- [ ] Publicar um texto com foto
- [ ] Editar e apagar as próprias publicações
- [ ] Curtir publicações
- [ ] Comentar
- [ ] Procurar publicações pelo texto
- [ ] Ter um perfil com foto e descrição
- [ ] Seguir outras pessoas
- [ ] Ver só as publicações de quem eu sigo
- [ ] Mudar o tamanho da letra do site

Vou marcando conforme fico pronto.

## Acessibilidade

Como o site é feito pra pessoas idosas, isso aqui não é um detalhe no final, é o
ponto principal do projeto:

- A letra começa em 18px e dá pra aumentar direto na tela, em três tamanhos
- Conferi o contraste das cores pelo WCAG. O azul que eu usava antes reprovava,
  então escureci ele
- Botões grandes, no mínimo 48x48 pixels, com espaço entre eles
- Dá pra navegar o site inteiro só pelo teclado
- Todo botão tem a palavra escrita do lado, não só o ícone
- Os textos são simples. O site fala "Início" e "Enviar foto" em vez de "Feed" e
  "Upload"

## Documentação

Antes de começar a programar eu parei pra escrever o que o projeto ia ser. Ajudou
bastante a não ficar mudando de ideia no meio:

- [Escopo e requisitos](docs/01-escopo.md) — o que o site faz e o que ele não faz
- [Arquitetura](docs/02-arquitetura.md) — como o código está organizado, com os
  diagramas
- [Modelo do banco](docs/03-modelo-de-dados.md) — as tabelas e como se conectam
- [API](docs/04-api.md) — a lista dos endpoints com exemplos
- [Design](docs/05-design.md) — cores, fontes, componentes e o desenho das telas
- [Roadmap](docs/06-roadmap.md) — as etapas e o que já terminei

## Como rodar

Ainda vou escrever essa parte direito quando o backend estiver funcionando. Por
enquanto o que vai ser necessário é:

- Java 21
- Maven
- Uma conta gratuita no [Supabase](https://supabase.com)

Backend:

```bash
cd backend
mvn spring-boot:run
```

Frontend: abrir a pasta `frontend` com o Live Server do VS Code.

## Organização das pastas

```
comunidade/
├── docs/       documentação
├── backend/    a API em Java
└── frontend/   as páginas do site
```

## Licença

[MIT](LICENSE)
