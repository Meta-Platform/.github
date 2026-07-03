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

Os pacotes não ficam soltos: eles são agrupados em uma hierarquia de quatro
níveis que dá contexto e namespace a cada unidade.

```
Repository            (ex.: essential-repository)
└── Module            (*.Module — ex.: Main.Module, Runtime.Module, Commons.Module)
    └── Layer         (*.layer — ex.: Application.layer, Libraries.layer)
        ├── Package   (*.cli, *.lib, *.service, ... — folha executável/reutilizável)
        └── Group     (*.group — agrupa pacotes que formam uma aplicação)
            └── Package
```

### Repository
Unidade distribuível e versionável (um repositório Git). Os repositórios
oficiais têm um `README.md` na raiz e um diretório `metadata/` (ex.:
`applications.json`) — a exceção é o `native-applications-lab-repository`,
laboratório experimental que ainda não segue esse padrão. Os repositórios
oficiais ficam em [`repos/`](https://github.com/Meta-Platform):

- [**essential-repository**](https://github.com/Meta-Platform/meta-platform-essential-repository/blob/main/README.md) — o núcleo:
  runtime, executor de tarefas, bibliotecas comuns e as CLIs essenciais
  (`repo`, `supervisor`, etc.).
- [**ecosystem-core-repository**](https://github.com/Meta-Platform/meta-platform-ecosystem-core-repository/blob/main/README.md) —
  o gerenciador de instâncias do ecossistema, painéis de controle e serviços.
- [**applications-repository**](https://github.com/Meta-Platform/meta-platform-applications-repository/blob/main/README.md) —
  aplicações de usuário final (datasource-manager, api-designer,
  package-developer, MyDesktop, etc.).
- **native-applications-lab-repository** — laboratório **experimental** de
  pacotes nativos (`.nativelib`); ainda sem `README.md`/`metadata/` e fora do
  padrão de descoberta da plataforma.

### Module (`*.Module`)
Divisão macro de responsabilidade dentro de um repositório. Convenções
observadas nos repositórios oficiais:

- **`Commons.Module`** — utilitários e bibliotecas compartilhadas.
- **`Runtime.Module`** — peças de tempo de execução (executor de tarefas,
  task loaders, helpers de metadados).
- **`Main.Module`** — as aplicações/CLIs/serviços principais do repositório.
- **`Apps.Module` / `Base.Module`** — aplicações de usuário e suas bases
  reutilizáveis (no applications-repository).

### Layer (`*.layer`)
Camada que agrupa pacotes do mesmo tipo ou da mesma preocupação. Exemplos reais:
`Application.layer`, `Libraries.layer`, `Utilities.layer`,
`PlatformLibraries.layer`, `Services.layer`, `Webservices.layer`,
`Executor.layer`, `MetadataHelpers.layer`, `EssentialTaskLoaders.layer`.

### Group (`*.group`)
Agrupa pacotes que, juntos, formam **uma aplicação completa**. É comum uma
aplicação web ser composta por três pacotes no mesmo group — por exemplo, em
`APIDesigner.group`: `api-designer.webapp`, `api-designer.webgui` e
`api-designer.webservice`.

### Package (folha)
A unidade atômica. O **sufixo** indica o tipo do pacote:

| Sufixo | Tipo | Descrição |
|--------|------|-----------|
| `.cli` | Command-line | Aplicação de linha de comando (expõe um executável). |
| `.lib` | Library | Biblioteca reutilizável (JavaScript). |
| `.nativelib` | Native library | Biblioteca nativa (ex.: addon C++/Rust). **Experimental**: ainda não é reconhecida pela descoberta de pacotes da plataforma (fora de `REPOS_CONF_EXTLIST_PKG_TYPE`). |
| `.service` | Service | Serviço de back-end de longa duração. |
| `.webservice` | Web service | Serviço HTTP / API. |
| `.webgui` | Web GUI | Interface web (front-end). |
| `.webapp` | Web application | Aplicação web (webgui + webservice integrados). |
| `.desktopapp` | Desktop application | Aplicação desktop nativa; abre uma ou mais janelas Electron; tipicamente encapsula uma app web local que sobe junto (`loadURL`); também suporta HTML local (`loadFile`). |
| `.app` | Application | Aplicação/instância do ecossistema. |

---

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

Quando você roda `mywizard install`, a plataforma materializa um **ecossistema
instalado** em um diretório `EcosystemData` (por padrão `~/EcosystemData`). Esse
diretório é gerado e fica fora do controle de versão (veja `.gitignore`). Sua
estrutura é definida pelo
[Ecosystem Data Directory Hierarchy Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/ecosystem-data-directory-hierarchy-standard.md):

```
EcosystemData/
├── binaries/             # binários necessários para executar pacotes
├── config-files/
│   └── ecosystem-defaults.json   # parâmetros padrão de todo o ecossistema
├── npm-dependencies/     # dependências NPM mínimas usadas pelo package-executor
├── environments/         # 1 ambiente isolado por pacote executado (nome + hash)
├── sockets/              # sockets Unix para IPC entre processos
├── supervisor-sockets/   # sockets de supervisão (gRPC)
├── executables/          # executáveis instalados (deve estar no PATH)
├── repos/                # repositórios instalados
├── sources.json          # fontes de repositório disponíveis para instalar
└── repositories.json     # registro dos repositórios instalados
```

Pontos importantes:

- **`environments/`** — cada execução de pacote ganha um diretório isolado,
  nomeado com o nome do pacote + um hash do seu caminho (ex.:
  `repository-manager.cli-e9766fee...`). Dentro dele ficam
  `metadata-hierarchy.json` (tudo que precisa ser carregado) e
  `execution-params.json` (as unidades de execução e suas interdependências,
  consumidas pelo *task executor*) além de `.dependencies/node_modules`.
- **`executables/`** precisa estar no `PATH` para que comandos como `repo` e
  `supervisor` funcionem no terminal.
- **`supervisor-sockets/`** expõe operações gRPC por processo: `KillInstance`,
  `GetStartupArguments`, `GetProcessInformation`, `GetStatus`, `ListTasks`,
  `GetTask`, `LogStreaming` e `StatusChangeNotification`.

---

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
