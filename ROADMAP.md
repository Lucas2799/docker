# Roteiro de Estudos — De Júnior/Básico a Pleno em Docker

Contexto: você é dev pleno em .NET, também trabalha com Ruby on Rails e estuda Flutter/Dart.
Os casos práticos usam essa stack de propósito — o objetivo não é aprender Docker "no vácuo",
mas aplicá-lo exatamente nos cenários que você encontra (ou vai encontrar) no trabalho.

## Como este material está organizado

- `ROADMAP.md` (este arquivo): visão geral, ordem sugerida, o que estudar antes de cada caso.
- `MATERIAL-DE-APOIO.md`: a teoria de cada caso, na mesma ordem do roteiro — **leia a seção
  correspondente antes de tentar o caso**. É onde ficam os conceitos (camadas, cache,
  multi-stage, networking, volumes, healthcheck, etc) e os links pra documentação oficial.
- `casos/NN-nome/README.md`: um desafio prático por pasta. Cada README traz **objetivo,
  contexto, requisitos, dicas e critério de conclusão** — não a solução pronta. A ideia é
  você errar, debugar e chegar lá, que é como se fixa conhecimento de pleno.
- `casos/_shared/`: código-fonte mínimo reaproveitado entre vários casos (uma API .NET
  minimalista), para você não perder tempo escrevendo app só pra ter algo pra containerizar.

## O gap júnior → pleno em Docker

Alguém "básico" sabe `docker run`, `docker build`, talvez um `docker-compose up`. Pleno em
Docker significa:

1. Entender **camadas e cache de build** o suficiente pra escrever Dockerfiles rápidos de buildar.
2. Saber projetar imagens **multi-stage**, pequenas e sem lixo de build na imagem final.
3. Orquestrar **múltiplos serviços** com Compose (redes, dependências, healthchecks).
4. Entender **networking** de containers (bridge, DNS interno, isolamento).
5. Gerenciar **estado e persistência** (volumes vs bind mounts, backup).
6. Separar **configuração por ambiente** (dev/staging/prod) sem duplicar Dockerfiles.
7. **Debugar** um container que não sobe, sem chutar — usando logs, exec, inspect.
8. Integrar build/push de imagem em **CI/CD**.
9. Aplicar **segurança básica** (non-root, superfície de ataque, scan de vulnerabilidades).
10. Ter senso crítico sobre **tamanho e composição de imagem** (Alpine, distroless, etc).

Esse roadmap cobre os 10 pontos, em ordem crescente de complexidade.

## Pré-requisitos

- Docker Desktop instalado e rodando (`docker version` funcionando no terminal).
- .NET SDK instalado (você já tem, mas note: em alguns casos o objetivo é justamente
  **não depender** do SDK local — o container é que compila).
- Git e uma conta GitHub (necessário só no caso 09).
- Opcional: [Docker Scout](https://docs.docker.com/scout/) (já vem com Docker Desktop) para o caso 10.

## Ordem sugerida (ritmo de ~1 caso a cada 2-4 dias, estudando em casa)

| # | Caso | Foco principal | Tempo estimado |
|---|------|-----------------|-----------------|
| 01 | [Multi-stage .NET](casos/01-multistage-dotnet/README.md) | Dockerfile multi-stage, camadas, cache | 1-2h |
| 02 | [Rails dev containerizado](casos/02-rails-dev-containerizado/README.md) | Ambiente de dev sem instalar a linguagem localmente | 2-3h |
| 03 | [Orquestração multi-serviço](casos/03-orquestracao-multiservico/README.md) | Compose, depends_on, healthcheck | 2h |
| 04 | [Networking com bridge](casos/04-networking-bridge/README.md) | Redes custom, DNS interno, isolamento | 1-2h |
| 05 | [Volumes e persistência](casos/05-volumes-persistencia/README.md) | Named volumes, bind mounts, backup/restore | 1h |
| 06 | [Ambientes com .env e override](casos/06-ambientes-env-override/README.md) | Config por ambiente, `docker-compose.override.yml` | 1-2h |
| 07 | [Otimização de imagem](casos/07-otimizacao-imagem/README.md) | Reduzir uma imagem "gorda" de forma mensurável | 2h |
| 08 | [Debugando um container quebrado](casos/08-debug-container-quebrado/README.md) | logs/exec/inspect, causa raiz | 1-2h |
| 09 | [CI/CD com GitHub Actions](casos/09-cicd-github-actions/README.md) | Build + push automatizado, tagging | 2-3h |
| 10 | [Segurança de containers](casos/10-seguranca-container/README.md) | Non-root, read-only fs, scan de CVEs | 2h |
| 11 (bônus) | [Container de build para Flutter](casos/11-bonus-flutter-build-container/README.md) | Reprodutibilidade de ambiente de build | 1-2h |
| 12 (capstone) | [Mini plataforma de microsserviços](casos/12-capstone-mini-microservicos/README.md) | Junta tudo: .NET + Rails + proxy + DB + cache | 4-6h |

## Como estudar cada caso

1. Leia a seção correspondente em `MATERIAL-DE-APOIO.md` — é a teoria mínima necessária.
2. Leia o README do caso inteiro antes de digitar qualquer comando.
3. Tente resolver sozinho. Trave em algo? Releia a seção "Dicas" antes de me chamar.
4. Valide pelo critério de conclusão do próprio README (evita "achismo" de que funcionou).
5. Quando terminar, me chame e me conte o que você fez — eu reviso sua solução, aponto o
   que um pleno faria diferente, e sigo para o próximo caso.

## Regra de ouro

Nenhum `Dockerfile` ou `docker-compose.yml` de solução vem pronto nos casos (exceto quando o
próprio desafio é "conserte isto"). Você escreve, quebra, conserta. É assim que se aprende
Docker de verdade — decorar um Dockerfile de exemplo não gera fluência.
