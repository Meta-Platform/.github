# Guia de log da Meta Platform

Como registrar log em qualquer pacote da plataforma. A especificação formal
está em
[logging-standard.md](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/logging-standard.md);
aqui é o dia a dia.

## O básico

Não importe nada. Não declare nada no metadado. O `Log` já existe:

```ts
Log.info("UpdateRepository", "Atualizando...")
```

Ele é instalado no bootstrap do processo — pelo `package-executor`, pelo
`package-runner`, pelo wizard ou pelo processo do Electron — e fica disponível
em qualquer arquivo que rode dentro do ecossistema.

## Escolher o nível

A pergunta é **para quem** aquela linha existe.

| Se a linha é… | use | e ela |
|---|---|---|
| o resultado que o usuário pediu (listagem, tabela, "pronto!") | `Log.message` | aparece limpa no terminal, sem carimbo |
| o programa registrando o que fez (subiu, parou, conectou) | `Log.info` | vai só para o arquivo |
| diagnóstico de quem está desenvolvendo | `Log.debug` | fica invisível até você baixar o piso |
| fluxo fino (entrada/saída, payload) | `Log.trace` | idem |
| algo degradou mas seguiu | `Log.warn` | aparece no terminal |
| algo falhou | `Log.error` | aparece no terminal |
| falhou e o processo vai morrer | `Log.fatal` | aparece no terminal |

O erro mais comum é usar `info` para falar com o usuário. Se a pessoa que
digitou o comando precisa ler aquilo, é `message`.

## Os três argumentos

```ts
Log.error("CreateEnvironmentDir", "falhou ao criar o diretório", { path, error })
```

1. **source** — de onde vem: o nome da função, comando ou componente. Não é o
   pacote; esse vem do contexto.
2. **mensagem** — curta, uma linha.
3. **data** (opcional) — o estruturado. É aqui que vai o `Error`: o `stack` é
   preservado, e você pode filtrar por esses campos depois.

Não concatene dados na mensagem quando eles couberem em `data`:

```ts
Log.error("GetIcon", "falha ao obter o ícone", { packageName, workspace })   // filtrável
Log.error("GetIcon", `falha em ${packageName} / ${workspace}`)                // texto morto
```

## Não repetir o source

```ts
const log = Log.source("UpdateRepository")

log.info("Atualizando...")
log.error("falhou", { error })
```

## Carimbar uma execução

Num *object loader*, carimbe uma vez e todo log daquela execução sai
identificado — sem passar nada adiante:

```ts
const log = Log
    .child({ instanceId : process.env.META_LAUNCH_ID, environmentPath })
    .source("ServiceInstance")
```

## Onde o log vai parar

```
EcosystemData/
├── logs/
│   ├── ecosystem/2026-07-27.jsonl        # daemon, CLIs, instalação, wizard
│   ├── applications/<pacote>/…
│   └── instances/<instanceId>.jsonl
└── environments/<pacote>-<hash>/logs/    # o log daquela execução
```

Formato JSONL: uma linha JSON por evento, filtrável com `grep` e `jq`. Rotação
diária, com teto de tamanho e descarte por prazo — tudo configurável nas vars
`LOG_CONF_*` do `ecosystem-defaults.json`.

Para ler pela interface: **Ecosystem Control Panel → Logs**, ou a aba `logs`
dentro de um ambiente.

## No navegador (webgui)

```ts
import BrowserLog from "../Utils/BrowserLog"

BrowserLog.warn("MinhaTela", "o usuário tentou X sem Y")
```

Mesmos sete níveis. Os eventos são acumulados e enviados em lote ao backend,
que os grava com `origin: "browser"`. Erro não tratado da tela entra sozinho.

## O que NÃO usa o `Log`

`Log` é para código que roda **dentro** do ecossistema. Seguem com `console`,
de propósito:

- ferramentas standalone (`scripts/`, `tools/`, `test.ts` avulso) — rodam por
  `node`, fora do ecossistema, e ali `globalThis.Log` não existe;
- código carregado **durante** a construção do próprio logger (os
  `SmartRequire`);
- o servidor MCP, cujo stdout é o canal do protocolo.

Fora esses casos, `console.*` é regressão — e o lint
`maintenance-toolkit.cli/scripts/lint-no-console-log.js` reprova.

## Se você não vê seu log

1. **É `info` e você olha o terminal?** O piso padrão do terminal é `message`.
   Ele está no arquivo.
2. **Nada aparece em arquivo nenhum?** O binário do `package-executor` precisa
   ser posterior à injeção do logger; um binário antigo faz o log existir só no
   terminal.
3. **Rodando fora do ecossistema?** Aí não há `Log` — veja a seção acima.
