# Caso 05 — Volumes e persistência de dados

## Objetivo

Rodar um Postgres com dado persistido em **named volume**, provar que o dado sobrevive à
remoção do container, e fazer um backup/restore manual usando `pg_dump`/`pg_restore` a
partir de outro container.

## Por que isso importa

"Meu banco perdeu os dados quando eu recriei o container" é um erro clássico de quem não
entende a diferença entre o filesystem de um container (efêmero) e um volume (persistente).
Em pleno nível, você também precisa saber tirar um backup de um banco que só existe dentro
de containers, sem instalar `postgresql-client` na sua máquina.

## Requisitos

1. Suba um Postgres com named volume:
   `docker run -d --name pg-estudo -e POSTGRES_PASSWORD=estudo123 -v pg-dados:/var/lib/postgresql/data postgres:16`
2. Conecte nele (de outro container, ou via `docker exec -it pg-estudo psql -U postgres`) e
   crie uma tabela simples com alguns registros.
3. Remova o container (`docker rm -f pg-estudo`) e suba um **novo** container Postgres
   apontando pro **mesmo** volume nomeado. Confirme que a tabela e os dados continuam lá.
4. Agora faça o oposto: suba um Postgres **sem** volume nomeado (usando o filesystem interno
   do container), crie dados, remova o container, suba outro sem reaproveitar volume, e
   confirme que os dados **sumiram**. Isso é pra você sentir na pele a diferença.
5. Com o container do passo 3 (dados persistidos) rodando, use `docker exec` para rodar
   `pg_dump` de dentro do container e redirecionar a saída para um arquivo `.sql` na sua
   máquina (bind mount de uma pasta local para gravar o dump, ou `docker exec ... > backup.sql`
   no host).
6. Derrube tudo, suba um Postgres novo do zero, e restaure o backup nele.

## Dicas (sem entregar a resposta)

- Named volume (`-v nome:/caminho`) é gerenciado pelo Docker; bind mount (`-v
  /caminho/no/host:/caminho`) aponta pra uma pasta real do seu disco. São mecanismos
  parecidos na sintaxe, mas semânticas diferentes — não confunda os dois neste caso.
- `docker volume ls` e `docker volume inspect pg-dados` mostram onde o Docker guarda o dado
  de fato (você não deveria mexer nesse caminho manualmente, mas vale olhar uma vez).
- `pg_dump -U postgres nomedobanco` escreve o dump no stdout; redirecionar isso pra fora do
  container é o pulo do gato aqui.

## Critério de conclusão

- Você reproduziu os dois cenários: dado sobrevive (com volume) e dado some (sem volume),
  e sabe explicar por quê em uma frase.
- Existe um arquivo `backup.sql` no seu disco, gerado a partir de um container.
- Você restaurou esse backup num Postgres novo e os dados batem com o original.
