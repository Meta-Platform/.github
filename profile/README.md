# Meta Platform

**Meta Platform** é um padrão e um modo de desenvolvimento de software que
promove **atomização, modularização e componentização** de sistemas — permitindo
criar soluções mais escaláveis, flexíveis, interoperáveis e fáceis de manter.

Tudo que é executável ou reutilizável é empacotado em uma unidade pequena e
autodescritiva — o **pacote** — organizada em repositórios e executada por um
**ecossistema** instalado na máquina. Em torno desse núcleo, um conjunto de
componentes oficiais (e extensões da comunidade) permite que pessoas e empresas
estruturem, expandam e governem o seu próprio ecossistema tecnológico.

---

## 📚 Documentação

| Documento | Conteúdo |
|-----------|----------|
| [Guia de Início Rápido](https://github.com/Meta-Platform/.github/blob/main/docs/GUIA-INICIO-RAPIDO.md) | Instalar um ecossistema e usar os comandos do dia a dia. |
| [Arquitetura e Organização](https://github.com/Meta-Platform/.github/blob/main/docs/ARQUITETURA.md) | Hierarquia `Repository → Module → Layer → Group → Package` e o ecossistema. |
| [Guia: Criar um Pacote](https://github.com/Meta-Platform/.github/blob/main/docs/GUIA-CRIAR-PACOTE.md) | Tutorial passo a passo (biblioteca e CLI). |
| [Guia de Desenvolvimento](https://github.com/Meta-Platform/.github/blob/main/docs/GUIA-DESENVOLVIMENTO.md) | Estilo de código, convenções de nomes e criação de arquivos. |
| [Referência de Comandos](https://github.com/Meta-Platform/.github/blob/main/docs/REFERENCIA-COMANDOS.md) | Todos os comandos de cada CLI em detalhe. |

---

## 🧩 Componentes (ferramentas)

| Repositório | Comando | Função |
|-------------|---------|--------|
| [meta-platform-open-standard](https://github.com/Meta-Platform/meta-platform-open-standard) | (docs) | O padrão aberto: hierarquia, metadados, RPC do executor, etc. |
| [meta-platform-setup-wizard-command-line](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line) | `mywizard` | Instala e atualiza ecossistemas a partir de perfis. |
| [meta-platform-package-executor-command-line](https://github.com/Meta-Platform/meta-platform-package-executor-command-line) | `pkg-exec` | Executa um pacote isolado, com supervisão. |
| [meta-platform-cli-script-loader-library](https://github.com/Meta-Platform/meta-platform-cli-script-loader-library) | (lib) | Prepara o ambiente e carrega scripts para CLIs. |

## 📦 Repositórios oficiais de pacotes

| Repositório | Conteúdo |
|-------------|----------|
| [meta-platform-essential-repository](https://github.com/Meta-Platform/meta-platform-essential-repository) | Núcleo: runtime, executor de tarefas, libs comuns e as CLIs `repo`/`supervisor`. |
| [meta-platform-ecosystem-core-repository](https://github.com/Meta-Platform/meta-platform-ecosystem-core-repository) | Gerenciador de instâncias, painéis de controle e serviços do ecossistema. |
| [meta-platform-applications-repository](https://github.com/Meta-Platform/meta-platform-applications-repository) | Aplicações de usuário final. |

---

## 🚀 Início rápido

```bash
# 1. baixar o wizard (release binária)
wget https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line/releases/latest/download/meta-platform-setup-wizard-command-line-linux-x64 -O mywizard
chmod +x mywizard

# 2. instalar um ecossistema
./mywizard list-profiles
./mywizard install --profile release-standard

# 3. usar os comandos (garanta EcosystemData/executables no PATH)
repo list installed
```

O passo a passo completo está no
[Guia de Início Rápido](https://github.com/Meta-Platform/.github/blob/main/docs/GUIA-INICIO-RAPIDO.md).

---

## Licença

Os componentes são distribuídos sob licença **BSD-3-Clause** (o
[Open Standard](https://github.com/Meta-Platform/meta-platform-open-standard)
sob **GPL-3.0**). Veja o arquivo `LICENSE` de cada repositório.

## Contato

Kaio Cezar — kadisk.shark@gmail.com
