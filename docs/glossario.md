# Glossário Canônico — Meta Platform

Definições oficiais dos termos da Meta Platform. Os **nomes técnicos em inglês**
(package, executable, runtime environment, supervisor socket, task executor,
metadata hierarchy, execution params) são preservados por serem nomes de
conceitos do sistema.

| Termo | Definição |
|-------|-----------|
| **Meta Platform** | Padrão e ecossistema modular para **construir, instalar, executar e supervisionar** pacotes de software. Não é uma aplicação única nem só um conjunto de CLIs: é um modelo de atomização, modularização, componentização e governança de ecossistemas. |
| **My Platform** | Ecossistema concreto, baseado na Meta Platform, mantido pela KADISK Engenharia de Software. Serve como base pronta para uso e como referência para criar um padrão próprio (a distribuição de referência é o repositório privado `my-platform-reference-distro`). |
| **Open Standard** | A especificação aberta e independente de implementação da Meta Platform. Repositório: [meta-platform-open-standard](https://github.com/Meta-Platform/meta-platform-open-standard). |
| **Ecosystem** (ecossistema) | O conjunto instalado e em execução de repositórios, executáveis e dados que rodam a plataforma em uma máquina. Materializado no diretório **EcosystemData**. |
| **EcosystemData** | Diretório (gerado, fora do controle de versão; padrão `~/EcosystemData`) onde o ecossistema instalado vive. Ver [ecosystemdata.md](./ecosystemdata.md) e [Ecosystem Data Directory Hierarchy Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/ecosystem-data-directory-hierarchy-standard.md). |
| **Repository** | Unidade distribuível e versionável que agrupa packages. Ver [Repository](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/repository.md). |
| **Module** (`*.Module`) | Divisão macro de responsabilidade dentro de um repositório (ex.: `Commons.Module`, `Runtime.Module`, `Main.Module`). |
| **Layer** (`*.layer`) | Camada que agrupa packages da mesma preocupação/tipo (ex.: `Libraries.layer`, `Application.layer`). |
| **Group** (`*.group`) | Agrupamento opcional dos packages que, juntos, formam uma aplicação (ex.: `APIDesigner.group`). |
| **Package** | A unidade atômica e autodescritiva da plataforma; pasta com sufixo de tipo e uma pasta `metadata/`. Ver [Package](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/package.md). |
| **Executable** | Ponto de entrada nomeado que um package expõe (comando no terminal ou app/serviço iniciado pelo ecossistema). Declarado em `boot.json` (package) e `applications.json` (repositório). Ver [Executable](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/executable.md). |
| **Application** | Package que representa uma aplicação do ecossistema (sufixo `.app`), capaz de subir serviços e endpoints. |
| **App** | Valor de `appType` em `applications.json` para aplicações (em oposição a `CLI`). Tipicamente vem de packages `.app`/`.webapp`. |
| **CLI** | Aplicação de linha de comando (package `.cli`); expõe um comando no terminal. `appType: "CLI"`. |
| **Service** | Serviço de back-end (package `.service`); expõe serviços listados em `services.json`, consumidos por outros packages. |
| **WebApp** | Aplicação web composta (package `.webapp`); integra webgui + webservice sobre um servidor HTTP. |
| **DesktopApp** | Aplicação desktop nativa (package `.desktopapp`); abre uma ou mais janelas Electron; tipicamente encapsula uma app web local que sobe junto (`loadURL`); também suporta HTML local (`loadFile`). `appType: "DESKTOP"`. |
| **WebGUI** | Interface web / front-end (package `.webgui`). |
| **WebService** | Serviço HTTP / API (package `.webservice`); expõe um `endpoint-group`. |
| **Library** | Biblioteca reutilizável (package `.lib`; `.nativelib` para nativas); consumida por outros packages via namespace. |
| **Runtime Environment** | Diretório isolado criado para cada execução de um package, em `EcosystemData/environments/` (nome do package + hash). Contém `metadata-hierarchy.json`, `execution-params.json` e `.dependencies/`. Ver [Runtime Environment](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/runtime-environment.md). |
| **Package Executor** | Ferramenta que executa um package isolado: cria o runtime environment, resolve dependências, gera o execution params e dispara o task executor. Repositório: [meta-platform-package-executor-command-line](https://github.com/Meta-Platform/meta-platform-package-executor-command-line) (comando `pkg-exec`). |
| **Setup Wizard** | Instalador do ecossistema a partir de installation profiles. Repositório: [meta-platform-setup-wizard-command-line](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line) (comando `mywizard`). |
| **Script Loader** | Biblioteca que prepara o ambiente e carrega scripts/libs da plataforma para aplicações CLI (bootstrap). Repositório: [meta-platform-cli-script-loader-library](https://github.com/Meta-Platform/meta-platform-cli-script-loader-library). |
| **Task Executor** | Motor que lê o `execution-params.json` e instancia cada unidade via object loaders, gerenciando o ciclo de vida das tasks. Implementado na `task-executor.lib` do essential-repository. Ver [Environment Runtime Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/environment-runtime-standard.md). |
| **Supervisor Socket** | Socket Unix em `EcosystemData/supervisor-sockets/` por onde um processo expõe sua interface gRPC de supervisão. Ver [Supervisor Socket Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/supervisor-socket-standard.md). |
| **Metadata Hierarchy** | Grafo de metadados (`metadata-hierarchy.json`) de um runtime environment: o package raiz + todas as dependências resolvidas por namespace. |
| **Execution Params** | Plano de execução (`execution-params.json`) derivado da metadata hierarchy; lista as unidades (object loaders) e suas ligações. Ver [Execution Params Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/packages/execution-params-standard.md). |
| **Installation Profile** | Arquivo `*.install.json` que descreve o que instalar (repositórios e executáveis) e onde. Consumido pelo Setup Wizard. Ver [Ecosystem Installation Profile Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/ecosystem-installation-profile-standard.md). |
