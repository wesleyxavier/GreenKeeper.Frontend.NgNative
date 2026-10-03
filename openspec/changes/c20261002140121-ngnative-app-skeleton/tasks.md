# Tasks

> Regras: todo comando Node/Gradle/emulador/Playwright roda via `docker compose` (regra `docker-isolation`); commit, push e PR exigem **autorização explícita do usuário** a cada vez (regra `authorization`); o PR sai por `gh` (remote GitHub), com squash em `develop`. O NG Native é alpha: se um spike crítico falhar, **parar e replanejar** (design.md Decisão 19).

## 0. Bootstrap e Branch

- [x] 0.1 **[AUTORIZAÇÃO] Commit inicial e `develop`** (repo vazio, sem `develop`)
  - **Comando:** `git add openspec .claude && git commit -m "chore: bootstrap openspec" && git push -u origin main && git branch develop && git push -u origin develop`
  - **Contexto:** design.md Decisão 20 — a regra de worktree parte de `develop`.
  - **Done:** `git branch -a` mostra `main`, `develop` e os remotos.

- [x] 0.2 **Criar worktree `feature/ngnative-app-skeleton`** (skill `openspec-worktree`)
  - **Comando:** `git worktree add worktrees/skeleton -b feature/ngnative-app-skeleton develop` (path curto: aapt2/OneDrive)
  - **Done:** `git worktree list` mostra `worktrees/skeleton` na branch `feature/ngnative-app-skeleton`. Todas as tasks seguintes rodam nesse worktree.

---

## 1. Spikes (ponto de replanejamento)

> Código dos spikes é descartável; só as decisões vão para o design.md.

- [x] 1.1 **Spike (a): template NG Native e APK no Docker**
  - **O que fazer:** `create-expo-app --template @ng-native/template` num serviço `node` do compose; `Dockerfile.build` provisório (Node 22.18+, JDK 17, Android cmdline-tools); `expo prebuild --platform android` e `gradlew assembleRelease`.
  - **Contexto:** design.md Decisões 2, 18 e 19.
  - **Done:** APK gerado; registrar versões reais de Angular, `@ng-native/*`, Expo, RN, Node, JDK, `compileSdk` e `build-tools`. **Se falhar, parar.**

- [x] 1.2 **Spike (b): tabs, ícone ⚙ e modal como sheet**
  - **O que fazer:** `native-tabs-outlet` com 4 rotas + botão ⚙ que não navega; `<modal>` com animação como sheet; `HardwareBack` fechando o sheet.
  - **Contexto:** design.md Decisões 4 e 11.
  - **Done:** funciona no emulador/dispositivo; decisão registrada (sheet com ou sem gesto). **Se falhar, parar.**

- [x] 1.3 **Spike (c): NgRx e HttpClient nativo**
  - **O que fazer:** `provideStore`/`provideEffects` + `selectSignal` no app nativo; `provideNativeHttpClient()` com um interceptor.
  - **Contexto:** design.md Decisões 5 e 12.
  - **Done:** Store e interceptor funcionam no Android; decidir NgRx Store, SignalStore ou store próprio.

- [x] 1.4 **Spike (d): Database, migrations e reatividade**
  - **O que fazer:** módulo `Database` com 2 migrations; repositório com signal de listagem e invalidação; testar `addDatabaseChangeListener`.
  - **Contexto:** design.md Decisão 6.
  - **Done:** inserção externa atualiza a lista sem recarga; decisão registrada.

- [x] 1.5 **Spike (e): shell web, islands, service worker e IndexedDB**
  - **O que fazer:** `apps/web` Angular + `ngNativeWeb()`, uma island com um componente nativo, `@angular/service-worker`, IndexedDB.
  - **Contexto:** design.md Decisões 7 e 8.
  - **Done:** island renderiza no Chromium, SW ativo, abertura offline funciona.

- [x] 1.6 **Spike (f): emulador + Maestro com KVM**
  - **O que fazer:** emulador x86_64 headless num container (WSL2 com `/dev/kvm`), instalar o APK do spike (a) e rodar um fluxo Maestro.
  - **Contexto:** design.md Decisão 18.
  - **Done:** Maestro abre o app e tira screenshot; sem KVM, registrar o plano B (runner Linux).

- [x] 1.7 **Commit + push do grupo 1** (descartar o código dos spikes; manter só as decisões em design.md)
  - **Comando:** `git add openspec && git commit -m "docs: registra decisoes dos spikes" && git push -u origin feature/ngnative-app-skeleton`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 2. Fundação do repo

