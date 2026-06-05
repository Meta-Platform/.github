# Fluxo de Execução — Meta Platform

Como um pacote sai de uma instalação até virar uma **instância em execução
supervisionada**. Cada etapa é confirmada na implementação de referência.

```
Installation Profile
      ↓                (mywizard lê o perfil *.install.json)
Setup Wizard
      ↓                (monta o ecossistema)
EcosystemData
      ↓                (repos/, executables/, config-files/, ...)
Repositories
      ↓                (instalados em EcosystemData/repos/)
metadata/applications.json
      ↓                (lista executable ↔ packageNamespace ↔ supervisorSocketFileName)
Executables
      ↓                (scripts em EcosystemData/executables/, no PATH)
Package Executor
      ↓                (cria ambiente isolado, resolve dependências por namespace)
Runtime Environment
      ↓                (metadata-hierarchy.json → execution-params.json + .dependencies/)
Task Executor
      ↓                (instancia cada unidade via object loaders)
Package Instance
      ↓                (CLI / service / endpoints no ar)
Supervisor Socket
                       (gRPC sobre Unix socket: status, tasks, log, kill)
```

## Detalhe por etapa

1. **Installation Profile** → o usuário escolhe um perfil
   ([`*.install.json`](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/ecosystem-installation-profile-standard.md)).
2. **Setup Wizard** (`mywizard`) → resolve fontes, baixa/copia repositórios e cria
   os executáveis.
3. **EcosystemData** → diretório do ecossistema instalado
   ([hierarquia](./ecosystemdata.md)).
4. **Repositories** → ficam em `EcosystemData/repos/`.
5. **applications.json** → liga `executable` ↔ `packageNamespace` ↔
   `supervisorSocketFileName`
   ([Repository Metadata Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/repository-metadata-standard.md)).
6. **Executables** → scripts em `EcosystemData/executables/` (devem estar no `PATH`).
7. **Package Executor** → cria o ambiente isolado e resolve as dependências por
   namespace
   ([package-executor](https://github.com/Meta-Platform/meta-platform-package-executor-command-line)).
8. **Runtime Environment** → `metadata-hierarchy.json` é traduzido em
   `execution-params.json`
   ([Runtime Environment](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/runtime-environment.md)).
9. **Task Executor** → executa o plano instanciando cada unidade via **object
   loaders**
   ([Environment Runtime Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/environment-runtime-standard.md)).
10. **Package Instance** → o pacote no ar (CLI, serviço, endpoints).
11. **Supervisor Socket** → supervisão via gRPC
    ([Supervisor Socket Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/supervisor-socket-standard.md)).
