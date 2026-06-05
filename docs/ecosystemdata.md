# EcosystemData — Diretório do Ecossistema

O **EcosystemData** é o diretório onde o ecossistema **instalado** vive (gerado,
fora do controle de versão; padrão `~/EcosystemData`). É montado pelo
[Setup Wizard](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line).

A especificação formal é o
[Ecosystem Data Directory Hierarchy Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/ecosystem-data-directory-hierarchy-standard.md);
os nomes dos diretórios vêm de
[`ecosystem-defaults.json`](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/metadados/ecosystem-defaults.json).

```
EcosystemData/
├── binaries/             # binários necessários para executar pacotes
├── config-files/         # configuração do ecossistema (ex.: ecosystem-defaults.json)
├── npm-dependencies/     # dependências NPM mínimas usadas pelo package executor
├── environments/         # 1 runtime environment por package executado (nome + hash)
├── sockets/              # sockets Unix de IPC entre serviços
├── supervisor-sockets/   # sockets de supervisão (gRPC sobre Unix socket)
├── executables/          # executáveis instalados (precisa estar no PATH)
├── repos/                # repositórios instalados
├── sources.json          # fontes de repositório disponíveis para instalar
└── repositories.json     # registro dos repositórios instalados
```

| Item | Papel |
|------|-------|
| `binaries/` | Binários necessários para a execução de pacotes quando o SO não os fornece. |
| `config-files/` | Configuração que padroniza a execução (inclui `ecosystem-defaults.json`). |
| `npm-dependencies/` | Dependências mínimas para o package executor carregar um pacote. |
| `environments/` | Um [Runtime Environment](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/runtime-environment.md) isolado por execução (contém `metadata-hierarchy.json`, `execution-params.json`, `.dependencies/`). |
| `sockets/` | Sockets Unix de IPC entre processos/serviços (alta velocidade, baixo acoplamento). |
| `supervisor-sockets/` | Sockets de [supervisão](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/supervisor-socket-standard.md) (gRPC). |
| `executables/` | Comandos do ecossistema (equivalente a `bin`). **Deve estar no `PATH`.** |
| `repos/` | Repositórios instalados. |
| `sources.json` | Fontes registradas (de onde instalar/atualizar repositórios). |
| `repositories.json` | Registro dos repositórios instalados e suas aplicações. |

> **`sockets/` vs `supervisor-sockets/`:** o primeiro é para **IPC entre
> serviços** (HTTP/aplicação); o segundo é para **supervisão** de processos
> (gRPC). São diretórios distintos.
