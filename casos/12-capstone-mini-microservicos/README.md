# Caso 12 (capstone) — Mini plataforma de microsserviços

## Objetivo

Juntar tudo que você praticou nos casos anteriores numa única stack orquestrada por Compose,
simulando um cenário real de dois serviços de linguagens diferentes atrás de um proxy reverso,
com banco, cache, healthchecks, configuração por ambiente e um mínimo de hardening.

## Por que isso importa

Este é o exercício que prova que você não só entende cada peça isolada, mas sabe **compor**
elas — que é, na prática, o trabalho de pleno em Docker no dia a dia: manter um
`docker-compose.yml` real de um sistema com múltiplos serviços interdependentes.

## Arquitetura esperada

```
                    ┌──────────────┐
     :80  ───────▶  │  nginx (proxy) │
                    └──────┬───────┘
                 ┌─────────┴─────────┐
                 ▼                   ▼
         ┌───────────────┐   ┌───────────────┐
         │  api-dotnet    │   │  api-rails    │
         │  (Caso 01/03)  │   │  (Caso 02)    │
         └───────┬────────┘   └───────┬───────┘
                 ▼                   ▼
         ┌───────────────┐   ┌───────────────┐
         │   postgres     │   │     redis     │
         └───────────────┘   └───────────────┘
```

## Requisitos

1. Reaproveite (copiando, não referenciando) os apps dos Casos 01/03 (.NET) e 02 (Rails).
2. `nginx` como proxy reverso: `/api/dotnet/*` roteia para `api-dotnet`, `/api/rails/*`
   roteia para `api-rails`. Escreva o `nginx.conf` você mesmo.
3. `api-dotnet` conecta no `postgres`; `api-rails` conecta no `redis` (ou inverta, o
   importante é ter os dois serviços de app falando com pelo menos um serviço de estado).
4. Healthchecks em `postgres` e `redis`; `depends_on` com `condition: service_healthy` nos
   serviços de aplicação correspondentes.
5. Todos os serviços de aplicação rodam como usuário não-root (aplicando o Caso 10).
6. Configuração de ambiente via `.env` (aplicando o Caso 06) — pelo menos a diferenciação
   entre um modo "dev" (com bind mounts pra hot-reload) e um modo mais próximo de "prod".
7. Uma rede dedicada para o tráfego "de borda" (nginx ↔ APIs) e, se fizer sentido, redes
   separadas para os serviços de dados, isolando o que não precisa ser exposto amplamente
   (aplicando o Caso 04).
8. Named volumes para os dados de `postgres` e `redis` (aplicando o Caso 05).

## Critério de conclusão

- `docker compose up` sobe a stack inteira sem erro.
- `curl http://localhost/api/dotnet/health` e `curl http://localhost/api/rails/...` (uma
  rota equivalente no Rails) respondem corretamente através do nginx.
- Derrubar e recriar os containers de app não perde dado do Postgres/Redis.
- Você sabe justificar cada decisão de rede/volume/healthcheck que tomou — se eu perguntar
  "por que você separou essa rede" ou "por que esse healthcheck existe", você responde sem
  titubear.

## Depois de terminar

Este é o último caso do roteiro base. Se quiser continuar evoluindo depois deste, os
próximos passos naturais (fora do escopo deste material, mas bons temas pra pesquisar por
conta própria) são: Docker Swarm ou Kubernetes para orquestração multi-host, service mesh,
e observabilidade (Prometheus/Grafana) para os containers que você já sabe operar bem.