- [ ] 2.1 **Criar `.gitignore`**
  - **Conteúdo:** `node_modules/`, `.nx/`, `.expo/`, `dist/`, `android/`, `ios/`, `.gradle/`, `local.properties`, `artifacts/`, `*.apk`, `.env`, `worktrees/`, `**/graphify-out/cache`, `.DS_Store`, `Thumbs.db`.
  - **Done:** `git status` não lista esses caminhos depois de gerados.

- [ ] 2.2 **Criar `.graphifyignore`**
  - **Conteúdo:** `node_modules`, `.nx`, `android`, `dist`, `.expo`, `artifacts`, `*.apk`, lockfiles e assets binários.
  - **Done:** `graphify update .` ignora esses caminhos.

- [ ] 2.3 **Criar `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`**
  - **O que mudar:** mesmo padrão do repo MAUI, apontando para o README.
  - **Done:** os 3 arquivos existem.

- [ ] 2.4 **Criar `README.md` inicial**
  - **Conteúdo:** visão, stack, aviso de alpha, estrutura de pastas, comandos Docker (placeholders), seção KVM/WSL2.
  - **Done:** README renderiza e lista os comandos previstos.

- [ ] 2.5 **Commit + push do grupo 2**
  - **Comando:** `git add . && git commit -m "chore: arquivos de repo (ignore, docs, instrucoes de agentes)" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 3. Workspace Nx, camadas e testes

- [ ] 3.1 **Criar o workspace Nx com `apps/native`, `apps/web`, `libs/domain`, `libs/application`, `libs/ui-shared`**
  - **Comando:** `docker compose run --rm node npx create-nx-workspace ...` + template `@ng-native/template` no app nativo; versões fixadas (spike 1.1).
  - **Contexto:** design.md Decisões 1 e 2.
  - **Done:** `docker compose run --rm node npx nx run-many -t typecheck` passa.

- [ ] 3.2 **ESLint + Prettier + `@nx/enforce-module-boundaries`**
  - **O que mudar:** tags por camada (`domain`, `application`, `infrastructure`, `ui`); regra proibindo cor literal fora dos arquivos de tema.
  - **Contexto:** spec project-foundation; design.md Decisão 3.
  - **Done:** `nx run-many -t lint` passa; um import proibido de teste falha o lint.

- [ ] 3.3 **Ports e providers por app**
  - **O que mudar:** `InjectionToken`s em `libs/application` (`Database`, `SecureStore`, `Preferences`, `Logger`, `Notifier`, `HttpGateway`); `app.config.ts` de cada app registrando os adapters (fakes por enquanto).
  - **Done:** os dois apps sobem com fakes; teste confirma a seleção por plataforma.

- [ ] 3.4 **Vitest + `@ng-native/testing` e teste de arquitetura**
  - **O que mudar:** configs de Vitest (`ngNative()` no nativo), um teste de domain, um de componente e `architecture.spec.ts` varrendo imports de `libs/domain` e `ui`.
  - **Done:** `docker compose run --rm node npx nx run-many -t test` fica verde.

- [ ] 3.5 **Commit + push do grupo 3**
  - **Comando:** `git add . && git commit -m "feat: workspace Nx, camadas e testes" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 4. Tema, ícones e shell de abas

- [ ] 4.1 **Tokens, Tailwind e `ThemeService`**
  - **O que mudar:** converter os tokens `oklch` do mockup para hex em CSS variables; derivar o dark; `@ng-native/tailwind` (versão decidida no spike); `ThemeService` com `ColorScheme`.
  - **Contexto:** design.md Decisão 9; UC-02.
  - **Done:** testes unit do UC-02 passam (light, dark e troca em runtime).

- [ ] 4.2 **Ícones Lucide**
  - **O que mudar:** `@ng-native/icons` + `@ng-icons/lucide` com `provideIcons`; componente de ícone com cor do tema.
  - **Contexto:** design.md Decisão 10.
  - **Done:** os 5 ícones do footer renderizam no Android.

- [ ] 4.3 **Rotas, `native-tabs-outlet` e `BottomNav`**
  - **O que mudar:** rotas Início/Tarefas/Plantas/Alertas; botão ⚙ que despacha `openSettings`; ativo em `accent`, Plantas em `pink`; `accessibilityLabel` e `testID`.
  - **Contexto:** ux-design.md; spec app-shell; UC-01.
  - **Done:** testes unit do UC-01 passam.

- [ ] 4.4 **Telas placeholder (`ScreenContainer`, `ScreenHeader`, `EmptyState`)**
  - **O que mudar:** 4 telas com título e "Em breve"; barra de status roxa.
  - **Done:** as 4 abas navegáveis com tema aplicado.

