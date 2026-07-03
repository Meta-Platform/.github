# Referência de Comandos

Referência **completa** dos comandos das ferramentas do Meta Platform, agrupada
por ferramenta. Para uma introdução guiada (passo a passo), veja o
[Guia de Início Rápido](./GUIA-INICIO-RAPIDO.md) — este documento é a
referência detalhada que o complementa.

Os comandos abaixo foram conferidos no código-fonte (`metadata/command-group.json`
de cada CLI e os `Executables` do `mywizard`/`pkg-exec`).

> **Pré-requisito:** o diretório `EcosystemData/executables` precisa estar no
> `PATH` para usar `repo`, `supervisor`, `executor`, etc.

---

## `mywizard` — Setup Wizard

Instala e atualiza ecossistemas a partir de perfis. Fonte:
[`meta-platform-setup-wizard-command-line`](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line/blob/main/README.md).

| Comando | Descrição |
|---------|-----------|
| `mywizard list-profiles` | Lista os perfis de instalação registrados. |
| `mywizard install [profile] [installation-path]` | Instala um ecossistema conforme o perfil. |
| `mywizard update [profile] [installation-path]` | Atualiza um ecossistema instalado. |
| `mywizard show-profile [profile]` | ⚠️ Existe no CLI, mas está **quebrado** na versão atual — ver [known-issues](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line/blob/main/docs/known-issues.md). |

- `profile` / `--profile` — nome do perfil. **Informe sempre** um perfil
  registrado (o default interno `standard` não é válido — ver known-issues).
- `installation-path` / `--installation-path` — caminho de instalação alternativo.
- Perfis registrados: `release-minimal`, `release-standard`,
  `localfs-release-standard` (detalhes em
  [installation-profiles](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line/blob/main/docs/installation-profiles.md)).

```bash
mywizard list-profiles
mywizard install --profile release-minimal
mywizard install --profile localfs-release-standard
mywizard install --profile release-standard --installation-path "~/meu-projeto/EcosystemData"
mywizard update  --profile release-standard
```

> Os `mywizard install/update <perfil>` na forma posicional também funcionam
> (ex.: `mywizard install release-minimal`).

---

## `repo` — Repository Manager

Gerencia fontes e repositórios de pacotes. Fonte: `repository-manager.cli`
(`essential-repository`). Tipos de fonte (`sourceType`): `GITHUB_RELEASE`,
`GOOGLE_DRIVE`, `LOCAL_FS`.

| Comando | Descrição |
|---------|-----------|
| `repo sources` | Mostra as fontes agregadas disponíveis para instalação. |
| `repo list sources` | Lista as fontes registradas. |
| `repo list installed` | Lista os repositórios instalados. |
| `repo show [repositoryNamespace]` | Detalhes de um repositório instalado. |
| `repo install [repositoryNamespace] [sourceType]` | Instala um repositório. |
| `repo update [repositoryNamespace]` | Atualiza um repositório instalado. |
| `repo register source [repositoryNamespace] [sourceType]` | Registra uma nova fonte. |
| `repo remove source [repositoryNamespace] [sourceType]` | Remove uma fonte. |
| `repo change installed source [repositoryNamespace] [sourceType]` | Troca a fonte de um repositório instalado. |

Opções (`--flag`):

| Opção | Tipo | Usada em | Descrição |
|-------|------|----------|-----------|
| `--executables` | array | `install` | Executáveis a instalar (subconjunto). |
| `--localPath` | string | fonte `LOCAL_FS` | Caminho do repositório local. |
| `--repoName` | string | fonte `GITHUB_RELEASE` | Nome do repositório no GitHub. |
| `--repoOwner` | string | fonte `GITHUB_RELEASE` | Dono (username) no GitHub. |
| `--fileId` | string | fonte `GOOGLE_DRIVE` | ID do `.tar.gz` no Google Drive. |

```bash
# registrar fontes
repo register source MyPersonalRepo GITHUB_RELEASE --repoName "my-personal-repo" --repoOwner "my-user"
repo register source MyPersonalRepo GOOGLE_DRIVE   --fileId "AaBbCc...123456"
repo register source WormsSolutionsRepo LOCAL_FS   --localPath "~/Workspaces/meta-platform-repo/thrid-party-repos/WormsSolutionsRepo"

# instalar / atualizar
repo install WormsSolutionsRepo LOCAL_FS
repo install WormsSolutionsRepo LOCAL_FS --executables _3dview _3dservice "worms-website"
repo update  WormsSolutionsRepo
repo update  EssentialRepo
repo update  EcosystemCoreRepo

# inspecionar
repo sources
repo list sources
repo list installed
repo show WormsSolutionsRepo
```

---

