# Código compartilhado

`dotnet-api-sample/` é uma Web API mínima em .NET 8 (endpoints `GET /` e `GET /health`),
sem banco, sem dependências externas. Ela existe só para ter algo real pra containerizar
nos casos que pedem uma API .NET.

Quando um caso disser "copie o app compartilhado", copie o **conteúdo** da pasta
`dotnet-api-sample/` para dentro da pasta do caso (ex: `casos/01-multistage-dotnet/app/`).
Não edite o original aqui — cada caso deve ter sua própria cópia, porque em alguns casos
(07 e 08) você vai alterar o Dockerfile ou os arquivos de forma proposital.

Teste local rápido (sem Docker, só para confirmar que o app funciona):

```
dotnet run --project dotnet-api-sample
```

Depois acesse `http://localhost:5xxx/health` (a porta aparece no log do `dotnet run`).