- [ ] 4.5 **Commit + push do grupo 4**
  - **Comando:** `git add . && git commit -m "feat(ui): tema, icones e shell de abas" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 5. Estado (NgRx)

- [ ] 5.1 **Store, features `ui`/`session` e effects**
  - **O que mudar:** `provideStore`/`provideEffects`; feature `ui` (`settingsOpen`, `toast`) e `session` (placeholder); selectors com `selectSignal` (ou o store decidido no spike 1.3).
  - **Contexto:** design.md Decisão 5; spec ui-state-store.
  - **Done:** testes de reducer, selector e effect passam.

- [ ] 5.2 **Teste de reflexão "sem entidades no Store"**
  - **O que mudar:** teste que inspeciona o estado inicial de todas as features e falha com tipo de `domain` ou array de entidades.
  - **Done:** teste verde; uma feature com entidade o faz falhar (validado localmente).

- [ ] 5.3 **Commit + push do grupo 5**
  - **Comando:** `git add . && git commit -m "feat(state): NgRx para UI e sessao" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 6. BottomSheet e sheet de Configurações

- [ ] 6.1 **`BottomSheet` próprio (decisão do spike 1.2)**
  - **O que mudar:** `<modal>` + backdrop + animação (+ gesto, se validado); `open()/close()/closed`; `HardwareBack` fecha o sheet; flag no estado `ui` do NgRx.
  - **Contexto:** design.md Decisão 11; UC-03.
  - **Done:** testes unit do UC-03 passam.

- [ ] 6.2 **Sheet de Configurações**
  - **O que mudar:** 5 `SettingsOption` desabilitadas com selo "em breve" (Perfil, Sincronização, Casas, Áreas, Sobre o App) e botão "Fechar"; abre pelo ícone ⚙.
  - **Contexto:** ux-design.md.
  - **Done:** o sheet abre e fecha sem trocar a rota.

- [ ] 6.3 **Commit + push do grupo 6**
  - **Comando:** `git add . && git commit -m "feat(ui): BottomSheet e sheet de configuracoes" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 7. Logger, erros e toast

- [ ] 7.1 **Logger com `ConsoleSink` e `FileSink`**
  - **O que mudar:** port `Logger`; `ConsoleSink` (todos); `FileSink` Android (`@ng-native/expo/file-system`, rotação, máscara de segredos); web só console; falha de gravação nunca propaga.
  - **Contexto:** design.md Decisão 14; spec logging-and-error-handling.
  - **Done:** testes unit do UC-06 (log, rotação, máscara) passam.

- [ ] 7.2 **`AppErrorHandler` e tela de erro**
  - **O que mudar:** `ErrorHandler` customizado que loga e aciona `ErrorScreen` por signal; "Tentar de novo" reinicia a navegação.
  - **Contexto:** design.md Decisão 15.
  - **Done:** teste com componente que lança exibe a tela e chama `logger.error`.

- [ ] 7.3 **`ToastHost`, `ToastService` e effect de erro**
  - **O que mudar:** toast no root (erro `red`, info `accent`, ~4s, 1 visível); effect que converte o evento de erro em toast.
  - **Done:** testes de toast (aparece, some, substitui) passam.

- [ ] 7.4 **Commit + push do grupo 7**
  - **Comando:** `git add . && git commit -m "feat: logger, tela de erro e toast" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 8. Banco de dados

- [ ] 8.1 **`Database`, migrations e `baseColumns`**
  - **O que mudar:** schema base (`id`, `version`, `syncStatus`, `deletedAt`), tabela de exemplo `exemplo` (só diagnóstico), runner de migrations por `user_version`.
  - **Contexto:** design.md Decisão 6; UC-04.
  - **Done:** testes unit do UC-04 (migrate em memória, no-op, falha com rollback) passam.

- [ ] 8.2 **Migrations no boot com tratamento de falha**
  - **O que mudar:** inicialização antes de renderizar as abas (`APP_INITIALIZER`/provider de init); falha vai para o `AppErrorHandler`.
  - **Done:** falha forçada exibe a tela de erro sem fechar o app.

- [ ] 8.3 **Port `Repository` e suíte de contrato**
  - **O que mudar:** repositório genérico (insert, update, soft delete, list) + suíte de contrato reutilizável por adapter.
  - **Done:** suíte verde contra o adapter nativo (fake/in-memory) e contra o fake.

- [ ] 8.4 **Adapter web IndexedDB**
  - **O que mudar:** adapter atrás do mesmo port.
  - **Done:** a suíte de contrato passa no adapter web.

