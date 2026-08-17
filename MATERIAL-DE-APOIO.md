# Material de Apoio — Teoria por caso

Leia a seção correspondente **antes** de tentar o caso. Isso não entrega a solução do
desafio — entrega o conceito que você precisa pra construir a solução sozinho.

## Glossário rápido (leia isso primeiro, vale pra tudo)

- **Imagem**: um pacote imutável (filesystem + metadados) a partir do qual containers são
  criados. Você não "edita" uma imagem em execução — você builda uma nova.
- **Container**: uma instância em execução de uma imagem. Tem seu próprio filesystem
  (efêmero, por padrão), processos e, opcionalmente, rede isolada.
- **Camada (layer)**: cada instrução do Dockerfile que modifica o filesystem (`RUN`, `COPY`,
  `ADD`) gera uma camada, cacheada e reaproveitada em builds futuros se nada antes dela mudou.
- **Registry**: onde imagens ficam armazenadas/versionadas (Docker Hub, GHCR, ECR, etc).
  `docker push`/`docker pull` falam com um registry.
- **Tag**: um rótulo de versão de uma imagem (`minha-api:1.2.0`, `minha-api:latest`). Uma
  mesma imagem pode ter várias tags.
- **Volume**: mecanismo de persistência de dados gerenciado pelo Docker, independente do
  ciclo de vida do container.
- **Bind mount**: uma pasta do seu host "espelhada" dentro do container. Diferente de
  volume: você controla o caminho no host diretamente.
- **Rede (network)**: mecanismo de isolamento/roteamento entre containers. Containers na
  mesma rede se enxergam pelo nome; em redes diferentes, não (por padrão).
- **`ENTRYPOINT` vs `CMD`**: `ENTRYPOINT` define o processo principal do container (o que
  roda sempre); `CMD` define argumentos padrão pra ele (sobrescrevíveis na hora do `docker run`).
  Um container "morre" quando esse processo principal termina.

