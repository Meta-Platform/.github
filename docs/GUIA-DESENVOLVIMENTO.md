# Guia de Desenvolvimento

Este guia define os **padrões de desenvolvimento** do Meta Platform: estilo de
código, convenções de nomes e organização de arquivos. O objetivo é que qualquer
pessoa consiga criar e evoluir pacotes de forma **padronizada e previsível**.

> Os padrões aqui descritos foram extraídos do código real dos repositórios
> oficiais (`essential-repository`, `ecosystem-core-repository`) e da ferramenta
> de scaffolding [`package-toolkit.lib`](https://github.com/Meta-Platform/meta-platform-ecosystem-core-repository/blob/main/Main.Module/Libraries.layer/package-toolkit.lib/README.md),
> que **gera** novos pacotes — ou seja, ela é a referência executável deste guia.

Leituras relacionadas:
- [Arquitetura e Organização](./ARQUITETURA.md) — a hierarquia e o ecossistema.
- [Guia: Criar um Pacote](./GUIA-CRIAR-PACOTE.md) — o passo a passo.
- [Referência de Comandos](./REFERENCIA-COMANDOS.md) — CLIs do dia a dia.
- [Open Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/README.md) — as definições formais.

---

## 1. Princípios

1. **Atomização** — cada arquivo tem **uma única responsabilidade** e exporta
   **um único** valor (`module.exports = X`).
2. **Autodescrição** — todo pacote se descreve pela pasta
   [`metadata/`](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/metadata.md);
   o código nunca precisa ser lido para a plataforma entender o pacote.
3. **Acoplamento por namespace** — pacotes dependem uns dos outros **por
   namespace** (`@/...`), nunca por caminho relativo no sistema de arquivos.
4. **Padrão pela ferramenta** — quando possível, crie pacotes com
   `mypkg create ...`; o esqueleto gerado já segue este guia.

---

## 2. Convenções de nomes

### 2.1 Pacotes

- Nome do pacote em **kebab-case**, sempre com o **sufixo do tipo**:
  `repository-manager.cli`, `print-data-log.lib`, `server-manager.service`.
- Sufixos válidos: `.lib`, `.cli`, `.service`, `.webservice`, `.webgui`,
  `.webapp`, `.desktopapp`, `.app` (ver
  [Package](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/package.md)).
  Há ainda o sufixo **experimental** `.nativelib` (bibliotecas nativas), que a
  descoberta de pacotes da plataforma ainda **não** reconhece (fora de
  `REPOS_CONF_EXTLIST_PKG_TYPE` no `ecosystem-defaults.json`).
- O **namespace** (em `metadata/package.json`) é `@/<nome-do-pacote>` — ex.:
  `@/repository-manager.cli`.

### 2.2 Itens da hierarquia

| Item | Convenção | Exemplo |
|------|-----------|---------|
| Module | `PascalCase` + `.Module` | `Commons.Module`, `Runtime.Module` |
| Layer | `PascalCase` + `.layer` | `Libraries.layer`, `Application.layer` |
| Group | `PascalCase` + `.group` | `APIDesigner.group` |

### 2.3 Arquivos e pastas em `src/`

- **Módulos** (`.js`): **`PascalCase`**, e o nome do arquivo é **igual** ao nome
  da função/valor exportado. Ex.: `ResolvePackageName.js` exporta
  `ResolvePackageName`.
- **Pastas** em `src/`: `PascalCase`, geralmente no plural por papel —
  `Commands/`, `Services/`, `Managers/`, `Helpers/`, `Utils/`, `Configs/`.

#### Sufixos de arquivo por papel

| Sufixo | Usado para | Pasta | Exporta |
|--------|------------|-------|---------|
| `*.command.js` | handler de um comando de CLI | `Commands/` | `XCommand` |
| `*.service.js` | um serviço | `Services/` | `XService` |
| `*.taskLoader.js` | um *object loader* (task loader) | (raiz de `src/`) | `XTaskLoader`/`X` |
| `*.manager.js` | um *manager* (orquestrador) | `Managers/` | `XManager`/`X` |

> Exemplos reais: `ListSources.command.js`, `HTTPServer.service.js`,
> `ApplicationInstance.taskLoader.js`, `InstanceMonitoring.manager.js`.

### 2.4 Variáveis e constantes

- **Funções e módulos exportados:** `PascalCase`.
- **Variáveis e parâmetros locais:** `camelCase`.
- **Constantes de nível de módulo:** `UPPER_SNAKE_CASE`
  (ex.: `const EXT_TYPE = "lib"`, `const FILE_EXT = "js"`).
- **`bound-params`** (dependências injetadas): `camelCase` com sufixo que indica o
  tipo — `...Lib` para bibliotecas e `...Service` para serviços
  (ex.: `jsonFileUtilitiesLib`, `printDataLogLib`, `serverService`).

---

## 3. Estilo de código (JavaScript)

A base é **Node.js + CommonJS** (`require`/`module.exports`), sem TypeScript no
back-end (as `.webgui` usam React/TSX). As regras abaixo refletem o padrão
predominante no código.

### 3.1 Estrutura de um módulo

```javascript
// 1) imports de libs externas e builtins (builtins com prefixo node: quando aplicável)
const { resolve } = require("path")
const { mkdir }   = require("node:fs/promises")

// 2) imports internos do próprio pacote
const CreateUtf8TextFile = require("./Utils/CreateUtf8TextFile")

// 3) constantes de módulo (UPPER_SNAKE_CASE)
const FILE_EXT = "js"

// 4) a função/valor único do arquivo (PascalCase, == nome do arquivo)
const CreateThing = async ({ packagePath, functionName }) => {
    // ...
}

// 5) export único, no fim do arquivo
module.exports = CreateThing
```

### 3.2 Regras

- **Arrow functions** com `const` para definir funções; evite `function`.
- **Um único parâmetro objeto desestruturado** quando houver mais de um argumento:
  `const F = ({ a, b }) => ...` em vez de `const F = (a, b) => ...`.
- **`async`/`await`** para fluxo assíncrono (não use callbacks aninhados nem
  `.then()` encadeado quando `await` resolver).
- **Aspas duplas** para strings (`"texto"`); aspas simples são toleradas para
  caracteres e nomes de builtin (`'node:fs/promises'`).
- **Sem ponto e vírgula** ao final das instruções (padrão predominante).
- **Um export por arquivo** (`module.exports = X`), sempre na última linha.
- Mensagens ao usuário em **português** (logs, descrições, erros visíveis).

### 3.3 Indentação

> **Estado atual:** o código JS está **misturado** — há arquivos com **tabs** e
> arquivos com **4 espaços**. Para código novo, **use 4 espaços** (é o que o
> `package-toolkit.lib` gera). **Não misture** tabs e espaços no mesmo arquivo.
>
> Os **arquivos de metadados JSON** são gerados com **tab** (`WriteObjectToFile`
> usa `'\t'`); mantenha tab nos `*.json` de `metadata/` para ficar consistente
> com o que a ferramenta produz.

Um `.editorconfig` ajuda a padronizar entre editores. Sugestão (ainda não
existe no repositório):

```ini
root = true

[*.js]
indent_style = space
indent_size = 4
charset = utf-8
insert_final_newline = true
trim_trailing_whitespace = true

[metadata/*.json]
indent_style = tab
```

---

## 4. Padrões por tipo de pacote

### 4.1 Biblioteca (`.lib`)

- Apenas `metadata/package.json` (namespace) + módulos em `src/`.
- Cada módulo é exportado individualmente e consumido por outros pacotes via
  `lib.require("NomeDoModulo")`:

```javascript
// quem recebe `jsonFileUtilitiesLib` como bound-param:
const ReadJsonFile = jsonFileUtilitiesLib.require("ReadJsonFile")
```

### 4.2 Comando de CLI (`*.command.js`)

Handler assíncrono que recebe `{ args, startupParams, params }`. `params` traz os
`bound-params` declarados em `parametersToLoad`:

```javascript
const ListSourcesCommand = async ({ startupParams, params }) => {
    const { jsonFileUtilitiesLib } = params
    const { installDataDirPath } = startupParams

    const ReadJsonFile = jsonFileUtilitiesLib.require("ReadJsonFile")
    // ...
}

module.exports = ListSourcesCommand
```

Esqueleto gerado por `mypkg create commandline <nome>`:

```javascript
const XCommand = async ({ args, startupParams, params }) => {

}

module.exports = XCommand
```

### 4.3 Serviço (`*.service.js`)

Função que recebe `params` (incluindo callbacks de ciclo de vida como `onReady`/
`onClose`) e **retorna um objeto** com a API pública do serviço:

```javascript
const HTTPServerService = (params) => {
    const { name, port, onReady, middlewareService } = params
    // ... inicialização ...
    onReady()
    return { /* métodos do serviço */ }
}

module.exports = HTTPServerService
```

Esqueleto gerado por `mypkg create services <nome>`:

```javascript
const XService = (params) => {
    const {
        // params e bound-params...
        onReady
    } = params

    onReady()

    return {}
}

module.exports = XService
```

### 4.4 Task loader (`*.taskLoader.js`)

Implementa um tipo de *object loader* consumido pelo *task executor* (ver
[Tipos de Object Loader](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/tipos-de-object-loader.md)).
Fica na raiz de `src/` e seu nome reflete o tipo (`ApplicationInstance.taskLoader.js`
→ `application-instance`).

### 4.5 Aplicação desktop (`.desktopapp`)

Aplicação desktop nativa: abre uma ou mais **janelas Electron**. No **modo
primário**, o `.desktopapp` **combina a composição de um webapp** — os mesmos
`services` e `endpoints` que sobem um backend HTTP local e o webgui (compilado em
runtime) — com uma seção `windows`. A janela faz
`BrowserWindow.loadURL(http://localhost:{port}/)`, apontando para esse servidor
local. No **modo alternativo**, a janela carrega HTML estático do disco via
`BrowserWindow.loadFile` (sem servidor HTTP), para conteúdo autossuficiente.

O `boot.json` combina, então, a composição do webapp (services/endpoints) com uma
seção `windows` (array). Cada janela aceita:

| Campo | Obrigatório | Descrição |
|-------|-------------|-----------|
| `url` | sim (modo `loadURL`) | URL do servidor local a carregar. Usa template de **valor inteiro** (`"url": "{{windowUrl}}"`, onde `windowUrl` é uma startup-param, ex.: `http://localhost:8083/`). Interpolação embutida na string **não** é suportada. |
| `bound-params` | modo `loadURL` | Dependências resolvidas por namespace; use `{ "serverService": "@@/server-service" }` para a janela esperar o servidor. |
| `file` | sim (modo `loadFile`) | Caminho do HTML estático, relativo à raiz do package que fornece o conteúdo. |
| `dependency` | não (modo `loadFile`) | Namespace do package que fornece o `file` (ex.: `@/api-designer.webgui`). |
| `title` | não | Título da janela. |
| `width` / `height` | não | Dimensões da janela. |

Exemplo real: o `api-designer.desktopapp` roda a **mesma aplicação** do
`api-designer.webapp` (mesmos services/endpoints) dentro de uma janela Electron, na
porta `8083` (o webapp usa `8082`). O webgui **não** tem `dist/` estático — é
buildado em runtime e depende do backend HTTP; por isso `loadFile` puro não serve
para ele.

```json
{
    "params": [],
    "windows": [
        {
            "title": "API Designer",
            "url": "{{windowUrl}}",
            "bound-params": {
                "serverService": "@@/server-service"
            },
            "width": 1280,
            "height": 800
        }
    ]
}
```

Cada janela é instanciada pelo object loader `desktop-window-instance`
(implementado pela `desktop-window-instance.lib`, em `EssentialTaskLoaders.layer`).
No modo `loadURL`, a janela **espera o serviço `@@/server-service` ficar `ACTIVE`**
(via os `agentLinkRules` gerados a partir do `bound-param` `serverService`) e
reintenta o load enquanto o webgui compila. No modo `loadFile`, a janela abre o
HTML estático diretamente. O Electron é dependência declarada da
`desktop-window-instance.lib` (no `package.json`) e instalado em runtime como
qualquer outra dependência de package. No `applications.json` o executável
correspondente usa `appType: "DESKTOP"`.

---

## 5. Metadados (resumo prático)

Arquivos em `metadata/` (JSON com indentação **tab**). Detalhes em
[Metadata](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/metadata.md).

| Arquivo | Quando | Conteúdo |
|---------|--------|----------|
| `package.json` | sempre | `{ "namespace": "@/<pacote>" }` |
| `boot.json` | executáveis | `executables` + `bound-params` |
| `command-group.json` | `.cli` | árvore de comandos (`path: "Commands/<Nome>.command"`) |
| `services.json` | `.service` | serviços (`path: "Services/<Nome>.service"`) |
| `startup-params.json` | opcional | valores de inicialização |

O `package.json` da **raiz** do pacote (identidade NPM) segue o padrão gerado:

```json
{
    "name": "meu-pacote.lib",
    "author": "Seu Nome <voce@exemplo.com>",
    "version": "0.0.1",
    "license": "BSD-3-Clause"
}
```

E um `.gitignore` com pelo menos `node_modules`.

---

## 6. Dependências

- **Entre pacotes:** sempre por **namespace** (`@/...`) declarado em
  `bound-params` — nunca `require("../../../outro-pacote")`.
  > Há um item de planejamento para melhorar as dependências internas e o
  > `SmartRequire`.
- **NPM (de um pacote):** declaradas no `package.json` da raiz do pacote.

---

## 7. Checklist para um novo pacote

- [ ] Pasta com o **sufixo** correto (`.lib`, `.cli`, …) em um `Layer`/`Group` adequado.
- [ ] `metadata/package.json` com `namespace` `@/...`.
- [ ] Módulos em `src/` em `PascalCase`, um export por arquivo.
- [ ] Sufixos de papel corretos (`.command.js`, `.service.js`, …) nas pastas certas.
- [ ] Dependências entre pacotes **por namespace**.
- [ ] Estilo: arrow functions, aspas duplas, sem ponto e vírgula, 4 espaços (JS) / tab (JSON de metadata).
- [ ] `README.md` descrevendo propósito, namespace, exports/comandos e dependências.
- [ ] (Opcional, recomendado) Gerar o esqueleto com `mypkg create ...` e ajustar.

---

## 8. Próximos passos

- [Guia: Criar um Pacote](./GUIA-CRIAR-PACOTE.md)
- [Arquitetura e Organização](./ARQUITETURA.md)
- [Meta Platform Open Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/README.md)
