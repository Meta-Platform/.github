# Guia de Início Rápido

Este guia mostra como instalar um ecossistema **Meta Platform** e usar os
comandos do dia a dia. Para entender a organização do projeto, veja
[Arquitetura](./ARQUITETURA.md).

> **Pré-requisitos:** Git instalado e ambiente Linux x64 (as releases binárias
> são publicadas para `linux-x64`). **Node.js** só é necessário para instalar a
> partir do código (Opção B) — veja as notas de cada cenário abaixo.

---

## 1. Instalar o Setup Wizard (`mywizard`)

O `mywizard` é a porta de entrada: ele instala e atualiza ecossistemas.

### Opção A — usar a release binária

```bash
# baixe a última release (ajuste a versão se necessário)
wget https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line/releases/download/0.0.19/meta-platform-setup-wizard-command-line-0.0.19-preview-linux-x64 -O mywizard
chmod +x mywizard
./mywizard --help
```

> O binário é **autocontido** (empacotado com [`pkg`](https://github.com/yao-pkg/pkg),
> target `node22-linux-x64` no script `build`): **não exige Node.js instalado**
> para ser executado.

### Opção B — a partir do código (desenvolvimento)

```bash
cd Meta-Platform/meta-platform-setup-wizard-command-line
npm install
npm link        # disponibiliza o comando `mywizard` globalmente
```

> Requer **Node.js** instalado. O `package.json` não declara `engines`; o build
> oficial visa `node22-linux-x64`, então uma versão LTS atual (22+) atende.

---

## 2. Escolher um perfil e instalar

Liste os perfis disponíveis:

```bash
mywizard list-profiles
```

Perfis **registrados** (selecionáveis por nome):

| Perfil | Origem dos pacotes | Repositórios |
|--------|--------------------|--------------|
| `release-minimal`        | Releases do GitHub        | `EssentialRepo` |
| `release-standard`       | Releases do GitHub        | `EssentialRepo` + `EcosystemCoreRepo` |
| `localfs-release-standard` | Sistema de arquivos local | `EssentialRepo` + `EcosystemCoreRepo` |

Instale informando **sempre** um perfil registrado:

```bash
# uso/produção
mywizard install --profile release-standard
mywizard install --profile release-minimal

# desenvolvimento a partir do disco
mywizard install --profile localfs-release-standard

# escolhendo onde instalar
mywizard install --profile release-standard --installation-path "~/meu-projeto/EcosystemData"
```

> Detalhes (e inconsistências conhecidas, como o `show-profile` e o default
> `standard`) em
> [installation-profiles](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line/blob/main/docs/installation-profiles.md)
> e [known-issues](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line/blob/main/docs/known-issues.md).

> Após instalar, garanta que `EcosystemData/executables` esteja no seu `PATH`
> para usar os comandos abaixo (`repo`, `supervisor`, `executor`, …).

---

## 3. Atualizar um ecossistema

```bash
mywizard update --profile localfs-release-standard
mywizard update --profile release-standard --installation-path "~/meu-projeto/EcosystemData"
```

---

## 4. Gerenciar repositórios (`repo`)

A CLI `repo` (do `repository-manager.cli`) registra fontes, instala e atualiza
repositórios de pacotes.

```bash
# listar
repo sources              # fontes agregadas disponíveis para instalação
repo list sources         # fontes registradas
repo list installed       # repositórios instalados
repo show EssentialRepo   # detalhes de um repositório

# registrar uma fonte
repo register source MyPersonalRepo GITHUB_RELEASE --repoName "my-personal-repo" --repoOwner "my-user"
repo register source MyPersonalRepo GOOGLE_DRIVE   --fileId "AaBbCc...123456"
repo register source WormsSolutionsRepo LOCAL_FS   --localPath "~/Workspaces/meta-platform-repo/thrid-party-repos/WormsSolutionsRepo"

# instalar / atualizar
repo install WormsSolutionsRepo LOCAL_FS
repo install WormsSolutionsRepo LOCAL_FS --executables _3dview _3dservice "worms-website"
repo update  WormsSolutionsRepo

repo update EssentialRepo
repo update EcosystemCoreRepo
```

Tipos de fonte (`sourceType`): `GITHUB_RELEASE`, `GOOGLE_DRIVE`, `LOCAL_FS`.

---

## 5. Supervisionar processos (`supervisor`)

Cada processo em execução pode expor um socket de supervisão. A CLI `supervisor`
inspeciona e controla esses processos.

```bash
supervisor sockets                      # lista os sockets de supervisão

supervisor status <SOCKET>              # status do processo
supervisor tasks  <SOCKET>              # tarefas em execução
supervisor log    <SOCKET>              # streaming de log
supervisor kill   <SOCKET>              # encerra o processo
supervisor show task <TASK_ID> <SOCKET> # detalhes de uma tarefa

# exemplos com o instance-manager
supervisor status instance-manager.sock
supervisor tasks  instance-manager.sock
supervisor log    instance-manager.sock
supervisor show task 46 instance-manager.sock
```

---

## 6. Executar pacotes (`executor` / `pkg-exec`)

Para iniciar o gerenciador de instâncias e executar pacotes avulsos:

```bash
executor-manager            # inicia o gerenciador de instâncias do ecossistema

executor package            # executa um pacote
executor tasks              # lista tarefas do executor
executor show task 50       # detalhes de uma tarefa
```

Para execução isolada de baixo nível (fora do ecossistema), use `pkg-exec` —
veja o
[README do package-executor](https://github.com/Meta-Platform/meta-platform-package-executor-command-line/blob/main/README.md).

---

## Fluxo típico de desenvolvimento local

```bash
# 1. instalar o ambiente apontando para os repos locais
mywizard install --profile localfs-release-standard

# 2. registrar/instalar um repositório local
repo register source EssentialRepo LOCAL_FS --localPath ~/Workspaces/meta-platform-repo/repos/essential-repository
repo install EssentialRepo LOCAL_FS

# 3. subir o gerenciador de instâncias e acompanhar
executor-manager
supervisor status instance-manager.sock
supervisor log    instance-manager.sock

# 4. ao alterar código, atualizar
mywizard update --profile localfs-release-standard
repo update EssentialRepo
```

---

## Próximos passos

- [Arquitetura e Organização](./ARQUITETURA.md)
- [Guia: Criar um Pacote](./GUIA-CRIAR-PACOTE.md)
- [Referência de Comandos](./REFERENCIA-COMANDOS.md) — todos os comandos de
  `mywizard`, `repo`, `supervisor`, `executor`, `mypkg` e `pkg-exec` em detalhe.
