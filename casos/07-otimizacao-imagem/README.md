# Caso 07 — Otimização de imagem, de forma mensurável

## Objetivo

Partir de um `Dockerfile.ruim` (já pronto, funcional, mas cheio de práticas ruins) e escrever
do zero um `Dockerfile` novo que resolva os mesmos requisitos com uma imagem **muito menor**
e **mais segura**. No final, você compara os dois com números reais, não "achismo".

## Por que isso importa

Em algum momento você vai herdar um Dockerfile que "funciona" mas ninguém nunca otimizou.
Saber diagnosticar *por que* uma imagem está grande — e corrigir isso com dados, não
intuição — é uma habilidade prática de pleno que junta tudo que você viu nos casos 01 e
(se já fez) 10.

## Requisitos

0. `Program.cs` e `docker-study-api.csproj` já estão prontos nesta pasta (raiz do caso, no
   mesmo nível do `Dockerfile.ruim`) — não precisa copiar nada de `_shared`.
1. **Não edite** `Dockerfile.ruim`. Faça o build dele como está:
   `docker build -f Dockerfile.ruim -t caso07-ruim .`
   Anote o tamanho: `docker images caso07-ruim`.
2. Liste, por escrito (num arquivo `ANALISE.md` que você cria), pelo menos **5 problemas**
   concretos que você identifica no `Dockerfile.ruim`. Não vale só "está grande" — aponte a
   linha e o motivo.
3. Escreva um `Dockerfile` novo (bom) que resolva cada problema listado. Deve continuar
   expondo `GET /health` na porta 8080.
4. Builde: `docker build -t caso07-bom .` e compare o tamanho com o `caso07-ruim`.
5. Rode `docker history caso07-bom` e `docker history caso07-ruim` e compare quantas camadas
   cada um tem e o peso de cada camada.

## Dicas (sem entregar a resposta)

- Pacotes de debug (`vim`, `htop`, `net-tools`...) não têm lugar numa imagem de produção.
  Pense em quando/se você realmente precisa deles (spoiler: praticamente nunca na imagem
  final; no máximo temporariamente, num container à parte, para debugar).
- Ordem de instruções no Dockerfile afeta cache. `COPY . .` antes de `dotnet restore` invalida
  o cache de restore toda vez que **qualquer** arquivo muda, mesmo um `README.md`.
  `ASPNETCORE_ENVIRONMENT=Development` numa imagem que seria de produção é um problema de
  comportamento, não só de tamanho — pesquise o que essa variável muda (paginas de erro
  detalhadas, etc.) e por que isso é arriscado expor.
- Rodar como root dentro do container é uma prática ruim mesmo que este caso seja focado em
  tamanho — se quiser ir além, já aplique o que fizer sentido do Caso 10.

## Critério de conclusão

- `ANALISE.md` lista pelo menos 5 problemas reais, cada um com a correção aplicada.
- `caso07-bom` é significativamente menor que `caso07-ruim` (espere uma redução de várias
  vezes o tamanho — nesse tipo de comparação é comum sair de 800MB+ para menos de 220MB).
- Ambas as imagens respondem corretamente em `GET /health`.
