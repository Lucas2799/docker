# Caso 09 — CI/CD: build e push automatizado de imagem

## Objetivo

Configurar um workflow de GitHub Actions que, a cada push na branch principal, builda a
imagem Docker da API .NET (reaproveite o Caso 01) e faz push para o **GitHub Container
Registry (GHCR)**, com uma estratégia de tags sensata.

## Por que isso importa

Buildar e rodar localmente é uma parte da história. Em qualquer time, a imagem que vai pra
produção é buildada por uma pipeline, não pelo `docker build` de alguém na máquina. Saber
configurar isso — incluindo autenticação no registry e estratégia de tags — é esperado de
um pleno.

## Pré-requisitos

- Um repositório no GitHub (pode ser um repo novo, só pra este estudo, com o conteúdo do
  Caso 01 dentro).
- Não precisa de nenhum secret manual: o GHCR aceita o token automático
  `GITHUB_TOKEN` que toda Action já recebe, desde que o workflow tenha a permissão certa.

## Requisitos

1. Suba para um repositório GitHub o app + Dockerfile do Caso 01 (ou uma cópia).
2. Crie `.github/workflows/docker-build.yml` que:
   - Dispara em push para `main` (e, se quiser, em pull requests, mas sem fazer push da
     imagem nesse caso — só build, pra validar).
   - Usa `docker/build-push-action` (ou `docker build`/`docker push` manual, se preferir
     entender o passo a passo cru antes de usar a action pronta).
   - Faz login no GHCR usando `docker/login-action` com `${{ secrets.GITHUB_TOKEN }}`.
   - Builda e publica a imagem com **duas tags**: uma fixa por commit
     (`ghcr.io/<usuario>/<repo>:${{ github.sha }}`) e uma móvel (`:latest`, só quando for
     push em `main`).
3. Confirme, na aba "Packages" do seu perfil/repositório GitHub, que a imagem foi publicada.
4. Puxe a imagem publicada pra sua máquina (`docker pull ghcr.io/...`) e rode-a, provando que
   o que subiu no CI realmente funciona.

## Dicas (sem entregar a resposta)

- O workflow precisa da permissão `packages: write` declarada explicitamente (o padrão de
  permissões do `GITHUB_TOKEN` mudou ao longo do tempo; não assuma, confira a doc).
- Tag por SHA de commit (`github.sha`) te dá rastreabilidade exata de qual código gerou qual
  imagem — isso importa pra debugar produção depois. `:latest` sozinho não te diz nada sobre
  qual versão está rodando.
- Pense em usar cache de build da própria action (`cache-from`/`cache-to`) para acelerar
  builds subsequentes — não é obrigatório, mas é uma boa prática de pleno pra cima.

## Critério de conclusão

- Existe um workflow do GitHub Actions rodando com sucesso, visível na aba "Actions" do repo.
- A imagem aparece publicada em GHCR com pelo menos duas tags (SHA e `latest`).
- Você consegue rodar localmente a imagem que veio do CI (não uma buildada localmente) e ela
  funciona.