- [ ] 8.5 **Repositórios reativos por signal**
  - **O que mudar:** signals de listagem + invalidação após escritas (e listener do expo-sqlite, se validado no spike 1.4); testes do UC-05 (escrita externa, join, soft delete).
  - **Done:** testes unit do UC-05 passam.

- [ ] 8.6 **Commit + push do grupo 8**
  - **Comando:** `git add . && git commit -m "feat(db): Database, migrations e repositorios reativos" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 9. Rede e storage

- [ ] 9.1 **Ports e adapters de storage seguro e preferências**
  - **O que mudar:** `SecureStorage` e `Storage` no nativo; adapters web próprios (`localStorage`/IndexedDB); fakes para testes.
  - **Contexto:** design.md Decisão 13.
  - **Done:** teste de salvar, ler e apagar passa; no web a escrita persiste após recarregar.

- [ ] 9.2 **`DeviceIdProvider`**
  - **O que mudar:** gera UUID uma vez e persiste em `Storage`.
  - **Done:** teste confirma o mesmo id após "reabrir".

- [ ] 9.3 **`HttpClient` com interceptor `X-Device-Id`, timeout e erros tipados**
  - **O que mudar:** `provideNativeHttpClient()` no nativo e `provideHttpClient()` no web; erros `NetworkError`, `TimeoutError`, `AuthError`, `HttpError`; URL por `environment`.
  - **Contexto:** design.md Decisão 12; UC-07.
  - **Done:** testes unit do UC-07 passam.

- [ ] 9.4 **Commit + push do grupo 9**
  - **Comando:** `git add . && git commit -m "feat(net): HttpClient, storage e device id" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 10. Notificações (base)

- [ ] 10.1 **Canal Android e `Notifier`**
  - **O que mudar:** `@ng-native/expo/notifications` cria o canal no boot sem pedir permissão; `NoopNotifier` na web.
  - **Contexto:** design.md Decisão 16.
  - **Done:** teste do adapter confirma o canal e nenhuma chamada de permissão.

- [ ] 10.2 **Background task no-op**
  - **O que mudar:** `@ng-native/expo/background-task` registra tarefa que só loga; desligada na web.
  - **Done:** teste confirma o registro no Android e nenhum registro na web.

- [ ] 10.3 **Commit + push do grupo 10**
  - **Comando:** `git add . && git commit -m "feat(notifications): canal e background task base" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 11. Diagnóstico (build de teste)

- [ ] 11.1 **Rota `__diagnostico` condicional à flag de build de teste**
  - **O que mudar:** inserir registro de exemplo, lista reativa, chamar `/version`, "Forçar erro"; excluída do build de produção.
  - **Contexto:** ux-design.md; UC-05, UC-06, UC-07.
  - **Done:** com a flag a rota abre; sem a flag não existe (teste de roteamento).

- [ ] 11.2 **`testID` estáveis**
  - **Done:** lista de `testID` documentada no README e usada pelos fluxos dos grupos 14 e 15.

- [ ] 11.3 **Commit + push do grupo 11**
  - **Comando:** `git add . && git commit -m "feat: tela de diagnostico para E2E" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 12. PWA (shell Angular web)

- [ ] 12.1 **Shell `apps/web` com Angular Router web e islands**
  - **O que mudar:** rotas Início/Tarefas/Plantas/Alertas, cada uma montando sua tela por `ng-native-island`; footer e sheet equivalentes; tema por `prefers-color-scheme`.
  - **Contexto:** design.md Decisão 7; spec pwa-shell.
  - **Done:** as 4 rotas navegam no Chromium e o sheet abre.

- [ ] 12.2 **Manifest e `@angular/service-worker`**
  - **O que mudar:** `manifest.webmanifest` (PlantumHibis, 192/512 + maskable, `standalone`, cores), `ngsw-config.json`, `provideServiceWorker` só em produção.
  - **Contexto:** design.md Decisão 8; UC-08.
  - **Done:** teste unit valida o manifest; Playwright confirma SW ativo e `installable`.

- [ ] 12.3 **Recursos nativos desligados na web**
  - **O que mudar:** `NoopNotifier`, sem background task; log só no console.
  - **Done:** o shell web inicia sem erros no console.

