## Context

O ecossistema GreenKeeper tem o frontend MAUI (`GreenKeeper.Frontend`, C#) e o esqueleto React Native proposto (`GreenKeeper.Frontend.Reactnative`). Esta change cria a terceira base, `GreenKeeper.Frontend.NgNative` (repo vazio), em **Angular 22 com NG Native** (`ng-native.com`): o `Renderer2` do Angular é substituído por um que monta a árvore para o **Fabric** do React Native, desenhando `UIView`/`android.view.View` sem WebView e sem React no caminho de render. Base declarada: **Expo SDK 57, React Native 0.86 (New Architecture), Node ≥ 22.18**. O projeto está em **alpha** (APIs 0.x mudam, "não pronto para produção"; 6 contribuidores, 365 stars).

Restrições: Docker obrigatório para build e E2E (regra `docker-isolation`); Windows sem KVM nativo; paths curtos para worktrees; remote GitHub; MAUI, RN e Backend intactos; mockup de referência em `GreenKeeper.Frontend/design/mockups/screens/`; **mesmos princípios do esqueleto RN** (camadas, Redux, offline-first, tema, 4 abas + sheet, PWA, Docker, Maestro/Playwright).

O que o NG Native oferece (documentação): Angular Router nativo (`native-stack-outlet`, `native-tabs-outlet`, `native-header`, deep links, botão voltar), componentes (`view`, `text`, `pressable`, `scroll-view`, `virtual-list`, `modal`, `text-input`, `switch`, `image`, `activity-indicator`), `@ng-native/device` (signals: `ColorScheme`, `Screen`, `HardwareBack`, `DeepLinks`...), `@ng-native/expo` (Database/expo-sqlite, Storage/AsyncStorage, SecureStorage/expo-secure-store, file-system, notifications, background-task, location, camera, image-picker, network), `@ng-native/icons` (+ `@ng-icons/lucide`, via react-native-svg), `@ng-native/tailwind`, `@ng-native/testing` (Vitest + Testing Library + fake Fabric), Maestro documentado e web por **islands** (`@ng-native/web`, Vite). Limitações documentadas: sem `@angular/animations`, i18n parcial, sem SSR/hidratação, `provideNativeHttpClient()` obrigatório, `section-list` limitado, hot reload só de template/estilo, componentes em minúsculas. **Não existem**: BottomSheet, ErrorBoundary, live query de banco, versão web do router nativo.

Insumos: `concept.md`, `proposal.md`, 10 specs, `use-cases.md` (UC-01 a UC-10) e `ux-design.md`.

## Goals / Non-Goals

**Goals:**
- Workspace Nx com `apps/native`, `apps/web` e libs `domain`/`application` compartilhadas.
- Camadas verificáveis, adapters por plataforma via DI do Angular.
- Shell de UI: 4 abas + sheet, tema Tailwind, ícones Lucide, tela de erro, toast.
- Banco local reativo (Database + invalidação por signal) e adapter IndexedDB na web.
- NgRx só para UI/sessão/eventos, sem entidades.
- Logger arquivo + console; base de notificações; PWA; Docker (build, E2E Android e web).
- **Avaliar a viabilidade do NG Native alpha** por spikes antes de construir sobre ele.

**Non-Goals:**
- Módulos de negócio, sync, lógica de alertas, deploy do PWA, iOS, assinatura/Play Store.
- Compartilhar a camada de UI entre native e web (o router nativo não existe na web).
- `@angular/animations`, SSR, i18n completa.
- Alterar o Backend, o MAUI ou o RN.

## Decisions

1. **Workspace Nx.** `apps/native` (template `@ng-native/template`), `apps/web` (Angular + Vite), `libs/domain`, `libs/application`, `libs/ui-shared` (tokens e modelos de tema, sem componentes). Nx tem suporte e schematics do NG Native e impõe fronteiras com `@nx/enforce-module-boundaries`. Alternativas: dois repos (duplica domain/application); npm workspaces puros (sem grafo de dependência nem fronteiras).

2. **App nativo via template oficial e prebuild.** `npx create-expo-app --template @ng-native/template`, Metro preset do NG Native; `android/` gerado por `expo prebuild` dentro do Docker e não versionado; build com `expo run:android --variant release`/Gradle. EAS fica fora de escopo (Docker local). Versões de Angular/NG Native/Expo/RN **fixadas sem `^`** após o spike.

3. **Camadas e DI do Angular.** `domain` (TS puro), `application` (casos de uso + **ports** como `InjectionToken`s), `infrastructure` (adapters), `ui` (componentes). Cada app tem `app.config.ts` que registra os adapters do seu alvo; as telas só injetam ports/casos de uso. Fronteiras por `@nx/enforce-module-boundaries` + teste de arquitetura. Alternativa: container DI externo (desnecessário, Angular já tem DI).

4. **Navegação nativa: Angular Router + `native-tabs-outlet`.** Rotas `''` (Início), `tarefas`, `plantas`, `alertas`; o ícone ⚙ é um botão customizado da tab bar que despacha `openSettings` (o NG Native indica suporte a "custom buttons opening sheets"; confirmar no spike). Rota escondida `__diagnostico` condicional a flag de build de teste. `provideNativeRouter()` com `withLinkParent`, `withHeaderDefaults`, `withTabDefaults`.

5. **Estado: NgRx (Store + Effects) com signals.** Features `ui` (`settingsOpen`, `toast`) e `session` (placeholder); `selectSignal` nos componentes; Effects para eventos (erro → toast). Teste de reflexão garante que nenhuma feature guarde entidades. **Compatibilidade com o renderer nativo e com o runtime Hermes não é documentada** → spike; fallback: store próprio com signals e a mesma API de selectors. Alternativas: NgRx SignalStore (mais leve, avaliar no spike); Akita/NGXS (menos alinhados à ideia de Redux já aprovada).

6. **Banco: módulo `Database` (expo-sqlite), sem ORM.** Migrations versionadas por `PRAGMA user_version` (cada uma em transação, rollback total em falha). Schema com helper `baseColumns` (`id`, `version`, `syncStatus`, `deletedAt`). Acesso por **repositórios** (ports em `application`) que usam SQL parametrizado. **Reatividade:** o módulo não documenta live query; cada repositório expõe signals de listagem e uma invalidação disparada após qualquer escrita (e, se o spike validar, via `addDatabaseChangeListener` do expo-sqlite para escritas feitas por outro caminho). Joins declaram as tabelas observadas, com teste por tela. Alternativas: Drizzle com expo-sqlite (funciona sem o Angular, mas tem live query atrelada a hooks React, incompatível com o princípio "sem React no render"); WatermelonDB (modelo próprio e hooks).

7. **Web: shell Angular + islands.** `apps/web` é um Angular web padrão (Angular Router web, Vite com `ngNativeWeb()`), que monta as telas como **islands** (`ng-native-island`) — o router nativo não tem versão web, e `<view>` não pode aparecer em templates do host, só dentro de islands. Reuso: `libs/*`, estado NgRx e componentes de tela. Banco web: **IndexedDB** atrás do mesmo port (Dexie ou wrapper próprio), validado pela mesma **suíte de contrato**. `Storage`/`SecureStorage` do NG Native descartam escritas na web, então o web tem adapters próprios (`localStorage`/IndexedDB). Alternativa: bundle único com o mesmo router (inviável, o router nativo não tem versão web).

8. **PWA: `@angular/service-worker`** (`ngsw-config.json`, `provideServiceWorker`, registro só em produção), `manifest.webmanifest` (PlantumHibis, 192/512 + maskable, `display: standalone`, cores do tema). Offline: precache do shell + dados no IndexedDB. Hospedagem fora de escopo (Docker/local).

9. **Tema: Tailwind + CSS variables + `ColorScheme`.** `@ng-native/tailwind` (Tailwind 4 com `native.css` ou 3 com preset, definir no spike); tokens `oklch` do mockup **convertidos uma vez para hex** (o Fabric/CSS nativo não garante `oklch`) em variáveis CSS; variante `dark:` ligada ao signal `ColorScheme`; variantes `android:`/`web:` quando necessário. Os tokens dark não existem no mockup e serão **derivados** e validados visualmente. Regra de lint proíbe cor literal fora dos arquivos de tema.

10. **Ícones: `@ng-native/icons` + `@ng-icons/lucide`** (`home`, `list-checks`, `leaf`, `bell`, `settings`, ...), renderizados por react-native-svg por baixo, recoloridos via input `color`.

11. **BottomSheet próprio.** O NG Native não tem sheet. Componente baseado em `<modal>` com backdrop e animação por Reanimated/`AnimatedStyle` (worklets) e gesto de arrastar; fecha também por `HardwareBack` (device service). API `open()/close()/closed`. Alternativa: sheet sem gesto (só botão/backdrop) se o spike mostrar fragilidade dos worklets.

12. **Rede:** `HttpClient` com `provideNativeHttpClient()` no nativo e `provideHttpClient()` no web; interceptor de `X-Device-Id`, timeout e mapeamento para erros tipados (`NetworkError`, `TimeoutError`, `AuthError`, `HttpError`); URL por arquivo de ambiente (`environment.ts` por alvo). Dados remotos por `httpResource`/signals (sem TanStack Query).

13. **Storage:** `SecureStorage` (expo-secure-store) para token e `Storage` (AsyncStorage) para preferências, atrás de ports; device id UUID gerado uma vez e persistido em `Storage`.

14. **Logger próprio** com `ConsoleSink` e `FileSink` (Android, `@ng-native/expo/file-system`, rotação por tamanho/quantidade, máscara de segredos); web só console; falha ao gravar nunca propaga. Legível via `adb run-as`.

15. **Erros:** `AppErrorHandler` substitui o `ErrorHandler` do Angular, loga e aciona a tela de erro por signal (não há ErrorBoundary); "Tentar de novo" reinicia a árvore/navegação. `ToastHost` no root + `ToastService`/evento `notify`.

16. **Notificações:** `@ng-native/expo/notifications` cria o canal Android no boot **sem pedir permissão**; `@ng-native/expo/background-task` registra uma tarefa no-op que só loga; `NoopNotifier` na web.

17. **Testes:** Vitest em todos os projetos; `@ng-native/testing` (fake Fabric + Testing Library) no app nativo; Maestro (`e2e/android/*.yaml`) com `testID`/`accessibilityLabel` (o NG Native documenta `testID`); Playwright (`e2e/web/*.spec.ts`).

18. **Docker:** `Dockerfile.build` (Node 22.18+, **JDK 17**, Android cmdline-tools; versões finais registradas no spike), `expo prebuild` + Gradle release, keystore de debug só para E2E, cache do Gradle em volume; `Dockerfile.e2e` (emulador x86_64 headless + Maestro, exige `/dev/kvm` com pré-checagem); `Dockerfile.web` + Playwright; `docker-compose.yml` com `build`, `test`, `e2e`, `web-e2e` e `backend-mock` (UC-07). KVM: WSL2 com virtualização aninhada ou runner Linux, como no RN.

19. **Spikes primeiro (Task 1), com ponto de replanejamento:** (a) template do NG Native roda e builda APK no Docker (Node/JDK/SDK); (b) tabs + ícone ⚙ customizado + `modal` como sheet no Android; (c) NgRx e `provideNativeHttpClient()` no renderer nativo; (d) `Database` + migrations + invalidação reativa; (e) islands + Angular service worker + IndexedDB no shell web; (f) emulador + Maestro com KVM. **Critério de parada:** se (a) ou (b) falharem, a change é replanejada ou o NG Native é abandonado em favor do RN; falhas em (c) a (e) têm fallback no design.

20. **Bootstrap do repo:** repo vazio; commit inicial (`openspec/`, `.claude/`, `.gitignore`, README mínimo) em `main`, `develop` criada dele, trabalho em `worktrees/skeleton` na branch `feature/ngnative-app-skeleton`. Commit/push/PR exigem autorização explícita; PR por `gh` (remote GitHub), squash em `develop`.

## Risks / Trade-offs

- [NG Native alpha: APIs mudam, bugs, base pequena de contribuidores] → Spikes; versões fixadas sem `^`; framework isolado atrás de ports; projeto tratado como avaliação paralela ao RN; critério de parada explícito.
- [Sem BottomSheet e sem ErrorBoundary prontos] → Componente próprio e `AppErrorHandler`; testes dedicados.
- [Sem live query] → Invalidação por repositório + teste de reatividade por tela; avaliar o listener do expo-sqlite no spike.
- [Router nativo sem versão web] → Shell web separado com islands; só domain/application/estado compartilhados; mais código que no RN.
- [NgRx/bibliotecas Angular não validadas com o renderer] → Spike (c); fallback para store próprio com signals.
- [`Storage`/`SecureStorage` descartam escritas na web] → Adapters web próprios atrás dos ports.
- [`@angular/animations` e i18n completa indisponíveis] → Fora de escopo; usar `animate.enter/leave` e Reanimated.
- [Emulador Android em Docker exige KVM] → WSL2 com nested virtualization; plano B runner Linux; pré-checagem.
- [Expo SDK 57/RN 0.86 recentes: módulos de terceiros podem não suportar] → Preferir módulos do `@ng-native/expo`; verificar compatibilidade no spike.
- [Tokens dark inexistentes no mockup] → Derivados e revisados pelo usuário nos screenshots.
- [Esqueleto grande: 2 apps + libs + Docker] → Tasks granuladas, commit por grupo, spikes primeiro.
- [Trade-off] Terceira base do mesmo produto, em stack alpha: aceito por decisão do usuário como avaliação; o RN permanece como alternativa segura.

## Migration Plan

Repo novo, sem migração de dados. Sequência: bootstrap do repo → worktree → spikes → fundação (Nx, camadas, testes) → UI shell → estado → banco → rede/storage → logs/erros/notificações → PWA → Docker/E2E → docs/graphify → PR. **Rollback:** descartar a branch `feature/ngnative-app-skeleton`; nada fora do repo novo é alterado.

## Open Questions

- Versões exatas de Angular 22, `@ng-native/*`, Expo 57, RN 0.86, Node e `compileSdk`/`build-tools`; JDK 17 suficiente? (spike a).
- O tab bar nativo aceita um item que só dispara ação (⚙) sem rota? (spike b).
- NgRx Store ou SignalStore no renderer nativo? (spike c).
- `addDatabaseChangeListener` do expo-sqlite acessível pelo módulo `Database` do NG Native? (spike d).
- Dexie ou wrapper próprio para IndexedDB; Tailwind 3 (preset) ou 4 (`native.css`).
- Valores finais dos tokens dark (derivados; aprovação visual do usuário).
- Imagem de emulador Docker: pronta ou montada sobre `sdkmanager` (spike f).
