# Caso 08 — Debugando um container quebrado

## Objetivo

O `Dockerfile` e o `docker-compose.yml` desta pasta **buildam**, mas o container não fica
saudável. Existem **3 problemas distintos** propositais. Encontre e corrija todos, um de
cada vez, usando as ferramentas de diagnóstico do Docker — não adivinhando.

## Por que isso importa

"Builda mas não funciona" é o cenário mais comum de troubleshooting real. Ninguém te entrega
um erro de compilação óbvio — te entrega um container que sobe e morre, ou sobe e não
responde, e a causa raiz está nos logs, no exit code, ou num `docker exec` investigativo.
Isso é o que separa "sei escrever Dockerfile" de "sei operar Docker em produção".

## Regras

- **Não reescreva o Dockerfile do zero.** Corrija linha a linha, entendendo cada bug antes
  de mudar. O objetivo é treinar diagnóstico, não "resetar e copiar um bom exemplo".
- Para cada bug encontrado, anote num arquivo `DIAGNOSTICO.md`: o sintoma observado, o
  comando que usou para investigar, a causa raiz, e a correção aplicada.

## Passo a passo sugerido

1. `docker compose build && docker compose up` — observe o que acontece (sobe? cai? fica em
   loop de restart?).
2. Use `docker compose logs api` (ou `docker logs <container>`) para ler a mensagem de erro
   real. Não pule esse passo — a mensagem geralmente já entrega a causa.
3. Corrija o primeiro problema, rebuilde, suba de novo. Repita até o container ficar de pé.
4. Quando o container estiver rodando (não crashando mais), tente `curl http://localhost:5000/health`
   (ou a porta que você configurou). Se não responder, use `docker inspect <container>` para
   conferir quais portas o container realmente expõe/escuta, e `docker exec -it <container> sh`
   (se conseguir entrar) para investigar por dentro.
5. Só declare o caso resolvido quando `curl` no host retornar `{"status":"healthy"}`.

## Dicas (sem entregar a resposta)

- Exit code e a última linha de log de um container que morre na hora geralmente apontam
  exatamente pro arquivo/comando que falhou — leia com atenção, não só olhe por cima.
- Se o container "fica de pé" mas você não consegue acessar via `curl` do host, o problema
  não é necessariamente rede — pode ser a porta interna que o app realmente escuta sendo
  diferente da porta que você mapeou/exposed.
- Se você não consegue nem `docker exec -it ... sh` porque o container não fica vivo tempo
  suficiente, rode o container em foreground (sem `-d`) para ver o erro em tempo real, ou
  use `docker run --entrypoint sh -it <imagem>` para entrar sem executar o `ENTRYPOINT`
  original.
- Um dos bugs é sobre **permissão de arquivo dentro do container** — pense na ordem entre
  trocar de usuário (`USER`) e copiar arquivos.

## Critério de conclusão

- `docker compose up` sobe o serviço `api` sem crashar e sem loop de restart.
- `curl` (do seu host) contra a porta configurada retorna 200 em `/health`.
- `DIAGNOSTICO.md` documenta os 3 bugs, com sintoma, investigação e correção de cada um.
