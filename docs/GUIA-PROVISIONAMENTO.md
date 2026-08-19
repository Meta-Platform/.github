# Guia: como o código chega ao ar

Editar um arquivo não coloca a mudança em execução. Entre o que você edita e o
que executa existem **várias cópias do mesmo código**, e quase todo erro de
operação vem de confundir uma com outra: você altera a primeira, reinicia a
última, e nada muda — **sem nenhum erro**.

Este guia diz quais são as cópias, o que atualiza cada uma, e como **provar** que
a mudança chegou.

## As cópias

Duas existem sempre:

| # | Cópia | Onde | Quem atualiza |
|---|---|---|---|
| 1 | **Clone de trabalho** | o repositório onde você edita | você |
| 2 | **Cópia instalada** | `<EcosystemData>/repos/<Repositório>` | `repo update <Repositório>` |

**O que executa é a cópia 2, nunca o seu diretório de trabalho.**

Quando o ecossistema empacota o repositório para distribuir ou executar isolado,
surgem outras — um artefato versionado, uma imagem construída a partir dele, e a
instância em execução. Cada camada nasce da anterior, e **pular uma não produz
erro**: produz uma execução com o código antigo.

## Duas consequências que custam tempo

**Processo já em execução não vê código novo.** O Node carrega os módulos uma
vez, no início. `repo update` coloca o arquivo novo em disco; o processo em
execução continua com o que carregou. Atualizar exige `repo update` **e**
reiniciar o processo. Antes de reiniciar, confira: se o processo é **mais velho**
que o arquivo, ele não tem o código novo.

**Quem executa a partir de um artefato empacotado ignora a cópia instalada.**
Atualizar a cópia 2 e reiniciar a instância não muda nada: o artefato precisa ser
gerado de novo. Essa divergência é silenciosa — nenhuma ferramenta a acusa.

## Matriz de propagação

A coluna **prova** é o que constitui verificação. Nada além dela conta como
sucesso.

| O que mudou | Mínimo suficiente | Prova de que chegou |
|---|---|---|
| Código de pacote executado a partir da cópia instalada | `repo update` → reiniciar o processo | processo com identificador novo, respondendo pelo socket |
| Código de pacote executado a partir de artefato empacotado | `repo update` → reempacotar → reprovisionar | construção concluída **e** instância nova **e** a resposta esperada |
| Apenas parâmetros de execução | reprovisionar | os parâmetros da instância nova refletem o arquivo |
| Biblioteca de **outro** repositório | atualizar e reempacotar **aquele** repositório também | a construção conclui; sem isso o namespace não resolve |
| Núcleo do ecossistema embutido no artefato | reconstruir com **versão de ecossistema inédita** | um valor já usado antes **não serve**: a etapa de instalação está em cache |
| Manifesto de permissões | reprovisionar | o manifesto vem do artefato congelado; corrigir o arquivo e atualizar a cópia instalada não muda nada |
| Método novo de API | implementar **e** declarar no manifesto, e reprovisionar | a chamada responde; `grep` no código **não** é prova de publicação |

A regra que fecha a matriz: **quanto mais longe do clone de trabalho está o
alvo, mais etapas precisam ser refeitas** — e nunca menos.

## Nunca confie no código de saída

Comandos de provisionamento normalmente respondem quando o **pedido é aceito**.
A construção acontece depois, em segundo plano. O código de saída diz que o
pedido foi recebido, não que a mudança está no ar.

Verifique o **fato**: o estado da construção, o identificador da instância em
execução e a resposta do serviço.

## Cache de construção

Etapas de construção são reaproveitadas quando nada acima delas muda. Isso
produz um sintoma confuso: **"funciona para ele e não para mim"** quando ambos
estão na mesma versão de tudo. A diferença é cache, não configuração.

É por isso que atualizar o núcleo exige um **identificador de versão inédito** —
não basta o conteúdo ter mudado, o valor que decide o reaproveitamento precisa
ser novo.

## Quando a mudança cruza repositórios

Os repositórios do núcleo publicam separadamente, com ciclos próprios. Uma
mudança que atravessa mais de um exige ordem:

1. **Primeiro o consumidor tolerante** — o que sobrevive à ausência do que ainda
   não existe. Depois o repositório que remove ou renomeia. Nunca o inverso.
2. **Mover ou renomear biblioteca compartilhada é mudança incompatível.** Os
   binários publicados resolvem bibliotecas do repositório **instalado**;
   remover uma quebra todo binário que a usava, e o sintoma aparece longe.
3. **Versão sobe junto com a mudança.** Onde a publicação é disparada por
   alteração de versão, um commit sem esse incremento é uma correção que não
   existe para quem consome release.
4. **Referência a "a mais recente" não é fixada.** Publicar uma release muda o
   que toda construção futura instala, sem ninguém pedir.
5. **Depois de publicar, force uma construção de verificação.** Sem isso, o cache
   esconde a quebra por dias.

## Ver também

- [Referência de Comandos](./REFERENCIA-COMANDOS.md)
- [Guia de Log](./GUIA-LOG.md) — onde procurar quando não subiu
- [Declared Resources Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/declared-resources-standard.md)
- [Package Permissions Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/package-permissions-standard.md)
