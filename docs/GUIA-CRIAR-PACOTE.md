# Guia: Criar um Pacote

Este guia mostra, passo a passo, como criar um novo **pacote** do Meta Platform.
Antes de começar, recomendamos ler a seção
[Anatomia de um pacote](./ARQUITETURA.md#anatomia-de-um-pacote) do documento de
arquitetura.

Vamos criar dois exemplos:

1. Uma **biblioteca** (`.lib`) — a unidade reutilizável mais simples.
2. Uma **CLI** (`.cli`) — uma aplicação de linha de comando que usa essa lib.

---

## Onde criar o pacote

Um pacote sempre vive dentro da hierarquia
`Repository → Module → Layer → (Group →) Package`. Escolha o local pelo tipo:

- Bibliotecas reutilizáveis → uma `*.layer` de bibliotecas
  (ex.: `Commons.Module/Libraries.layer/`).
- Aplicações de linha de comando → `Main.Module/Application.layer/`.
- Aplicações web compostas → um `*.group` dentro de `Application.layer` ou
  `Apps.Module`.

O **nome da pasta** do pacote termina sempre com o sufixo do tipo: `.lib`,
`.cli`, `.service`, `.webservice`, `.webgui`, `.webapp`, `.desktopapp`, `.app`,
`.nativelib`.

---

## Parte 1 — Criar uma biblioteca (`.lib`)

### 1.1 Estrutura

```
saudacao-utilities.lib/
├── metadata/
│   └── package.json
├── src/
│   └── Saudar.js
└── package.json
```

### 1.2 `metadata/package.json` — o namespace

O único metadado obrigatório de uma lib é o seu **namespace**. Use o prefixo
`@/` (referência a um package por namespace no conjunto de repositórios
instalados, resolvida globalmente no `EcosystemData`):

```json
{
    "namespace": "@/saudacao-utilities.lib"
}
```

### 1.3 `package.json` — identidade e dependências Node.js

```json
{
  "name": "saudacao-utilities.lib",
  "version": "0.0.1",
  "author": "Seu Nome <voce@exemplo.com>",
  "license": "BSD-3-Clause"
}
```

### 1.4 `src/Saudar.js` — um módulo da lib

Cada arquivo em `src/` é um módulo exportado individualmente. Mantenha-os
pequenos e com responsabilidade única:

```javascript
const Saudar = (nome) => `Olá, ${nome}!`

module.exports = Saudar
```

Outros pacotes consomem a lib pelo namespace e carregam módulos por nome com
`.require(...)`:

```javascript
// dentro de um pacote que recebeu `saudacaoUtilitiesLib` como bound-param
const Saudar = saudacaoUtilitiesLib.require("Saudar")
console.log(Saudar("Meta Platform"))
```

> **Convenção:** arquivos em `src/` usam `PascalCase`. Comandos de CLI usam o
> sufixo `.command.js`.

---

## Parte 2 — Criar uma CLI (`.cli`)

Uma CLI expõe um executável no terminal e organiza sua lógica em comandos.

### 2.1 Estrutura

```
saudacao-manager.cli/
├── metadata/
│   ├── package.json         # namespace
│   ├── boot.json            # executáveis + dependências (bound-params)
│   ├── command-group.json   # árvore de comandos
│   └── startup-params.json  # parâmetros de inicialização
├── src/
│   ├── Commands/
│   │   └── Saudar.command.js
│   ├── Helpers/
│   └── Configs/
├── package.json
└── README.md
```

### 2.2 `metadata/package.json`

```json
{
    "namespace": "@/saudacao-manager.cli"
}
```

### 2.3 `metadata/boot.json` — o que o pacote expõe

Declara o(s) executável(is) e injeta dependências por namespace via
`bound-params`. O `executableName` é o comando que o usuário digitará no
terminal:

```json
{
    "executables": [
        {
            "dependency": "@//command-group",
            "executableName": "saudar",
            "bound-params": {
                "saudacaoUtilitiesLib": "@/saudacao-utilities.lib",
                "printDataLogLib": "@/print-data-log.lib"
            }
        }
    ]
}
```

- `dependency: "@//command-group"` indica que o executável é construído a partir
  do `command-group.json` deste pacote.
- As chaves de `bound-params` (ex.: `saudacaoUtilitiesLib`) viram os nomes pelos
  quais os comandos recebem as libs; os valores são os namespaces resolvidos no
  ecossistema.

### 2.4 `metadata/command-group.json` — a árvore de comandos

```json
{
    "bound-params": [
        "saudacaoUtilitiesLib",
        "printDataLogLib"
    ],
    "commands": [
        {
            "commandName": "Saudar",
            "path": "Commands/Saudar.command",
            "command": "saudar [nome]",
            "description": "Exibe uma saudação para o nome informado",
            "parameters": [
                {
                    "key": "nome",
                    "paramType": "positional",
                    "valueType": "string",
                    "describe": "nome de quem será saudado"
                }
            ],
            "parametersToLoad": [
                "saudacaoUtilitiesLib"
            ]
        }
    ]
}
```

Campos importantes de cada comando:

| Campo | Função |
|-------|--------|
| `commandName` | Nome interno do comando. |
| `path` | Caminho do handler em `src/` (sem `.js`). |
| `command` | Assinatura no estilo yargs (`install [a] [b]`). |
| `description` | Texto exibido no `--help`. |
| `parameters` | Lista de parâmetros: `paramType` é `positional` ou `option`; `valueType` é `string`, `array`, etc. |
| `parametersToLoad` | Quais `bound-params` são injetados neste comando. |
| `children` | (opcional) Lista de subcomandos, formando uma árvore. |

### 2.5 `metadata/startup-params.json`

Valores de inicialização específicos do pacote (lidos pelos comandos como
`startupParams`):

```json
{
    "installDataDirPath": "~/EcosystemData"
}
```

### 2.6 `src/Commands/Saudar.command.js` — o handler

Um handler é uma função assíncrona que recebe `{ args, startupParams, params }`:

- `args` — argumentos posicionais e *options* informados na CLI (ex.: o `nome`
  declarado no `command-group.json`);
- `startupParams` — valores de inicialização do pacote (de `metadata/startup-params.json`);
- `params` — dependências (libs) injetadas via `parametersToLoad` / `bound-params`.

```javascript
const SaudarCommand = async ({
    args,
    startupParams,
    params
}) => {
    const { saudacaoUtilitiesLib } = params

    const Saudar = saudacaoUtilitiesLib.require("Saudar")

    const { nome } = args
    console.log(Saudar(nome || "mundo"))
}

module.exports = SaudarCommand
```

> `args` pode ser omitido no *destructuring* quando o comando não lê argumentos
> (ex.: `async ({ startupParams, params }) => { … }`). Para comandos **com
> parâmetros**, use a forma completa acima e leia os valores de `args`.

### 2.7 `package.json` e `README.md`

```json
{
  "name": "saudacao-manager.cli",
  "version": "0.0.1",
  "author": "Seu Nome <voce@exemplo.com>",
  "license": "BSD-3-Clause"
}
```

Inclua um `README.md` descrevendo o pacote, seus comandos e exemplos de uso —
todos os pacotes oficiais seguem essa convenção.

---

## Parte 3 — Instalar e testar

Pacotes são distribuídos via repositórios. Durante o desenvolvimento, use uma
fonte `LOCAL_FS` para testar sem publicar nada:

```bash
# registre o repositório que contém seu novo pacote como fonte local
repo register source MeuRepo LOCAL_FS --localPath ~/caminho/para/meu-repo

# instale (opcionalmente, apenas executáveis específicos)
repo install MeuRepo LOCAL_FS --executables "saudar"

# teste o comando (executables precisa estar no PATH)
saudar Meta-Platform

# ao alterar o código, atualize
repo update MeuRepo
```

Para execução isolada de baixo nível (sem instalar no ecossistema), use
`pkg-exec` — veja o
[README do package-executor](https://github.com/Meta-Platform/meta-platform-package-executor-command-line/blob/main/README.md).

---

## Checklist do pacote

- [ ] A pasta tem o **sufixo** correto (`.lib`, `.cli`, …).
- [ ] `metadata/package.json` define o **namespace** com prefixo `@/`.
- [ ] (CLI) `boot.json` declara `executables` e `bound-params`.
- [ ] (CLI) `command-group.json` lista os comandos e seus `parametersToLoad`.
- [ ] Dependências entre pacotes são feitas **por namespace**, nunca por caminho
      relativo ao sistema de arquivos.
- [ ] Há um `README.md` descrevendo o pacote.
- [ ] O pacote está no `Module`/`Layer`/`Group` adequado ao seu tipo.

---

## Referências

- [Arquitetura e Organização](./ARQUITETURA.md)
- [Tipos de Object Loader](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/concepts/tipos-de-object-loader.md)
- [Execution Params Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/packages/execution-params-standard.md)