Leitura oficial de referência geral: [Dockerfile reference](https://docs.docker.com/engine/reference/builder/) ·
[Docker Compose file reference](https://docs.docker.com/compose/compose-file/) ·
[Documentação geral do Docker](https://docs.docker.com/).

---

## Caso 01 — Multi-stage build

**Conceitos:**
- **Cache de build**: o Docker executa as instruções do Dockerfile em ordem, e reaproveita o
  cache de uma camada enquanto a instrução e tudo que ela depende (arquivos copiados antes
  dela) não mudarem. Por isso a ordem das instruções importa: coisas que mudam pouco (ex:
  `.csproj`, usado só pro `restore`) devem vir antes de coisas que mudam sempre (o resto do
  código-fonte).
- **Multi-stage**: um Dockerfile pode ter múltiplos blocos `FROM ... AS nome`. Cada bloco é
  um estágio isolado. Você pode copiar arquivos de um estágio para outro com
  `COPY --from=nome_do_estagio`. Só o **último** `FROM` do arquivo vira a imagem final — os
  estágios anteriores (com SDK, compiladores, ferramentas de build) não vão pra imagem final,
  a menos que você copie algo deles explicitamente.
- **SDK vs Runtime**: imagens `.../dotnet/sdk` incluem compilador e ferramentas (grandes);
  imagens `.../dotnet/aspnet` ou `.../dotnet/runtime` só executam binários já compilados
  (pequenas). Builda com SDK, roda com runtime.
- **`.dockerignore`**: funciona como `.gitignore`, mas pro contexto de build — evita copiar
  `bin/`, `obj/`, `.git/` etc pra dentro da imagem/contexto, o que também acelera o build.

**Leitura oficial:** [Multi-stage builds](https://docs.docker.com/build/building/multi-stage/) ·
[Cache de build](https://docs.docker.com/build/cache/) ·
[Boas práticas de Dockerfile](https://docs.docker.com/build/building/best-practices/)

---

## Caso 02 — Ambiente de dev containerizado (Rails)

**Conceitos:**
- **`docker compose run`**: diferente de `docker compose up`, roda um comando pontual num
  serviço (útil pra rodar `rails new`, migrations, um shell interativo) sem necessariamente
  deixar o serviço "no ar" continuamente.
- **Bind mount para código-fonte em dev**: montar sua pasta local dentro do container
  (`volumes: - .:/app` no Compose) faz o container enxergar mudanças de arquivo em tempo
  real, sem rebuild de imagem — é assim que se consegue hot-reload em ambiente containerizado.
- **DNS interno do Compose**: todo `docker-compose.yml` cria uma rede própria por padrão, e
  cada serviço vira resolvível pelo próprio nome (o serviço `db` responde por `db`, não por
  `localhost`).

**Leitura oficial:** [Compose overview](https://docs.docker.com/compose/) ·
[`docker compose run`](https://docs.docker.com/reference/cli/docker/compose/run/) ·
[Networking no Compose](https://docs.docker.com/compose/how-tos/networking/)

---

## Caso 03 — Orquestração multi-serviço (healthcheck)

**Conceitos:**
- **`depends_on` puro** só controla **ordem de start do container**, não prontidão do
  serviço. Um Postgres "startado" pode ainda não aceitar conexões.
- **`healthcheck`**: instrução (no Dockerfile ou no Compose) que define um comando periódico
  pra Docker considerar o container "healthy" ou "unhealthy" — ex: `pg_isready` pro Postgres.
- **`depends_on` com `condition: service_healthy`**: faz um serviço só iniciar de fato depois
  que o serviço dependente reportar `healthy` no healthcheck, não só "container criado".

**Leitura oficial:** [`HEALTHCHECK` no Dockerfile](https://docs.docker.com/reference/dockerfile/#healthcheck) ·
[`depends_on` no Compose](https://docs.docker.com/compose/how-tos/startup-order/)

---

## Caso 04 — Networking (redes bridge)

**Conceitos:**
- **Driver `bridge`**: o tipo de rede padrão pra containers num único host. A rede `bridge`
  **padrão** (criada automaticamente pelo Docker) não resolve nomes de container por DNS;
  redes bridge **customizadas** (`docker network create`), sim.
- **Isolamento por rede**: containers só se enxergam por padrão se estiverem na mesma rede.
  Um container pode participar de múltiplas redes ao mesmo tempo (`docker network connect`).
- **IP interno vs porta publicada**: todo container numa rede Docker tem um IP interno,
  usável por outros containers da mesma rede. Publicar porta pro host (`-p`) é um mecanismo
  separado, só necessário se algo **fora** do Docker (seu navegador, por exemplo) precisa
  acessar o container.

**Leitura oficial:** [Docker networking overview](https://docs.docker.com/engine/network/) ·
[Redes bridge](https://docs.docker.com/engine/network/drivers/bridge/)

---

## Caso 05 — Volumes e persistência

**Conceitos:**
- **Named volume** (`-v nome:/caminho`): armazenamento gerenciado pelo Docker, com ciclo de
  vida independente do container. Sobrevive a `docker rm`.
- **Bind mount** (`-v /caminho/host:/caminho`): aponta direto pra uma pasta real do seu
  disco. Você controla e vê os arquivos fora do Docker o tempo todo.
- **Filesystem do container**: tudo que não está em volume ou bind mount é efêmero — some
  quando o container é removido (não só parado; parar e reiniciar o **mesmo** container
  preserva o filesystem, remover o container não).
- **`docker exec`**: executa um comando dentro de um container **já rodando** — é como você
  roda `pg_dump` de dentro do container sem precisar instalar `postgresql-client` no host.

**Leitura oficial:** [Volumes](https://docs.docker.com/engine/storage/volumes/) ·
[Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)

---

## Caso 06 — Configuração por ambiente

**Conceitos:**
- **`.env` + interpolação**: o Compose lê automaticamente um arquivo `.env` na mesma pasta
  do `docker-compose.yml` e usa seus valores para preencher `${VARIAVEL}` dentro do arquivo
  YAML. `--env-file outro.env` deixa você apontar pra um arquivo diferente.
- **`environment:` no Compose**: define variáveis de ambiente **dentro do container** (o que
  seu `Program.cs`/app enxerga via `Environment.GetEnvironmentVariable`), separado da
  interpolação do YAML — os dois podem usar a mesma variável, mas são mecanismos distintos.
- **`docker-compose.override.yml`**: arquivo aplicado **automaticamente** por cima do
  `docker-compose.yml` quando você roda `docker compose up` sem especificar `-f`. Passar
  `-f docker-compose.yml` explicitamente ignora esse override.

**Leitura oficial:** [Variáveis de ambiente no Compose](https://docs.docker.com/compose/how-tos/environment-variables/) ·
[Múltiplos arquivos Compose](https://docs.docker.com/compose/how-tos/multiple-compose-files/)

---

## Caso 07 — Otimização de imagem

**Conceitos:** (some dos anteriores se aplicam de novo aqui: multi-stage, cache, `.dockerignore`)
- **`docker history <imagem>`**: mostra cada camada da imagem e o tamanho que ela adicionou —
  a ferramenta certa pra achar "onde" o peso está.
- Pacotes de debug/ferramentas (`vim`, `curl`, `htop`...) instalados na imagem final inflam
  tamanho e superfície de ataque sem necessidade — só instale o que o **runtime** da
  aplicação precisa.
- Variáveis como `ASPNETCORE_ENVIRONMENT=Development` mudam comportamento da aplicação
  (páginas de erro detalhadas, por exemplo) — não é só uma questão de tamanho de imagem, é
  comportamento inadequado pra produção.

**Leitura oficial:** [`docker history`](https://docs.docker.com/reference/cli/docker/image/history/) ·
[Boas práticas de Dockerfile](https://docs.docker.com/build/building/best-practices/)

---

## Caso 08 — Debug de container quebrado

**Conceitos:**
- **`docker logs <container>`** (ou `docker compose logs <serviço>`): primeira ferramenta
  sempre. Mostra stdout/stderr do processo principal do container, incluindo o motivo de um
  crash.
- **Exit code**: `docker ps -a` mostra o exit code do último processo do container. Exit
  code diferente de 0 indica falha; alguns códigos têm significado convencional (ex: 137 =
  matado por sinal, geralmente falta de memória).
- **`docker inspect <container>`**: mostra configuração completa do container em execução —
  incluindo portas realmente expostas/mapeadas, variáveis de ambiente ativas, usuário
  configurado. Essencial quando o container "está de pé" mas não se comporta como esperado.
- **`docker exec -it <container> sh`**: abre um shell dentro de um container **já rodando**.
  Se o container morre rápido demais pra isso funcionar, `docker run --entrypoint sh -it
  <imagem>` entra sem executar o `ENTRYPOINT` original, só pra investigação.
- **Ordem de `USER` e `COPY`/`RUN` no Dockerfile**: tudo que roda depois de `USER algueim`
  executa com as permissões desse usuário. Arquivos copiados antes disso, sem `--chown`,
  continuam pertencendo a quem os copiou (geralmente root), o que pode gerar "permission
  denied" pro usuário não-root tentar executá-los ou escrevê-los depois.

**Leitura oficial:** [`docker logs`](https://docs.docker.com/reference/cli/docker/container/logs/) ·
[`docker inspect`](https://docs.docker.com/reference/cli/docker/inspect/) ·
[`USER` no Dockerfile](https://docs.docker.com/reference/dockerfile/#user)

---

## Caso 09 — CI/CD com GitHub Actions

**Conceitos:**
- **Build de imagem em CI** é conceitualmente igual a buildar localmente, mas rodando numa
  máquina efêmera do GitHub, autenticada num registry pra poder publicar o resultado.
- **`GITHUB_TOKEN`**: token automático que toda execução de Action recebe, com permissões
  configuráveis via bloco `permissions:` no workflow. Pra publicar em GHCR, precisa de
  `packages: write`.
- **Estratégia de tags**: tag por SHA de commit dá rastreabilidade exata (qual código gerou
  qual imagem); `:latest` é conveniente mas não diz nada sobre versão — as duas normalmente
  coexistem, uma pra rastreabilidade, outra pra conveniência de "pegar a mais recente".

**Leitura oficial:** [Docker Build GitHub Actions](https://docs.docker.com/build/ci/github-actions/) ·
[Publicando imagens no GHCR](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

---

## Caso 10 — Segurança de containers

**Conceitos:**
- **Rodar como root dentro do container é o padrão**, a menos que você defina `USER`
  explicitamente. Isso significa que uma falha explorada na aplicação roda com privilégios de
  root **dentro** do container (que, dependendo de configuração do host/kernel, pode ser
  mais perigoso do que parece).
- **`COPY --chown=usuario:grupo`**: copia arquivos já atribuindo o dono certo, evitando o
  problema de permissão visto no Caso 08.
- **`--read-only`**: roda o container com filesystem raiz somente leitura, reduzindo
  drasticamente o que um processo comprometido conseguiria alterar. Diretórios que
  legitimamente precisam de escrita (ex: `/tmp`) podem ser liberados pontualmente via
  `tmpfs`, sem abrir mão do resto.
- **Scan de vulnerabilidades (CVE)**: ferramentas como Docker Scout ou Trivy comparam os
  pacotes instalados na imagem (incluindo a imagem base) contra bancos de dados de
  vulnerabilidades conhecidas, e reportam por severidade.

**Leitura oficial:** [Docker Scout](https://docs.docker.com/scout/) ·
[`USER` no Dockerfile](https://docs.docker.com/reference/dockerfile/#user) ·
[Isolamento e segurança de containers](https://docs.docker.com/engine/security/)

---

## Caso 11 (bônus) — Build reprodutível (Flutter)

**Conceitos:**
- **Imagem de build ≠ imagem de runtime**: o mesmo raciocínio do multi-stage (Caso 01) se
  aplica mesmo quando o "produto final" não é outro container, e sim um artefato estático
  (HTML/JS compilado, um APK). A imagem que builda pode ser grande e não precisa ser
  otimizada do mesmo jeito que uma imagem de produção.
- **Volume de cache de dependências**: ferramentas de build (pub, npm, maven, nuget...)
  costumam ter um cache de pacotes que vale a pena persistir num volume nomeado entre builds,
  em vez de baixar tudo de novo a cada execução do container.

**Leitura oficial:** [Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/) (mesmo mecanismo do Caso 05, aplicado a build)

---

## Caso 12 (capstone) — Composição completa

Não introduz conceito novo — é aplicação combinada de tudo dos casos 01 a 10. Releia as
seções acima conforme for montando cada parte da stack (proxy = networking do Caso 04,
bancos = volumes do Caso 05 + healthcheck do Caso 03, etc).

**Leitura complementar (proxy reverso, conceito novo aqui):**
[Documentação oficial do nginx sobre proxy reverso](https://nginx.org/en/docs/beginners_guide.html#proxy)
