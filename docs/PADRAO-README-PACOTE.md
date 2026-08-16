# Padrão de README de Pacote

Todo pacote da Meta Platform tem um `README.md`. Ele não é uma nota interna: é
publicado na seção **Referência Repositórios** do site de documentação e exibido
na aba README do **Package Developer**. Como `metadata/package.json` carrega
apenas o `namespace`, o README é hoje a **única descrição** que um pacote tem.

Este documento especifica o formato. Ele vale para os pacotes dos repositórios
oficiais e para qualquer pacote novo criado com `mypkg create`.

## Esqueleto obrigatório

Nesta ordem:

```markdown
# <nome-exato-do-diretório-do-pacote>

- **Tipo:** <rótulo em prosa> (`.<ext>`)
- **Namespace:** `@/<nome>.<ext>`
- **Executável:** `<exec>`
- **Localização:** `<Module>/<layer>/[<group>/]<pkg>.<ext>` (<NamespaceRepo>)

## Propósito

<1 a 3 parágrafos: o que o pacote faz e por que ele existe>

## <Superfície pública — a seção varia por tipo, ver adiante>

## Execução

## Dependências (`metadata/boot.json` → `bound-params`)

<seções livres, à vontade>

> Veja o [README do repositório](../../../README.md).
```

### Regras do cabeçalho

- O **H1 é a primeira linha** do arquivo e repete o **nome exato do diretório**,
  com o sufixo de tipo. Nada de nome em prosa (`# Task Executor`) e nada de
  breadcrumb antes dele — no site o link relativo perde o `href` e o breadcrumb
  vira um parágrafo órfão no topo do artigo.
- Há **um único H1** por arquivo.
- O bloco de identidade é uma **lista de bullets, um por linha**. A variante de
  uma linha só, com `·` separando os campos, está descontinuada.
- **Executável** só aparece nos pacotes publicados em
  `metadata/applications.json` do repositório (é o nome que vai para o `PATH`).
- Em **Localização**, o repositório é identificado pelo `namespace` declarado no
  seu `metadata/repository.json` — `EssentialRepo`, `EcosystemCoreRepo`,
  `PlatformApplicationsRepo` —, não pelo nome do diretório clonado.
- O arquivo termina com **newline final**, em **LF** (nunca CRLF).

### Rótulos de tipo

| Sufixo | Rótulo a usar em **Tipo** |
|---|---|
| `.lib` | biblioteca |
| `.cli` | aplicação de linha de comando |
| `.service` | pacote de serviços |
| `.webservice` | serviço web, backend HTTP |
| `.webgui` | interface web |
| `.webapp` | composição web |
| `.desktopapp` | aplicação desktop Electron |
| `.taskLoader` | *task loader* |
| `.nativelib` | biblioteca nativa |
| `.wasmlib` | módulo WebAssembly |

## Superfície pública, por tipo de pacote

Cada tipo declara sua superfície pública num arquivo de metadado diferente. A
seção do README **espelha esse arquivo** — e cita qual é, entre crases, no
próprio título.

### `.wasmlib`

```markdown
## Manifesto (`metadata/wasmlib.json`)

| Campo | Valor |
|---|---|
| `alias` | `@wasm-reference` |
| `abi` | `core` |

## Exports do módulo

| Export | Assinatura | Responsabilidade |
|---|---|---|
| `sum_f32` | `(ptr, usize) -> f64` | Soma `len` floats a partir de `ptr`. |
```

A tabela de exports espelha o que o binário realmente exporta — o que
`WebAssembly.Module.exports()` devolve, e não o que a fonte pretendia exportar.
A seção **Build** é obrigatória neste tipo: o `.wasm` é versionado, então o
README é o único lugar que diz como reproduzi-lo.

### `.lib`

```markdown
## Exports (`src/`)

| Módulo | Responsabilidade |
|---|---|
| `RegisterLaunch.ts` | Registra uma instância recém-lançada. |
```

Um pacote `.lib` que também exponha serviços acrescenta a seção de `.service`.

### `.cli`

```markdown
## Comandos (`metadata/command-group.json`)

| Comando | Descrição |
|---|---|
| `executor package [path]` | Executa um pacote. |
```

A coluna **Descrição** copia o campo `commands[].description`. Se as duas
divergirem, o `command-group.json` é a verdade — o README é que está velho.

### `.service`

```markdown
## Serviços expostos (`metadata/services.json`)

| Namespace | Path | Dependências (bound-params) |
|---|---|---|
| `GitStatusManager` | `Services/GitStatusManager.service` | — |
```

Um serviço só recebe os `params` declarados no **seu** `services.json`; liste-os
logo abaixo da tabela quando existirem.

### `.webservice`

