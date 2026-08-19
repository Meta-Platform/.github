# EcosystemData — o diretório do ecossistema

O **EcosystemData** é onde o ecossistema **instalado** vive: gerado, fora do
controle de versão, por padrão em `~/EcosystemData`. Quem o monta é o
[Setup Wizard](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line).

> **A hierarquia completa é normativa e vive no Open Standard:**
> [Ecosystem Data Directory Hierarchy Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/ecosystem-data-directory-hierarchy-standard.md).
> Esta página não repete a árvore — ela existiria em duas versões, e divergiria.

O que um operador precisa saber para começar:

| Diretório | Por que importa na prática |
|---|---|
| `executables/` | Os comandos do ecossistema. **Precisa estar no `PATH`**, senão nada é encontrado. |
| `repos/` | Os repositórios instalados. **É o que executa** — não o seu diretório de trabalho. |
| `logs/` | Onde procurar quando algo não subiu. Ver o [Guia de Log](./GUIA-LOG.md). |
| `environments/` | Um ambiente isolado por execução; é onde se vê o que o executor montou. |
| `sockets/` e `supervisor-sockets/` | Diretórios **distintos**: o primeiro é IPC entre serviços, o segundo é supervisão de processos. Confundi-los é causa comum de conexão recusada. |

Os nomes dos diretórios não são fixos no código: vêm de
[`ecosystem-defaults.json`](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/metadados/ecosystem-defaults.json).
