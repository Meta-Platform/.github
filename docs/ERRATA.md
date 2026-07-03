# Errata e Pendências Conhecidas

Pendências detectadas na auditoria de documentação
que **exigem alteração de código, JSON, metadado, build/release ou decisão de
produto** — portanto **fora do escopo das etapas puramente documentais**. Este
documento apenas as registra; nenhuma delas foi aplicada.

> As correções **documentais** (P0–P3) já foram aplicadas nas etapas anteriores e
> não aparecem aqui.

| # | Pendência | Tipo | Impacto | Arquivo provável | Sugestão de etapa futura |
|---|-----------|------|---------|------------------|--------------------------|
| 1 | `StopEnvironment` tem `description` trocada ("Listar tarefas carregadas no task executor principal") para um comando que **para** um ambiente. | metadado | Baixo–médio: `--help` do `executor stop env` exibe texto incorreto. | `repos/ecosystem-core-repository/Main.Module/Application.layer/instance-executor.cli/metadata/command-group.json` | Corrigir a `description` do nó `StopEnvironment` para algo como "Para um ambiente em execução". |
| 2 | Setup Wizard não declara `engines` no `package.json`. | código/build | Médio: a versão de Node para desenvolvimento não está pinada; doc precisa ser cautelosa. | `Meta-Platform/meta-platform-setup-wizard-command-line/package.json` | Adicionar `engines` alinhado ao target real do `pkg` (decidir Node 18 vs upgrade). |
| 3 | Divergência de nomenclatura do artefato: o script `build` gera `...-0.0.19-alpha-linux-x64`, mas o asset efetivamente **publicado** na release (e usado nas docs) é `...-0.0.19-preview-linux-x64`. | build/release | Médio: uma re-release feita com o script `build` atual quebraria as URLs de download documentadas. | `Meta-Platform/meta-platform-setup-wizard-command-line/package.json` (script `build`) e assets de release | Padronizar `alpha`/`preview` entre o script `build` e os assets de release. |
| 4 | `mywizard` usa `default: 'standard'` em `install`/`update`/`show-profile`, mas `standard` **não** está registrado em `LoadAllInstalationProfiles`. | código | Alto: rodar sem `--profile` falha. Já documentado como known issue. | `Meta-Platform/meta-platform-setup-wizard-command-line/src/Executables/mywizard.js` e `src/Helpers/LoadAllInstalationProfiles.js` | Definir um default válido (ex.: `release-standard`) ou remover o default e exigir `--profile`. |
| 5 | `mywizard show-profile` está quebrado (aponta para arquivos inexistentes). | código | Médio: comando não funciona; já documentado como known issue. | `Meta-Platform/meta-platform-setup-wizard-command-line/src/Executables/mywizard.js` + `Commands/ShowProfileInfo.command` | Corrigir a resolução de perfil do `show-profile`. |
| 6 | Decisão sobre renomear `thrid-party-repos` → `third-party-repos` (typo de *third*). | decisão de produto | Baixo: cosmético; afeta paths reais, exemplos e eventuais scripts. | Diretório `thrid-party-repos/` no workspace e docs que citam o path | Avaliar custo/benefício de renomear o diretório e atualizar todas as referências; hoje documentado como nome legado. |

## Notas

- A política de **fonte canônica do `.proto`** e a **sincronização das cópias IDL**
  estão documentadas em
  [Package Executor RPC Standard](https://github.com/Meta-Platform/meta-platform-open-standard/blob/main/specifications/package-executor-rpc-standard.md)
  (seção "Fonte canônica e cópias de implementação").
- Itens 4 e 5 também constam nos `docs/known-issues.md` e `docs/troubleshooting.md`
  do Setup Wizard.
