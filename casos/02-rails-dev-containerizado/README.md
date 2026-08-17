# Caso 02 — Ambiente de desenvolvimento Rails 100% containerizado

## Objetivo

Criar um app Rails novo **sem instalar Ruby, Rails ou Bundler na sua máquina** — tudo roda
dentro de containers, incluindo o `rails new`.

## Por que isso importa

Você já mexe em um projeto Rails. Um cenário muito comum no dia a dia pleno é: "preciso
mexer nesse projeto Ruby/Rails mas não quero (ou não posso) instalar essa versão específica
de Ruby na minha máquina, que já tem .NET, Flutter e mais um monte de coisa". Saber montar
um ambiente de desenvolvimento inteiramente em container — incluindo geração de projeto,
instalação de gems e banco — é uma habilidade que separa quem "sabe usar um Dockerfile que
alguém deixou pronto" de quem monta o ambiente do zero.

## Requisitos

1. Dentro de `casos/02-rails-dev-containerizado/`, crie um `Dockerfile` baseado em uma imagem
   oficial do Ruby (ex: `ruby:3.3-slim` ou similar) que instale as dependências de sistema
   necessárias para rodar Rails com Postgres (ex: `build-essential`, `libpq-dev`, `nodejs` se
   for usar asset pipeline).
2. Crie um `docker-compose.yml` com dois serviços:
   - `web`: builda a partir do seu Dockerfile.
   - `db`: `postgres:16` (ou versão de sua escolha), com um volume nomeado para persistir dados.
3. Use o container `web` para **gerar o projeto Rails** (`rails new . --database=postgresql`)
   dentro de um bind mount para a pasta do caso — assim o código gerado fica no seu disco,
   não só dentro do container.
4. Configure o `database.yml` do Rails gerado para apontar para o serviço `db` do Compose
   (não para `localhost`).
5. Suba tudo com `docker compose up` e confirme que `rails server` responde em
   `http://localhost:3000`.

## Dicas (sem entregar a resposta)

- Pra gerar o projeto sem já ter um `Gemfile`, você vai rodar um comando `docker compose run`
  (ou `docker run` avulso) **antes** de ter um Dockerfile "definitivo" — pense em como
  resolver esse problema do ovo e da galinha: uma imagem base do Ruby puro gera o projeto
  primeiro, e só depois você ajusta o Dockerfile para instalar as gems do `Gemfile` que foi
  gerado.
- O hostname do banco dentro da rede do Compose **não é `localhost`** — é o nome do serviço
  (`db`). Isso é networking interno do Docker Compose; volte a esse conceito no Caso 04 se
  ficar confuso.
- Bind mount (pasta local montada dentro do container) é diferente de named volume (dado
  gerenciado pelo Docker). Você vai precisar dos dois aqui: bind mount pro código, named
  volume pro dado do Postgres.

## Critério de conclusão

- Existe um `Gemfile` e uma estrutura de app Rails real na pasta do caso (gerada por você,
  via container).
- `docker compose up` sobe `web` e `db` sem erro de conexão com banco.
- `http://localhost:3000` mostra a página padrão do Rails (ou uma rota sua).
- Você editar um arquivo Ruby localmente (no seu editor) e ver o efeito refletido no
  container sem rebuild (graças ao bind mount).