- [ ] 12.4 **Commit + push do grupo 12**
  - **Comando:** `git add . && git commit -m "feat(pwa): shell Angular web, manifest e service worker" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 13. Docker: build do APK

- [ ] 13.1 **`Dockerfile.build` e serviço `build`**
  - **O que mudar:** imagem final (versões registradas no spike 1.1), `expo prebuild` + Gradle release, keystore de debug, cache do Gradle em volume, APK em `artifacts/`.
  - **Comando:** `docker compose run --rm build`
  - **Contexto:** design.md Decisão 18; UC-09.
  - **Done:** `artifacts/*.apk` existe e `aapt dump badging` confere o pacote.

- [ ] 13.2 **Script de pós-build (`scripts/verify-apk.sh`)**
  - **Done:** sem APK retorna código ≠ 0 com mensagem clara (teste do UC-09).

- [ ] 13.3 **Serviço `test` (Vitest) no compose**
  - **Comando:** `docker compose run --rm test`
  - **Done:** suíte verde via Docker.

- [ ] 13.4 **Commit + push do grupo 13**
  - **Comando:** `git add . && git commit -m "feat(docker): build do APK e testes via Docker" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 14. Docker: E2E Android (Maestro)

- [ ] 14.1 **`Dockerfile.e2e` e serviço `e2e` com pré-checagem de KVM**
  - **O que mudar:** emulador x86_64 headless + Maestro; `scripts/check-kvm.sh` falha cedo apontando para o README.
  - **Contexto:** design.md Decisão 18; UC-10.
  - **Done:** sem `/dev/kvm` a mensagem aparece; com KVM o emulador sobe.

- [ ] 14.2 **Fluxos Maestro (`e2e/android/*.yaml`)**
  - **O que mudar:** smoke UC-01/02/03 (abas, dark, sheet, botão voltar), UC-04 (primeiro boot), UC-05/06/07 pelo build de teste com `backend-mock` no compose.
  - **Comando:** `docker compose run --rm e2e`
  - **Done:** fluxos passam; screenshots e logs em `artifacts/`; falha forçada gera código ≠ 0; `adb run-as` lê o log do app (UC-06).

- [ ] 14.3 **Commit + push do grupo 14**
  - **Comando:** `git add . && git commit -m "feat(e2e): Maestro no Android via Docker" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 15. Docker: E2E web (Playwright)

- [ ] 15.1 **`Dockerfile.web` e serviço `web-e2e`**
  - **O que mudar:** build do shell web servida localmente + imagem oficial do Playwright.
  - **Comando:** `docker compose run --rm web-e2e`
  - **Done:** specs executam no Chromium.

- [ ] 15.2 **Specs Playwright (`e2e/web/*.spec.ts`)**
  - **O que mudar:** UC-01 (abas), UC-02 (`colorScheme: 'dark'`), UC-03 (sheet), UC-04 (primeira visita), UC-08 (SW ativo, manifest válido, recarregar offline).
  - **Done:** todas verdes; relatório e traces em `artifacts/`.

- [ ] 15.3 **Commit + push do grupo 15**
  - **Comando:** `git add . && git commit -m "feat(e2e): Playwright no PWA via Docker" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 16. Pré-PR

- [ ] 16.1 **Atualizar `README.md`**
  - **O que mudar:** comandos reais, aviso de alpha e versões fixadas, instalação do PWA, `testID`, seção KVM/WSL2, estrutura final de pastas, gaps conhecidos.
  - **Done:** um dev novo executa só os comandos do README e obtém APK, testes e E2E.

- [ ] 16.2 **Validação completa via Docker**
  - **Comando:** `docker compose run --rm test && docker compose run --rm build && docker compose run --rm e2e && docker compose run --rm web-e2e`
  - **Done:** os 4 serviços passam; screenshots do Android (light e dark) e do PWA anexados ao PR.

- [ ] 16.3 **`graphify update .`**
  - **Contexto:** regra `graphify-update`.
  - **Done:** `graphify-out/` atualizado.

---

## 17. Commit final e PR

- [ ] 17.1 **Commit final + push**
  - **Comando:** `git add . && git commit -m "feat: esqueleto do app Angular NG Native (Android + PWA)" && git push`
  - **Done:** `git log --oneline -1` mostra o commit; árvore limpa.

- [ ] 17.2 **[AUTORIZAÇÃO] Criar Pull Request**
  - **Comando:** `gh pr create --base develop --head feature/ngnative-app-skeleton --title "feat: esqueleto do app Angular NG Native (Android + PWA)"` (descrição ASCII-safe, com screenshots e checklist de specs).
  - **Contexto:** o remote é GitHub, não Azure; não usa o MCP azure-repos.
  - **Done:** PR aberto com destino `develop`; aguardar aprovação e squash merge.

- [ ] 17.3 **Pós-merge**
  - **O que fazer:** sincronizar `develop`, mover a change para `openspec/changes/archive/`, remover o worktree (`git worktree remove worktrees/skeleton` + `git branch -d feature/ngnative-app-skeleton`) e propor a change 2 (auth).
  - **Done:** `git worktree list` sem o worktree; change arquivada.
