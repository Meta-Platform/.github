# Errata

Divergências conhecidas entre documentação e comportamento real que **dependem de
alteração de código, build ou decisão de produto** — não de reescrita de texto.

Ao resolver um item no código, remova-o daqui.

| # | Pendência | Onde | O que fazer |
|---|-----------|------|-------------|
| 1 | O Setup Wizard não declara `engines` no `package.json`, então a versão de Node para desenvolvimento não está fixada. | `meta-platform-setup-wizard-command-line/package.json` | Declarar `engines` alinhado ao piso da plataforma (Node ≥ 22.18) e ao target do `pkg`. |

## Nota

A política de **fonte canônica do `.proto`** e a sincronização das cópias IDL estão
em [Package Executor RPC Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/package-executor-rpc-standard.md).
