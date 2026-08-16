# Guia: WebAssembly na Meta Platform

Este guia é o ponto de entrada para trabalhar com **WebAssembly** na plataforma:
o que existe hoje, como criar um módulo novo, como consumi-lo no servidor e no
navegador, e — a parte que mais economiza tempo — **quando isso vale a pena e
quando não vale**.

> **Especificação normativa:** [WebAssembly Module Manifest Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/packages/wasmlib-manifest-standard.md).
> Este documento é o guia prático; a norma é a referência.

## Índice

- [O que existe](#o-que-existe)
- [Quando usar WebAssembly](#quando-usar-webassembly)
- [Criando um `.wasmlib`](#criando-um-wasmlib)
- [Consumindo no servidor](#consumindo-no-servidor)
- [Consumindo no navegador](#consumindo-no-navegador)
- [O protocolo de memória](#o-protocolo-de-memória)
- [Fidelidade: portar sem mudar o resultado](#fidelidade-portar-sem-mudar-o-resultado)
- [Como medir o ganho](#como-medir-o-ganho)
- [Armadilhas](#armadilhas)
- [Onde olhar](#onde-olhar)

---

## O que existe

| Peça | Onde | O que faz |
|---|---|---|
| Tipo de pacote `.wasmlib` | qualquer repositório | Um módulo WebAssembly com o binário versionado junto. |
| Object loader `wasm-module` | `EssentialRepo/Taskloaders.Module/Loaders.layer/wasm-module.taskLoader` | Compila e instancia o binário; entrega um handle. |
| Manifesto `metadata/wasmlib.json` | dentro do pacote | Declara alias, ABI, binário e exposição web. |
| Módulo de referência | `EssentialRepo/Commons.Module/Libraries.layer/wasm-reference.wasmlib` | Esqueleto mínimo: chamada, memória e laço numérico. |
| Caso real | `EngineeringToolsRepo/EngineeringTools.Module/Viewer.layer/mesh-normalizer.wasmlib` | Normalização de 14,6 M de vértices na ingestão do viewer 3D. |
| Suporte no bundle web | `EcosystemCoreRepo/.../web-interface-builder.lib` | Serve o `.wasm` ao `.webgui` como asset. |

O tipo é reconhecido por todo o ecossistema: `wasmlib` está em
`supportedPackageTypes` do `EssentialRepo`, e a whitelist é derivada da união dos
repositórios instalados.

## Quando usar WebAssembly

WebAssembly resolve **um** problema bem: laço numérico apertado sobre muitos
dados. Fora disso, ele custa mais do que rende.

**Vale a pena quando:**

- há um laço percorrendo centenas de milhares ou milhões de elementos;
- os dados são numéricos e cabem em typed arrays (`Float32Array`, `Uint8Array`…);
- o trabalho é determinístico e não depende de I/O;
- **ou** o problema é portabilidade: um addon nativo (`.node`) exigiria
  `node-gyp` e recompilação por plataforma, e um `.wasm` não exige nada.

**Não vale a pena quando:**

- o trabalho é I/O, orquestração ou manipulação de objetos e strings;
- o volume é pequeno — abaixo de algumas dezenas de milhares de elementos, a
  travessia come o ganho;
- já existe biblioteca nativa fazendo o serviço. `crypto`, `zlib`, `sqlite3` e
  `DuckDB` **já são C**: reescrever em WebAssembly os deixa mais lentos.

### O teto do ganho

A travessia não é grátis. No caso real medido (normalização de malha), o tempo
dentro do módulo se reparte assim:

| Etapa | Fração |
|---|---|
| cálculo em WebAssembly | 58% |
| copiar o resultado de volta para o JavaScript | 31% |
| copiar a entrada para a memória linear | 8% |
| `alloc`/`dealloc` | 2% |

Ou seja: **40% do tempo é fronteira**. Um módulo que fizesse o cálculo em tempo
zero seria só 2,4× mais rápido, não infinitamente. Isso define o que esperar —
ganhos de 1,5× a 3× são o resultado típico; ganhos de 10× indicam que o
JavaScript original fazia trabalho a mais (alocação, cópia, objetos), não que o
WebAssembly seja mágico.

E o ganho local se dilui no total. A normalização ficou **1,84× mais rápida**,
mas ela era só uma fatia da preparação do cache, cujo tempo é dominado pelo
exportador de glTF: **ponta a ponta o ganho ficou em ~7%**. Meça o todo antes de
prometer o número da parte.

### Sobre memória

**Não conte com economia de memória.** No caso real ela ficou empatada (0,90× a
1,10×), e a razão é estrutural: os arrays de saída — que o JavaScript precisa ter
no heap dele — são os mesmos nos dois caminhos, e é neles que o pico mora. O que
o módulo elimina são os intermediários, que o coletor recolheria de qualquer
jeito.

WebAssembly ajuda a memória em outro eixo: a memória linear é **previsível e
controlável**. Você sabe exatamente quanto pediu, e pode descartar uma instância
inteira para devolvê-la (ver [`Instantiate()`](#o-protocolo-de-memória)).

## Criando um `.wasmlib`

### 1. Estrutura

```
<nome>.wasmlib/
├── metadata/
│   ├── package.json    # { "namespace": "@/<nome>.wasmlib" }
│   └── wasmlib.json    # o manifesto
├── dist/
│   └── <nome>.wasm     # o artefato VERSIONADO
├── src-rust/           # a fonte (linguagem livre; o ecossistema não compila)
└── README.md
```

### 2. O manifesto

```json
{
    "alias": "@meu-modulo",
    "abi": "core",
    "binary": "dist/meu-modulo.wasm"
}
```

`abi` é a decisão que mais importa:

- **`core`** — WebAssembly puro. O módulo só enxerga o que você passar no import
  object: sem arquivos, sem relógio, sem rede. **É o padrão recomendado**, e o
  mesmo binário roda no servidor e no navegador.
- **`wasi`** — para código que usa biblioteca padrão (C, C++, Rust em
  `wasm32-wasip1`). Aceita argumentos, variáveis de ambiente e arquivos, mas a
  sandbox é fechada por padrão: sem `preopens` declarados o módulo não vê
  diretório nenhum.

### 3. A fonte, em Rust

```rust
use std::alloc::{alloc as raw_alloc, dealloc as raw_dealloc, Layout};

const ALIGNMENT: usize = 8;

#[no_mangle]
pub extern "C" fn alloc(size: usize) -> *mut u8 {
    if size == 0 { return std::ptr::null_mut() }
    unsafe { raw_alloc(Layout::from_size_align(size, ALIGNMENT).unwrap()) }
}

#[no_mangle]
pub unsafe extern "C" fn dealloc(ptr: *mut u8, size: usize) {
    if ptr.is_null() || size == 0 { return }
    unsafe { raw_dealloc(ptr, Layout::from_size_align(size, ALIGNMENT).unwrap()) }
}

#[no_mangle]
pub unsafe extern "C" fn somar(ptr: *const f32, len: usize) -> f64 {
    let valores = unsafe { std::slice::from_raw_parts(ptr, len) };
    valores.iter().fold(0.0_f64, |total, v| total + *v as f64)
}
```

`Cargo.toml`:

```toml
[lib]
crate-type = ["cdylib"]

[profile.release]
opt-level = 3      # ou "z", se o tamanho do binário importar mais que a velocidade
lto = true
codegen-units = 1
panic = "abort"    # WebAssembly não tem exceções; o código de unwind só engorda
strip = true
```

### 4. Compilar e versionar

```bash
cd src-rust
cargo build --release --target wasm32-unknown-unknown
cp target/wasm32-unknown-unknown/release/meu_modulo.wasm ../dist/meu-modulo.wasm
```

O `.wasm` **entra no repositório**. Quem instala o pacote não precisa de Rust —
é essa escolha que separa o `.wasmlib` do `.nativelib`, cuja compilação na
instalação é o seu ponto frágil. Adicione `src-rust/target` ao `.gitignore`.

> **Confirme que o binário foi mesmo versionado.** Muitos repositórios ignoram
> `dist/` como saída de build, e o binário de um `.wasmlib` é o oposto disso.
> A regra o engole em silêncio: o pacote entra no repositório sem o módulo e
> falha só em runtime, na ativação da task, com "binário não encontrado".
> Verifique com `git ls-files <pacote>/dist/` e, se preciso, abra a exceção:
>
> ```gitignore
> dist/
> !**/*.wasmlib/dist/
> !**/*.wasmlib/dist/**
> ```

## Consumindo no servidor

Declare no `metadata/boot.json` do pacote consumidor, como qualquer dependência:

```json
{
    "bound-params": {
        "meuModulo": "@/meu-modulo.wasmlib"
    }
}
```

Um `.webservice` precisa repassar até o controller, no `endpoint-group.json`:

```json
{
    "bound-params": ["meuModulo"],
    "endpoints": [{
        "bound-params": {
            "controller-params": { "meuModulo": "meuModulo" }
        }
    }]
}
```

E o código recebe o handle pronto:

```ts
const { somar, alloc, dealloc } = meuModulo.getExports()
const memory = meuModulo.getMemory()
```

**Sempre trate o handle como opcional.** Um ambiente antigo, um teste isolado ou
uma instalação parcial podem não tê-lo, e o caminho principal não pode morrer por
causa de um pacote opcional:

```ts
const resultado = meuModulo
    ? PelaWasm(dados)
    : PeloCaminhoAntigo(dados)
```

## Consumindo no navegador

Declare `web.expose` no manifesto e mapeie o alias no `endpoint-group.json` do
`.webgui`:

```json
{
    "bound-params": {
        "wasmModules": { "@meu-modulo": "meuModulo" }
    }
}
```

O bundle recebe a **URL** do artefato — não os exports:

```typescript
import wasmUrl from "@meu-modulo"

const { instance } = await WebAssembly.instantiateStreaming(fetch(wasmUrl), {})
```

O alias aponta para um arquivo, então em TypeScript é preciso declarar o módulo:

```typescript
declare module "@meu-modulo" {
    const url: string
    export default url
}
```

É o **mesmo binário** do servidor. Não há uma build para cada lado.

## O protocolo de memória

WebAssembly não recebe objetos — só números. Passar um array significa reservar
espaço na memória linear, escrever nele e passar o deslocamento:

```ts
const bytes = valores.length * Float32Array.BYTES_PER_ELEMENT
const ptr = exports.alloc(bytes)

// A view é criada DEPOIS do alloc e não é guardada.
new Float32Array(memory.buffer, ptr, valores.length).set(valores)

const resultado = exports.somar(ptr, valores.length)
exports.dealloc(ptr, bytes)
```

> **A view não se guarda.** `memory.buffer` é trocado sempre que a memória linear
> cresce, e uma view criada antes disso passa a apontar para um `ArrayBuffer`
> descolado (*detached*): a escrita se perde **em silêncio**. Crie a view depois
> de toda alocação, use e descarte.

Para ler o resultado de volta ao heap do JavaScript, use `.slice()` — ele copia,
e a cópia sobrevive ao `dealloc`.

### `Instantiate()`: uma instância por lote

Compilar é caro; instanciar é barato. O `WebAssembly.Module` é imutável e sem
estado, e o handle o expõe:

```ts
const lote = meuModulo.Instantiate()
ProcessarLotePesado(lote.exports, lote.memory)
// soltar `lote` devolve a memória linear dele
```

Isso existe porque **`WebAssembly.Memory` cresce e não devolve páginas**. Uma
instância única para o processo inteiro sobe até o pico do maior lote e fica lá.
Uma instância por lote morre com o lote.

### Um bloco de saída, não cinco

Quando a função produz vários arrays, devolva-os num **bloco único** com layout
fixo, e ponha todo campo de 4 bytes antes dos de 1 byte:

```text
[0        ] positions      n * 3 * f32
[n*12     ] normals        n * 3 * f32
[n*24     ] ids            n * f32
[n*28     ] bounds         6 * f32
[n*28+24  ] colors         n * 3 * u8
```

Cada alocação a mais é uma ida ao alocador **por item processado** — com 38 mil
itens isso aparece. E o alinhamento importa: um campo de 4 bytes em deslocamento
ímpar obriga o JavaScript a copiar byte a byte em vez de criar uma view.

## Fidelidade: portar sem mudar o resultado

Ao portar código existente, o resultado tem de ser o **mesmo** — não "parecido".
Três regras, todas aprendidas no port da normalização de malha:

1. **Contas intermediárias em `f64`, resultado final em `f32`.** É o que o
   JavaScript fazia sem querer: todo número dele é `f64`, e a redução para `f32`
   só acontece ao escrever no typed array. Fazer a conta inteira em `f32` dá
   outro número.

2. **Números que atravessam como `f64` continuam `f64`.** Um `Matrix4.elements`
   do `three` é array de números do JavaScript. Reduzi-lo a `f32` na travessia
   introduz erro **antes** da conta.

3. **Copie a ordem das operações, não a "melhore".** Soma de ponto flutuante não
   é associativa: reordenar termos muda o último bit.

Otimizações que **não** mudam o resultado são bem-vindas. Exemplo: toda matriz de
mundo é afim (última linha `0 0 0 1`), então o `w` da divisão perspectiva vale
exatamente `1.0` — pular a divisão é bit a bit o mesmo número e tira uma divisão
de ponto flutuante de cada vértice.

E **verifique com teste**, não por inspeção: rode os dois caminhos sobre as
mesmas entradas e compare as saídas. O teste mais forte compara o **artefato
final** — no caso do viewer, o `.glb` sai com o mesmo `sha256` pelos dois
caminhos.

## Como medir o ganho

### Meça cada caminho em processo separado

Medir os dois no mesmo processo produz um número de memória **falso**: o RSS não
volta ao sistema operacional quando o V8 libera memória, então o segundo caminho
parte de um processo já crescido e "não aloca nada". Na primeira versão do
benchmark do viewer isso apareceu como **41× a favor de quem fosse medido por
último** — que é medição de ordem, não de consumo.

### Aqueça antes de medir

As primeiras centenas de iterações rodam no interpretador do V8 e no código
não otimizado do WebAssembly. Medi-las junto premia quem for medido depois.

### Reproduza o ciclo de vida real dos objetos

Se o código real acumula resultados antes de liberá-los, o benchmark tem de
acumular também. Descartar cada item na hora esconde exatamente o pico que
interessa.

### Escreva testes que falham

Um benchmark que só imprime número não protege nada: a regressão entra e ninguém
vê. Transforme os ganhos em asserções, com limiares **frouxos** — um teste que
falha por barulho de máquina é um teste que alguém desliga. O que se protege é a
direção do resultado, não o valor exato.

Três asserções valem sempre:

- o caminho novo é mais rápido que o antigo;
- a memória linear **estabiliza** — se ela cresce a cada item, falta um
  `dealloc`, e o processo morre no meio de uma entrada grande sem erro de
  JavaScript;
- nada fica retido no heap do V8 depois de liberar os resultados.

## Armadilhas

| Sintoma | Causa |
|---|---|
| Escrita na memória some sem erro | View criada antes de um `alloc` que fez a memória crescer (`ArrayBuffer` descolado). |
| `incompatible import type` na instanciação | O manifesto declara `memory`, mas o módulo **exporta** a própria (é o caso do Rust em `wasm32-unknown-unknown`). Remova a seção. |
| `MODULE_NOT_FOUND` ao carregar o loader | `require` de dependência npm feito tarde demais. Builtins como `node:wasi` são seguros; pacotes npm não. |
| Módulo WASI sem `_initialize` nem imports | O código não toca a biblioteca padrão — o compilador não gerou a ABI. Use algo de `std` para exercitá-la. |
| `.wasm` vira JavaScript no bundle | Falta a regra `asset/resource` para a extensão, ou o suporte nativo do webpack a WebAssembly assumiu o controle. |
| Bundle serve o `.wasm` antigo | A assinatura do cache de build precisa ler o binário por **conteúdo**: recompilar troca os bytes sem mudar data nem tamanho. |
| Processo morre em entrada grande, sem stack | Estouro da memória linear, não do heap do V8. Procure `alloc` sem `dealloc`. |
| "binário não encontrado" só na instalação | O `.gitignore` do repositório engoliu `dist/`. O binário não está versionado. |

## Onde olhar

**Para copiar um esqueleto mínimo:**
`EssentialRepo/Commons.Module/Libraries.layer/wasm-reference.wasmlib` — chamada
simples, protocolo de memória e laço numérico, em ~100 linhas de Rust.

**Para ver um port real, com equivalência e medição:**
`EngineeringToolsRepo/EngineeringTools.Module/Viewer.layer/mesh-normalizer.wasmlib`
e, do lado do JavaScript,
`3d-viewer.webservice/src/Utils/CreateMeshNormalizer.ts` (a ponte),
`test/MeshNormalizerEquivalence.test.js` (equivalência byte a byte),
`test/MeshNormalizerPerformance.test.js` (ganhos como asserção) e
`tools/benchmark-mesh-normalizer.js` (medição em processos separados).

**Para o contrato do loader:**
`EssentialRepo/Taskloaders.Module/Loaders.layer/wasm-module.taskLoader/README.md`.

---

> [Guia: Criar um Pacote](./GUIA-CRIAR-PACOTE.md) ·
> [Guia de Desenvolvimento](./GUIA-DESENVOLVIMENTO.md) ·
> [Arquitetura](./ARQUITETURA.md)
