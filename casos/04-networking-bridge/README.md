# Caso 04 — Networking: redes bridge custom e DNS interno

## Objetivo

Fazer dois containers se comunicarem pelo nome, usando **redes Docker criadas manualmente**
com `docker network create` — sem usar Compose. E depois provar, na prática, que containers
em redes diferentes **não** se enxergam por padrão.

## Por que isso importa

Compose cria e gerencia redes pra você por baixo dos panos, o que é ótimo no dia a dia mas
esconde o mecanismo. Pra debugar de verdade um problema de "container A não consegue falar
com container B" em produção (ou num Compose complexo com múltiplas redes), você precisa
entender bridge networks, DNS embutido do Docker e por que isolamento de rede existe.

## Requisitos

Não precisa de app .NET nem Rails aqui — use imagens prontas (`nginx:alpine` e
`busybox` ou `alpine`) para focar 100% em rede.

1. Crie uma rede custom: `docker network create estudo-net`.
2. Suba um container `nginx:alpine` chamado `web` **nessa rede**, sem publicar porta pro host
   (`docker run -d --name web --network estudo-net nginx:alpine`).
3. Suba um segundo container (`alpine` ou `busybox`) na **mesma rede**, e de dentro dele,
   acesse o nginx pelo nome (`wget -O- http://web` ou `curl http://web`, instalando curl se
   necessário). Deve funcionar.
4. Agora crie uma segunda rede (`docker network create outra-net`), suba um terceiro
   container **só nela**, e tente acessar `web` a partir dele. Deve **falhar**.
5. Conecte esse terceiro container também à `estudo-net` (`docker network connect`), sem
   recriar o container, e tente de novo. Deve funcionar agora.
6. Rode `docker network inspect estudo-net` e identifique: o range de IP da rede, os
   containers conectados e seus IPs internos.

## Dicas (sem entregar a resposta)

- A rede `bridge` padrão do Docker **não** tem DNS interno entre containers (resolução por
  nome só funciona nela via `--link`, que é legado). Redes customizadas, sim. Isso é uma
  pegadinha clássica de quem aprendeu Docker há tempos e nunca atualizou o mental model.
- `docker network ls` lista as redes existentes; repare que Compose sempre cria uma rede
  própria por projeto — é por isso que serviços de um mesmo `docker-compose.yml` se
  enxergam pelo nome do serviço.
- Container sem `--network` explícito cai na `bridge` padrão.

## Critério de conclusão

- Você consegue explicar, sem consultar nada, a diferença entre a rede `bridge` padrão e uma
  rede custom em relação a DNS interno.
- Reproduziu o cenário de falha (redes diferentes = sem comunicação) e o de sucesso (mesma
  rede, ou container conectado a ambas).
- Sabe ler a saída de `docker network inspect` e apontar o IP interno de um container
  específico.