## `supervisor` — Instance Supervisor

Inspeciona e controla processos pelos sockets de supervisão (gRPC). Fonte:
`instance-supervisor.cli` (`essential-repository`). Contrato:
[Package Executor RPC Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/package-executor-rpc-standard.md).

| Comando | Descrição |
|---------|-----------|
| `supervisor sockets` | Lista os sockets de supervisão. |
| `supervisor status [socket]` | Status da execução da instância. |
| `supervisor tasks [socket]` | Tarefas carregadas no task executor da instância. |
| `supervisor log [socket]` | Streaming de log da instância. |
| `supervisor kill [socket]` | Encerra a instância. |
| `supervisor show task [taskId] [socket]` | Detalhes de uma tarefa da instância. |

```bash
supervisor sockets
supervisor status instance-manager.sock
supervisor tasks  instance-manager.sock
supervisor log    instance-manager.sock
supervisor kill   instance-manager.sock
supervisor show task 46 instance-manager.sock
```

---

## `executor` — Instance Executor

Executa e acompanha pacotes, ambientes e tarefas. Fonte: `instance-executor.cli`
(`ecosystem-core-repository`).

| Comando | Descrição |
|---------|-----------|
| `executor package [path]` | Executa um pacote. |
| `executor env [path]` | Executa um ambiente a partir de um caminho. |
| `executor stop env [executionId]` | Para um ambiente em execução. |
| `executor tasks` | Lista tarefas no task executor global. |
| `executor environments` | Lista ambientes em execução. |
| `executor monitor` | Monitora a atividade do Task Executor. |
| `executor show task [taskId]` | Detalhes de uma tarefa. |

```bash
executor package
executor tasks
executor environments
executor monitor
executor show task 50
```

Comandos relacionados (aplicação principal / executáveis instalados):

```bash
executor-manager            # inicia o gerenciador de instâncias do ecossistema
run package <packagePath>   # executa um pacote (package-runner.cli)
mypkg create library <nome> # cria um pacote (package-toolkit.cli)
```

---

## `mytoolkit` — Maintenance Toolkit

Variante de manutenção do wizard, embutida no `essential-repository`
(`maintenance-toolkit.cli`).

| Comando | Descrição |
|---------|-----------|
| `mytoolkit list-profiles` | Lista os perfis de instalação. |
| `mytoolkit install [profile]` | Instala um ecossistema conforme o perfil. Opções: `--profile-file` (arquivo de perfil de instalação) e `--installation-path` (onde instalar). |
| `mytoolkit update [profile]` | Atualiza um ecossistema conforme o perfil. Opções: `--profile-file` e `--installation-path`. |
| `mytoolkit show profile [profile]` | Mostra informações de um perfil. |

> **Atenção:** o carregador de perfis do `maintenance-toolkit.cli` está
> **quebrado** no momento — ele faz `require` de arquivos de perfil que não
> existem no pacote. Na prática, `list-profiles`, `install` e `update` falham
> com `MODULE_NOT_FOUND`, e `show profile` referencia perfis inexistentes.
> Para instalar/atualizar um ecossistema, use o `mywizard` (acima).

---

## `mypkg` — Package Toolkit

Cria novos pacotes (scaffolding). Fonte: `package-toolkit.cli`
(`ecosystem-core-repository`). Veja o
[Guia: Criar um Pacote](./GUIA-CRIAR-PACOTE.md).

| Comando | Descrição |
|---------|-----------|
| `mypkg create library [packageName]` | Cria uma biblioteca (`.lib`). |
| `mypkg create commandline [packageName]` | Cria uma CLI (`.cli`). |
| `mypkg create services [packageName]` | Cria um pacote de serviços (`.service`). |

> O scaffolding gera apenas estes três tipos (`library`, `commandline`,
> `services`). Outros tipos de pacote — como `.webapp`, `.desktopapp` ou `.app` —
> ainda **não** são gerados pelo `mypkg create` e devem ser criados manualmente
> seguindo o [Guia: Criar um Pacote](./GUIA-CRIAR-PACOTE.md).

---

## `pkg-exec` — Package Executor (baixo nível)

Execução isolada de um pacote, fora do ecossistema, com supervisão opcional.
Fonte e parâmetros completos:
[`meta-platform-package-executor-command-line`](https://github.com/Meta-Platform/meta-platform-package-executor-command-line/blob/main/README.md).

```bash
pkg-exec --package "/caminho/do/pacote" \
         --startupJson "/caminho/startup-params.json" \
         --ecosystemDefault "/caminho/platform-params.json" \
         --nodejsProjectDependencies "/caminho/nodejs-dependencies" \
         --ecosystemData "/caminho/EcosystemData"
```
