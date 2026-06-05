# Mapa de Repositórios — Meta Platform

A Meta Platform é composta por **vários repositórios que cooperam**. Cada um tem
um papel específico no ciclo de **construir → instalar → executar → supervisionar**
pacotes. Nenhum deles é um projeto isolado.

| Repositório | Papel no ecossistema | Leitura |
|-------------|----------------------|---------|
| [meta-platform-open-standard](https://github.com/Meta-Platform/meta-platform-open-standard) | **Especificação** aberta (conceitos e standards). | Ler **primeiro** para entender os conceitos. |
| [meta-platform-setup-wizard-command-line](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line) | **Instalador** do ecossistema (`mywizard`) a partir de installation profiles. | Para instalar. |
| [meta-platform-essential-repository](https://github.com/Meta-Platform/meta-platform-essential-repository) | **Runtime e bibliotecas essenciais**: task executor, task loaders, libs comuns; CLIs `repo`/`supervisor`. | Núcleo técnico. |
| [meta-platform-ecosystem-core-repository](https://github.com/Meta-Platform/meta-platform-ecosystem-core-repository) | **Núcleo operacional**: gerenciador de instâncias, serviços e painéis web. | Operação do ecossistema. |
| [meta-platform-package-executor-command-line](https://github.com/Meta-Platform/meta-platform-package-executor-command-line) | **Executor isolado de pacotes** (`pkg-exec`): cria runtime environment e roda um package. | Execução de baixo nível. |
| [meta-platform-cli-script-loader-library](https://github.com/Meta-Platform/meta-platform-cli-script-loader-library) | **Bootstrap / carregador de scripts** das CLIs (prepara ambiente e carrega libs da plataforma). | Suporte às CLIs. |
| [meta-platform-applications-repository](https://github.com/Meta-Platform/meta-platform-applications-repository) | **Aplicações** construídas sobre a plataforma (datasource-manager, api-designer, package-developer, …). | Exemplos de uso final. |

## Por onde começar

1. **Conceitos** → [meta-platform-open-standard](https://github.com/Meta-Platform/meta-platform-open-standard).
2. **Instalar** → [Guia de Início Rápido](./GUIA-INICIO-RAPIDO.md) + [setup-wizard](https://github.com/Meta-Platform/meta-platform-setup-wizard-command-line).
3. **Entender a execução** → [execution-flow](./execution-flow.md) + [package-executor](https://github.com/Meta-Platform/meta-platform-package-executor-command-line).
4. **Criar um pacote** → [Guia: Criar um Pacote](./GUIA-CRIAR-PACOTE.md) + [Guia de Desenvolvimento](./GUIA-DESENVOLVIMENTO.md).

## Como se relacionam

```
open-standard  ──especifica──►  todos os demais
setup-wizard   ──instala────►  repositórios em EcosystemData
essential      ──fornece────►  runtime/libs usados por executor, wizard e core
package-executor ─usa───────►  libs do essential para rodar um package
ecosystem-core ──opera──────►  instâncias, serviços e painéis do ecossistema
applications   ──constrói───►  apps sobre essential + ecosystem-core
cli-script-loader ─bootstrap►  CLIs (wizard e outras) carregam libs da plataforma
```
