# Caso 10 — Segurança básica de containers

## Objetivo

Pegar a imagem otimizada do Caso 07 (`caso07-bom`) e endurecê-la (hardening): rodar como
usuário não-root **corretamente** (sem repetir o bug de permissão do Caso 08), rodar com
filesystem raiz somente leitura, e escanear a imagem em busca de vulnerabilidades conhecidas.

## Por que isso importa

Container rodando como root por padrão significa que, se alguém explorar uma falha na sua
aplicação, ganha root **dentro do container** — e dependendo da configuração do host, isso
pode ser escalado. Escanear imagens por CVEs conhecidas antes de ir pra produção é prática
padrão de mercado, não luxo de time de segurança.

## Requisitos

1. A partir do `Dockerfile` do Caso 07, crie uma versão endurecida que:
   - Crie um usuário e grupo dedicados (não use `nobody` genérico) e rode a aplicação com
     `USER` apontando pra esse usuário — **na ordem certa**, para não repetir o bug de
     permissão do Caso 08 (dica: `COPY --chown=usuario:grupo` existe por um motivo).
   - Use `--read-only` ao rodar o container (`docker run --read-only ...`) e identifique se a
     aplicação precisa de algum diretório gravável (ex: `/tmp`) — se precisar, declare um
     `tmpfs` só pra esse caminho, em vez de liberar o filesystem inteiro.
2. Rode `docker scout cves caso10-hardened` (Docker Scout já vem com Docker Desktop) ou, se
   preferir, `trivy image caso10-hardened` (instalação separada). Liste as vulnerabilidades
   de severidade alta/crítica encontradas, se houver.
3. Se o scan encontrar CVEs na imagem base, teste trocar `mcr.microsoft.com/dotnet/aspnet:8.0`
   por uma tag mais específica/recente e rode o scan de novo — compare o resultado.
4. Documente em `SEGURANCA.md`: o que você mudou, por quê, e o resultado do scan antes/depois.

## Dicas (sem entregar a resposta)

- `USER` some depois de definido — toda instrução seguinte no Dockerfile roda como esse
  usuário. Se você faz `COPY` depois de `USER app` mas os arquivos de origem pertencem a
  root no build context, ainda pode dar problema de dono/permissão dependendo de como você
  usa `--chown`.
- Containers .NET escrevem em diretórios como `/tmp` para certas operações internas (ex:
  algumas configs de cache do runtime) — rodar `--read-only` sem mapear isso pode gerar
  falhas sutis só perceptíveis em uso mais pesado, não num simples `/health`.
- Nem toda CVE reportada é "crítica de verdade" pro seu contexto — parte do trabalho de
  pleno é saber ler um relatório de scan e priorizar, não entrar em pânico com a lista
  inteira.

## Critério de conclusão

- O container roda com um usuário não-root, comprovável via `docker exec ... whoami` (ou
  `id`) mostrando um UID diferente de 0.
- `docker run --read-only` funciona sem erro de escrita, com qualquer `tmpfs` necessário
  devidamente mapeado.
- `SEGURANCA.md` documenta o resultado do scan e qualquer ação tomada em cima dele.
