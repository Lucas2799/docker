# Caso 06 — Configuração por ambiente (dev/staging/prod)

## Objetivo

Rodar a mesma API .NET com configurações diferentes por ambiente, usando **um único
Dockerfile**, arquivos `.env` separados e `docker-compose.override.yml`, sem duplicar Compose
files inteiros nem hardcodar valores no Dockerfile.

## Por que isso importa

Um erro comum é ter `Dockerfile.dev`, `Dockerfile.staging`, `Dockerfile.prod` quase
idênticos, ou pior, variáveis hardcoded na imagem. O padrão maduro é: **uma imagem**, e a
diferença de comportamento entra via variáveis de ambiente e arquivos de override do
Compose, resolvidos no momento de subir o container — nunca no momento de buildar a imagem.

## Requisitos

1. Copie o app compartilhado para `casos/06-ambientes-env-override/app/` e adicione uma rota
   `GET /info` que retorna o valor de uma variável de ambiente `APP_ENVIRONMENT` (default
   `"unknown"` se não setada) e um `LOG_LEVEL`.
2. Crie:
   - `docker-compose.yml`: definição base do serviço `api` (build, porta, etc), **sem**
     valores de ambiente específicos hardcoded.
   - `.env.dev` e `.env.prod`: arquivos com `APP_ENVIRONMENT` e `LOG_LEVEL` diferentes.
   - `docker-compose.override.yml`: aplicado automaticamente em dev (é o comportamento
     padrão do Compose quando o arquivo tem esse nome exato), com algo que só faz sentido em
     dev (ex: bind mount do código para hot-reload, ou uma porta de debug exposta).
3. Suba em modo dev: `docker compose --env-file .env.dev up` e confirme `GET /info` refletindo
   os valores de dev.
4. Suba "como se fosse prod": `docker compose --env-file .env.prod -f docker-compose.yml up`
   (repare que aqui você **não** quer que o override de dev seja aplicado — pesquise como
   evitar isso passando `-f` explicitamente).

## Dicas (sem entregar a resposta)

- Compose aplica `docker-compose.override.yml` automaticamente **só** quando você não passa
  `-f` explicitamente. Passar `-f docker-compose.yml` sozinho ignora o override.
- Variáveis de `.env` viram interpolação dentro do `docker-compose.yml` (`${APP_ENVIRONMENT}`)
  e também podem virar variáveis de ambiente **dentro** do container via `environment:`. São
  dois usos relacionados mas distintos — não misture os dois sem entender a diferença.
- Nunca coloque segredo de verdade (senha de banco de prod, chave de API) num `.env` versionado
  no Git. Este caso é didático; no Caso 10 falamos de segurança de verdade.

## Critério de conclusão

- O mesmo `Dockerfile`/imagem é usado nos dois "ambientes" simulados.
- `GET /info` mostra valores diferentes dependendo de qual `.env` foi usado para subir.
- Você sabe explicar quando o `docker-compose.override.yml` é ou não aplicado.
