# Caso 03 — Orquestração multi-serviço com Compose

## Objetivo

Subir uma stack com **API .NET + PostgreSQL + Redis** via `docker-compose.yml`, com a API
só ficando "pronta" depois que o banco realmente aceita conexões (não apenas depois que o
container do banco iniciou).

## Por que isso importa

`depends_on` sem healthcheck só garante ordem de **start do container**, não que o serviço
lá dentro já está pronto para receber conexões. Postgres demora alguns segundos para aceitar
conexões depois que o container "startou". Uma API que tenta conectar cedo demais crasha ou
fica em retry-loop malfeito. Saber usar `healthcheck` + `depends_on: condition: service_healthy`
corretamente é básico de produção e um erro clássico de quem só usou Compose "no modo copiar
e colar".

## Requisitos

1. Copie o app compartilhado (`casos/_shared/dotnet-api-sample`) para
   `casos/03-orquestracao-multiservico/app/`.
2. Adicione ao `Program.cs` uma rota `GET /db-check` que tenta abrir uma conexão TCP simples
   (ou usa `Npgsql` se quiser ir além) com o serviço Postgres e retorna 200 se conectar, 503
   se não conseguir. Não precisa ser sofisticado — o objetivo é observar comportamento de
   rede entre containers, não fazer uma feature de produção.
3. Escreva um `docker-compose.yml` com três serviços:
   - `api`: builda a partir de um Dockerfile multi-stage (reaproveite o que você aprendeu no
     Caso 01).
   - `db`: `postgres:16`, com variáveis de ambiente de usuário/senha/banco e um named volume.
   - `cache`: `redis:7-alpine`.
4. Configure um `healthcheck` no serviço `db` (Postgres já expõe `pg_isready`) e faça o
   serviço `api` usar `depends_on` com `condition: service_healthy` apontando pro `db`.
5. Suba com `docker compose up` e confirme, olhando os logs, que a API só tenta conectar no
   banco depois que ele está healthy.

## Dicas (sem entregar a resposta)

- `pg_isready` é o comando padrão usado em healthchecks de Postgres — pesquise a sintaxe do
  `test:` dentro de `healthcheck:` no compose file.
- Dentro da rede do Compose, o hostname do Postgres é o **nome do serviço** (`db`), não
  `localhost` nem `127.0.0.1`.
- Redis não precisa de healthcheck pra este caso, mas pense em como você faria um
  (`redis-cli ping`) — é bom ter isso no repertório.
- Use `docker compose logs -f api` para observar em tempo real a ordem de inicialização.

## Critério de conclusão

- `docker compose up` sobe os três serviços.
- Nos logs, a `api` só aparece tentando falar com o banco **depois** que `db` reporta
  `healthy` (`docker compose ps` mostra isso).
- `GET /db-check` retorna 200 quando tudo está de pé.
- Se você derrubar o `db` (`docker compose stop db`) e chamar `/db-check` de novo, a resposta
  muda para 503 sem a API inteira cair.
