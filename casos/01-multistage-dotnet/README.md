# Caso 01 — Multi-stage build com .NET

## Objetivo

Escrever um `Dockerfile` multi-stage para a API .NET compartilhada (`casos/_shared/dotnet-api-sample`),
de forma que a imagem final **não contenha o SDK do .NET**, só o runtime necessário para rodar.

## Por que isso importa

Um Dockerfile "básico" costuma usar a imagem do SDK (`mcr.microsoft.com/dotnet/sdk`) tanto
para compilar quanto para rodar. Isso funciona, mas gera uma imagem final de ~800MB+ quando
o runtime sozinho resolveria com ~100-200MB. Em produção isso significa deploys mais lentos,
mais superfície de ataque e mais custo de storage/transferência em registries. Separar
"imagem que compila" de "imagem que roda" (multi-stage) é uma das primeiras coisas que
diferenciam um Dockerfile júnior de um pleno.

## Requisitos

1. Copie `casos/_shared/dotnet-api-sample/*` para `casos/01-multistage-dotnet/app/`.
2. Crie um `Dockerfile` em `casos/01-multistage-dotnet/` com pelo menos **dois estágios**:
   - Um estágio `build` baseado em `mcr.microsoft.com/dotnet/sdk:8.0`, que restaura
     dependências e publica o app (`dotnet publish`).
   - Um estágio final baseado em `mcr.microsoft.com/dotnet/aspnet:8.0` (runtime), que copia
     **apenas os artefatos publicados** do estágio de build.
3. A imagem final deve rodar com `ENTRYPOINT ["dotnet", "docker-study-api.dll"]` (ajuste o
   nome do dll ao seu `.csproj`).
4. O container deve expor a porta 8080 e responder em `GET /health` com HTTP 200.
5. Faça o build e rode o container mapeando a porta: `docker run -p 8080:8080 ...`

## Dicas (sem entregar a resposta)

- No estágio de build, copie primeiro só o `.csproj` e rode `dotnet restore` **antes** de
  copiar o resto do código. Pesquise por que essa ordem importa para o cache de camadas do
  Docker (o que muda mais: seu código ou seu `.csproj`?).
- A partir do .NET 8, containers ASP.NET Core escutam por padrão na porta 8080. Confirme isso
  lendo a doc oficial em vez de assumir — versões diferentes mudam esse padrão.
- Use `.dockerignore` para não copiar `bin/`, `obj/` e afins para dentro do contexto de build.
- `docker images` vai te mostrar o tamanho de cada imagem. Compare o que aconteceria se você
  usasse só a imagem do SDK como final (não precisa fazer, só raciocinar sobre o tamanho).

## Critério de conclusão

- `docker build -t caso01-api .` completa sem erro.
- `docker images caso01-api` mostra uma imagem com no máximo ~220MB (referência aproximada
  para uma API mínima em runtime ASP.NET 8; se estiver muito acima disso, algo do estágio de
  build vazou pra imagem final).
- `curl http://localhost:8080/health` retorna `{"status":"healthy"}`.
- Você consegue explicar, em uma frase, por que a imagem final não tem o SDK instalado.
