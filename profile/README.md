# Meta Platform

**Meta Platform** é um **padrão e ecossistema modular** para **construir,
instalar, executar e supervisionar** pacotes de software — com foco em
atomização, modularização, componentização e **governança** de ecossistemas.

Não é uma aplicação única nem apenas um conjunto de CLIs: é um modelo no qual
tudo que é executável ou reutilizável vira um **package** autodescritivo,
organizado em **repositories** e executado por um **ecosystem** instalado na
máquina.

## Por que ela existe

Sistemas corporativos crescem acoplados e difíceis de evoluir. A Meta Platform
ataca isso padronizando **como** software é empacotado, descoberto, resolvido
(por namespace, sem acoplamento ao disco), executado em ambientes isolados e
supervisionado — permitindo que pessoas e empresas montem e governem o seu
próprio ecossistema (como o **My Platform**, mantido pela KADISK).

## A arquitetura em 5 minutos

```
Installation Profile → Setup Wizard → EcosystemData → Repositories
   → applications.json → Executables → Package Executor
   → Runtime Environment → Task Executor → Package Instance → Supervisor Socket
```

- Um **Installation Profile** diz o que instalar; o **Setup Wizard** (`mywizard`)
  monta o **EcosystemData** (o ecossistema instalado).
- Cada **Repository** publica **Executables** em `applications.json`.
- Ao rodar um executável, o **Package Executor** cria um **Runtime Environment**
  isolado, resolve as dependências por namespace e gera o plano de execução.
- O **Task Executor** instancia o package; um **Supervisor Socket** (gRPC sobre
  Unix socket) permite acompanhar e controlar o processo.

Detalhes: [execution-flow](https://github.com/Meta-Platform/.github/blob/main/docs/execution-flow.md)
· [ecosystemdata](https://github.com/Meta-Platform/.github/blob/main/docs/ecosystemdata.md)
· [glossário](https://github.com/Meta-Platform/.github/blob/main/docs/glossario.md).

## Repositórios principais

| Repositório | Papel |
|-------------|-------|
| [meta-platform-open-standard](https://github.com/Meta-Platform/meta-platform-open-standard) | A especificação aberta (conceitos e standards). |
| [meta-platform-setup-wizard-command-line](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line) | Instalador do ecossistema (`mywizard`). |
| [meta-platform-essential-repository](https://github.com/Meta-Platform/meta-platform-essential-repository) | Runtime e libs essenciais; CLIs `repo`/`supervisor`. |
| [meta-platform-ecosystem-core-repository](https://github.com/Meta-Platform/meta-platform-ecosystem-core-repository) | Núcleo operacional: instâncias, serviços e painéis. |
| [meta-platform-package-executor-command-line](https://github.com/Meta-Platform/meta-platform-package-executor-command-line) | Executor isolado de pacotes (`pkg-exec`). |
| [meta-platform-cli-script-loader-library](https://github.com/Meta-Platform/meta-platform-cli-script-loader-library) | Bootstrap/carregador de scripts das CLIs. |
| [meta-platform-applications-repository](https://github.com/Meta-Platform/meta-platform-applications-repository) | Aplicações construídas sobre a plataforma. |

Mapa completo: [repository-map](https://github.com/Meta-Platform/.github/blob/main/docs/repository-map.md).

> **Qual ler primeiro?** Comece pelo
> [meta-platform-open-standard](https://github.com/Meta-Platform/meta-platform-open-standard)
> (conceitos), depois o
> [Guia de Início Rápido](https://github.com/Meta-Platform/.github/blob/main/docs/GUIA-INICIO-RAPIDO.md).

## Como instalar

```bash
# baixar o wizard (release binária)
wget https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line/releases/latest/download/meta-platform-setup-wizard-command-line-linux-x64 -O mywizard
chmod +x mywizard

# instalar um ecossistema
./mywizard list-profiles
./mywizard install --profile release-standard
```

Garanta que `EcosystemData/executables` esteja no `PATH`. Passo a passo:
[Guia de Início Rápido](https://github.com/Meta-Platform/.github/blob/main/docs/GUIA-INICIO-RAPIDO.md).

## Como executar o primeiro pacote

```bash
# usando o ecossistema instalado
repo list installed
executor-manager
supervisor status instance-manager.sock

# ou execução isolada de um package (baixo nível)
pkg-exec --package "/caminho/do/pacote" \
         --startupJson "/caminho/startup-params.json" \
         --ecosystemDefault "/caminho/ecosystem-defaults.json" \
         --nodejsProjectDependencies "/caminho/nodejs-dependencies" \
         --ecosystemData "/caminho/EcosystemData"
```

## Documentação

| Documento | Conteúdo |
|-----------|----------|
| [Open Standard](https://github.com/Meta-Platform/meta-platform-open-standard) | Especificação aberta (conceitos + standards). |
| [Glossário](https://github.com/Meta-Platform/.github/blob/main/docs/glossario.md) | Termos canônicos. |
| [Mapa de Repositórios](https://github.com/Meta-Platform/.github/blob/main/docs/repository-map.md) | Papel de cada repositório. |
| [Fluxo de Execução](https://github.com/Meta-Platform/.github/blob/main/docs/execution-flow.md) | Do perfil à instância supervisionada. |
| [EcosystemData](https://github.com/Meta-Platform/.github/blob/main/docs/ecosystemdata.md) | Hierarquia do ecossistema instalado. |
| [Guia de Início Rápido](https://github.com/Meta-Platform/.github/blob/main/docs/GUIA-INICIO-RAPIDO.md) · [Arquitetura](https://github.com/Meta-Platform/.github/blob/main/docs/ARQUITETURA.md) · [Criar um Pacote](https://github.com/Meta-Platform/.github/blob/main/docs/GUIA-CRIAR-PACOTE.md) · [Desenvolvimento](https://github.com/Meta-Platform/.github/blob/main/docs/GUIA-DESENVOLVIMENTO.md) · [Referência de Comandos](https://github.com/Meta-Platform/.github/blob/main/docs/REFERENCIA-COMANDOS.md) | Guias práticos. |

## Roadmap e estado atual

A plataforma está em **versões iniciais (preview)**. Já funcionam: instalação via
perfis (`mywizard`), repositórios essential/ecosystem-core, execução e supervisão
de pacotes, e o executor isolado (`pkg-exec`).

Em evolução / planejado (resumo): melhorias de resolução de dependências e
`SmartRequire`, fonte `LOCAL_FS_LINK`, novos níveis de log, empacotamento
(Electron/Flatpak/container), tema escuro nos painéis, e aplicações do
[applications-repository](https://github.com/Meta-Platform/meta-platform-applications-repository)
ainda em estágio inicial (ex.: `MetaCloud`, `home-screen`, `my-workspace`).

> Inconsistências conhecidas de cada ferramenta são registradas no respectivo
> `docs/known-issues.md` do repositório.

## Licença

Componentes sob **BSD-3-Clause**; o
[Open Standard](https://github.com/Meta-Platform/meta-platform-open-standard) sob
**GPL-3.0**. Veja o `LICENSE` de cada repositório.

**Contato:** Kaio Cezar — kadisk.shark@gmail.com