```markdown
## Serviços disponibilizados

### **Desktop Applications** [DesktopApplications]

Base: `/desktop-applications`

#### **List Desktop Applications** [ListDesktopApplications]

`GET` /desktop-applications/list

**Parâmetros**

| Name | Value Type | Parameter Type | Required |
|---|---|---|---|
```

O par **`**Nome legível** [IdentificadorPascalCase]`** espelha literalmente o
`summary` e o `name` dos `APIs/*.api.json`. Os controllers vêm de
`metadata/endpoint-group.json`.

### `.webgui`

```markdown
## Estrutura (`src/`)

## Boot (`metadata/boot.json`)
```

### `.webapp`

```markdown
## Composição (`metadata/boot.json`)
```

Diga qual `.webgui` e qual `.webservice` o pacote sobe, e sobre qual
`@@/server-service` — é isso que um `.webapp` é.

### `.desktopapp`

```markdown
## Janelas (`metadata/boot.json` → `windows`)
```

Para cada janela: `title`, dimensões, o `dependency` que ela carrega e, se for o
caso, o modo **GUI-host** (`gui-host.serviceGraph`).

### `.taskLoader`

```markdown
## Registro (`metadata/taskloaders.json` do repositório)
```

O `objectLoaderType`, o `entry` e as `npmDependencies` ficam no
`taskloaders.json` **do repositório**, não no pacote. O H1 e o namespace usam
`.taskLoader` — os loaders que ainda dizem `.lib` são resíduo da migração.

### `.nativelib`

Ficha reduzida: sem **Namespace** e sem **Localização** (esses pacotes não têm
`metadata/`), mais uma seção `## Build` com a toolchain (`node-gyp`/`binding.gyp`
ou Cargo/`build.rs`).

## Execução

Obrigatória em `.webgui`, `.webservice` e `.webapp` — pacotes que **não rodam
sozinhos**. Diga explicitamente quem os sobe. Exemplo:

> Não é executada de forma independente: é compilada em runtime pelo loader
> `web-graphic-user-interface`.

Nos pacotes executáveis (`.cli`, `.desktopapp`), a seção descreve como rodar.

## Dependências

```markdown
## Dependências (`metadata/boot.json` → `bound-params`)

- `@/command-executor.lib` (`commandExecutorLib`)
- `@/task-table-render.lib` (`taskTableRenderLib`)
```

Dependências entre pacotes são sempre **por namespace**, nunca por caminho
relativo — no README também.

## Seções livres

Abaixo das seções obrigatórias, escreva o que o pacote precisar: decisões de
design, ciclo de vida, exemplos de uso, tabelas de estado, o que for. A ficha é
o mínimo, não o teto — pacotes centrais merecem documentação de verdade.

Ao padronizar um README existente, **a prosa boa é preservada**: acrescente a
ficha no topo e mantenha o resto.

## Restrições de compatibilidade

O gerador do site converte o Markdown com um parser próprio, mais restrito que o
do GitHub. Escreva dentro deste subconjunto:

| Recurso | Situação |
|---|---|
| Headings `#` a `######` | OK (só ATX; nada de `===` embaixo do texto) |
| Tabelas pipe | OK — a linha separadora `\|---\|` é obrigatória |
| Listas | OK, mas **planas**: aninhamento é perdido |
| Blockquote | OK, mas as linhas são concatenadas — use **uma linha só** |
| Code fences | OK — declare a linguagem |
| Links `http(s)://` | OK |
| Links relativos | O texto sobrevive, **o href é descartado** |
| Imagens e badges | **Removidas** — sobra o texto alternativo |
| HTML embutido | **Escapado** — aparece como texto na tela |
| Task lists, footnotes, front-matter | Não suportados |

Consequência prática: nenhuma informação essencial pode viver só dentro de um
link relativo, e o footer canônico do README é um blockquote de uma linha.

## Checklist

- [ ] H1 na primeira linha, igual ao nome do diretório, e único no arquivo.
- [ ] Bloco de identidade completo, um bullet por linha.
- [ ] `Namespace` idêntico ao de `metadata/package.json`.
- [ ] `## Propósito` responde o que faz **e** por que existe.
- [ ] Seção de superfície pública correta para o tipo, coerente com o metadado.
- [ ] `## Execução` presente nos pacotes que não rodam sozinhos.
- [ ] `## Dependências` lista os `bound-params` por namespace.
- [ ] Sem badges, imagens, HTML ou listas aninhadas.
- [ ] Arquivo em LF, com newline final.

## Referências

- [Guia de Criação de Pacotes](./GUIA-CRIAR-PACOTE.md)
- [Guia de Desenvolvimento](./GUIA-DESENVOLVIMENTO.md)
- [Arquitetura e Organização](./ARQUITETURA.md)
