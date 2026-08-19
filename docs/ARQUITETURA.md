# Arquitetura e Organização do Meta Platform

Este documento explica como o código do **Meta Platform** está organizado, quais
são os conceitos centrais da plataforma e como os diretórios se relacionam entre
si. É a referência recomendada antes de criar um novo pacote ou contribuir com a
plataforma.

> Para a definição formal e independente de implementação, consulte o
> [Meta Platform Open Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/README.md).

---

## Visão geral

O Meta Platform promove **atomização, modularização e componentização** de
sistemas. Na prática, isso significa que tudo que é executável ou reutilizável é
empacotado em uma unidade pequena e autodescritiva chamada **pacote** (*package*).
Pacotes são organizados em **repositórios** e executados — isolados ou em
conjunto — por um **ecossistema** instalado na máquina.

O repositório raiz (`meta-platform-repo`) agrega quatro grandes peças:

| Diretório | Papel |
|-----------|-------|
| [`Meta-Platform/`](https://github.com/Meta-Platform) | Ferramentas e o padrão aberto que **constroem e executam** a plataforma (wizard, executor, loader, open standard). |
| [`repos/`](https://github.com/Meta-Platform) | Os **repositórios oficiais** de pacotes (essential, ecosystem-core, applications, native-applications-lab). |
| [`thrid-party-repos/`](https://github.com/Meta-Platform) | Repositórios de terceiros / experimentais. ⚠️ `thrid-party-repos` é o nome **legado** do diretório (typo de *third*); a documentação mantém esse nome para refletir o path real. Uma futura migração pode padronizar para `third-party-repos`. |
| `EcosystemData/` | (gerado, fora do controle de versão) O ecossistema **instalado** na máquina. Veja [Diretório do Ecossistema](#o-diretório-do-ecossistema-ecosystemdata). |

---

## Ferramentas centrais (`Meta-Platform/`)

| Ferramenta | Comando | Função |
|------------|---------|--------|
| [meta-platform-setup-wizard-command-line](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line/blob/main/README.md) | `mywizard` | Instala e atualiza ecossistemas a partir de **perfis de instalação**. |
| [meta-platform-package-executor-command-line](https://github.com/Meta-Platform/meta-platform-package-executor-command-line/blob/main/README.md) | `pkg-exec` | Executa um pacote isolado, dentro ou fora de um ecossistema, com supervisão. |
| [meta-platform-cli-script-loader-library](https://github.com/Meta-Platform/meta-platform-cli-script-loader-library/blob/main/README.md) | (lib) | Prepara o ambiente e carrega scripts para aplicações CLI. |
| [meta-platform-open-standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/README.md) | (docs) | Especificação aberta: hierarquia de diretórios, metadados, RPC do executor, etc. |

---

## A hierarquia de um repositório

Pacotes não ficam soltos: são agrupados numa hierarquia que dá contexto e
namespace a cada unidade.

```
Repository            um repositório distribuível
└── Module            (*.Module)  divisão macro de responsabilidade
    └── Layer         (*.layer)   camada de mesma preocupação
        ├── Package   (*.cli, *.lib, *.service, …)  a folha
        └── Group     (*.group)   pacotes que juntos formam uma aplicação
            └── Package
```

**As definições são normativas e vivem no Open Standard** — esta página não as
repete: [Repository](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/repository.md),
[Module / Layer / Group](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/module-layer-group.md),
[Package](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/package.md) (com a **tabela completa dos tipos** e os
prefixos de namespace `@/`, `@@/`, `@//`) e
[Executable](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/executable.md).

## Anatomia de um pacote

Um pacote é uma pasta autodescritiva. O coração de qualquer pacote é a pasta
`metadata/`, que torna o pacote compreensível para a plataforma sem precisar
executá-lo. Exemplo real (`repository-manager.cli`):

```
repository-manager.cli/
├── metadata/
│   ├── package.json         # namespace do pacote: { "namespace": "@/repository-manager.cli" }
│   ├── boot.json            # executáveis expostos e seus parâmetros vinculados (bound-params)
│   ├── command-group.json   # (CLI) árvore de comandos, parâmetros e libs a carregar
│   └── startup-params.json  # parâmetros de inicialização (ex.: installDataDirPath)
├── src/
│   ├── Commands/            # implementação de cada comando
│   ├── Helpers/
│   └── Configs/
├── package.json             # dependências Node.js do pacote
└── README.md
```

### Os metadados principais

- **`package.json` (metadata)** — declara o `namespace` do pacote. O namespace
  usa o prefixo `@/` para referenciar um package por namespace no conjunto de
  repositórios instalados — declarado dentro de um repositório, mas resolvido
  globalmente no `EcosystemData` (ex.: `@/repository-manager.cli`); `@@/` para
  instâncias/serviços e `@//` para referências internas de boot.
- **`boot.json`** — descreve o que o pacote *expõe* ao ser carregado. Para uma
  CLI, lista `executables` (o `executableName` é o comando que aparece no
  terminal, ex.: `repo`) e os `bound-params` — dependências resolvidas por
  namespace (ex.: `@/ecosystem-install-utilities.lib`).
- **`command-group.json`** (apenas CLIs) — a árvore de comandos. Cada comando tem
  `command` (assinatura no estilo yargs, ex.: `install [repositoryNamespace]
  [sourceType]`), `path` (arquivo do handler), `description`, `parameters`
  (positional/option) e `parametersToLoad` (libs injetadas no handler). Comandos
  podem ter `children` para subcomandos.
- **`startup-params.json`** — valores de inicialização específicos do pacote.

> Referências por **namespace** (em vez de caminhos relativos) são o que permite
> que pacotes se liguem entre si sem acoplamento físico ao sistema de arquivos. A
> resolução acontece no momento da execução, dentro do ecossistema.

---

## O diretório do ecossistema (`EcosystemData`)

`mywizard install` materializa o ecossistema num diretório gerado, por padrão
`~/EcosystemData`, fora do controle de versão. **É ele que executa** — não o seu
diretório de trabalho.

A árvore completa é normativa:
[Ecosystem Data Directory Hierarchy Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/ecosystem-data-directory-hierarchy-standard.md).
Uma visão prática, com o que importa ao operar, está em
[EcosystemData](./ecosystemdata.md).

## Como tudo se conecta

1. O **wizard** (`mywizard`) lê um **perfil de instalação** e monta o
   `EcosystemData`, instalando repositórios e executáveis.
2. Um pacote é executado pelo **package-executor** (`pkg-exec`), que cria um
   *environment* isolado, resolve as dependências por namespace e gera o
   `execution-params.json`.
3. O **task executor** lê o `execution-params.json` e instancia cada unidade
   (application-instance, service-instance, endpoint-instance, …) via os
   **object loaders** correspondentes — veja
   [Tipos de Object Loader](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/tipos-de-object-loader.md).
4. O **supervisor** acompanha os processos em execução pelos sockets de
   supervisão.

---

## Próximos passos

- [Guia de Início Rápido](./GUIA-INICIO-RAPIDO.md) — instale e use o ecossistema.
- [Guia: Criar um Pacote](./GUIA-CRIAR-PACOTE.md) — crie seu primeiro pacote.
