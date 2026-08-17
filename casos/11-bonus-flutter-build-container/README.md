# Caso 11 (bônus) — Container de build reprodutível para Flutter

## Objetivo

Buildar um app Flutter (web ou Android) inteiramente dentro de um container, sem depender do
Flutter SDK instalado localmente, produzindo os artefatos finais (build web estática, ou
APK) na sua máquina via bind mount.

## Por que isso importa

Isso não é sobre "rodar Flutter em produção dentro de Docker" (apps mobile não rodam em
container). É sobre um problema real e comum: **builds reprodutíveis**. "Funciona na minha
máquina" também vale pra ferramentas de build — versão de Flutter/Dart diferente entre
devs, ou entre sua máquina e o CI, gera builds diferentes. Containerizar o **ambiente de
build** (não o app em runtime) resolve isso, e é uma técnica que você pode usar tanto pra
Flutter quanto pra qualquer toolchain de build pesado.

## Requisitos

1. Escolha um app Flutter simples que você já tenha (ou crie um novo `flutter create` — pode
   ser feito também dentro do container, como fizemos com Rails no Caso 02, se quiser
   praticar de novo).
2. Escreva um `Dockerfile` baseado numa imagem com Flutter SDK instalado (pesquise imagens
   oficiais/community mantidas, ex: a partir de uma imagem `ubuntu`/`debian` instalando o SDK
   manualmente, ou uma imagem community já pronta — avalie a procedência antes de usar uma
   imagem de terceiros não-oficial).
3. O container deve rodar `flutter pub get` e `flutter build web` (mais simples de validar
   sem precisar de Android SDK/emulador) usando um bind mount do seu código-fonte.
4. O resultado do build (`build/web/`) deve aparecer no seu disco, fora do container, pronto
   pra servir com qualquer servidor estático.
5. Sirva o resultado localmente (pode ser com um container `nginx:alpine` simples, servindo
   a pasta `build/web` via bind mount) e confirme que abre no navegador.

## Dicas (sem entregar a resposta)

- Imagens com Flutter SDK completo tendem a ser grandes (SDK + toolchain Dart). Isso é
  aceitável aqui porque essa imagem **não vai pra produção** — ela só builda. É uma
  diferenciação importante: imagem de build pode ser gorda, imagem de runtime não.
  Isso conecta direto com o que você aprendeu multi-stage no Caso 01 e otimização no 07 —
  aqui você está aplicando o mesmo raciocínio, só que a "imagem final" nem existe como
  container de runtime, é só o artefato estático.
- Cache de dependências Dart/Flutter (pub cache) pode ser acelerado com um volume nomeado
  dedicado, pra não rebaixar tudo do zero a cada build.

## Critério de conclusão

- O build do app Flutter roda inteiramente dentro do container.
- Os arquivos gerados aparecem no seu disco (fora do container).
- Você consegue servir e abrir o resultado no navegador.
- Você consegue explicar por que essa imagem de build não deveria ser a mesma usada em
  produção/runtime.
