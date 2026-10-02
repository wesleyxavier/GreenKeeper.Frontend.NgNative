## Why

O app GreenKeeper/PlantumHibis já tem uma base em .NET MAUI e uma em React Native (esqueleto proposto). Wesley quer uma terceira em **Angular**, stack que domina, usando o **NG Native** (Angular 22 renderizando views nativas via Fabric/Expo), mantendo Android 100% nativo, Docker para build/E2E e um PWA. O NG Native está em alpha, então este esqueleto serve tanto de fundação quanto de **avaliação de viabilidade**, com spikes que podem replanejar o design antes de qualquer módulo de negócio.

## What Changes

- Novo workspace **Nx** no repo `GreenKeeper.Frontend.NgNative`: `apps/native` (Angular 22 + NG Native + Expo SDK 57), `apps/web` (shell Angular + Vite para o PWA) e `libs/` compartilhadas (`domain`, `application`).
- Camadas `domain / application / infrastructure / ui` com ports e adapters por plataforma (nativo vs web), injetados pelo DI do Angular.
- **Angular Router nativo** com `native-tabs-outlet`: Início, Tarefas, Plantas, Alertas como rotas; o ícone ⚙ é um gatilho que abre o sheet, sem rota própria.
- Tema PlantumHibis via **Tailwind + CSS variables** e `ColorScheme` (signal), light/dark pelo sistema; ícones **ng-icons Lucide**.
- **BottomSheet próprio** (`modal` + animação), já que o NG Native não tem sheet; sheet de Configurações com as 5 opções desabilitadas ("em breve").
- **NgRx** (Store + Effects) só para estado de UI/sessão/eventos, com a regra "sem entidades de domínio no Store" e teste de reflexão.
- **Database** (módulo `@ng-native/expo/database`, expo-sqlite) com migrations por `user_version`, colunas base de sync, repositórios reativos por signal com invalidação e adapter **IndexedDB** na web atrás do mesmo port.
- `HttpClient` nativo (`provideNativeHttpClient()`) com interceptor `X-Device-Id`, erros tipados e URL por ambiente; `SecureStorage`/`Storage` e equivalentes web atrás de ports.
- Logger em arquivo (Android) + console; `ErrorHandler` global do Angular, tela de erro padrão e toast global.
- Base de notificações: canal Android, toast global e background task no-op.
- **PWA:** shell Angular web, islands para os componentes nativos, `@angular/service-worker`, manifest instalável e abertura offline.
- **Docker:** build do APK (Node 22 + JDK 17 + Android SDK), E2E Android (emulador headless + Maestro) e E2E web (Playwright), cada um com smoke test.
- **Vitest** + `@ng-native/testing`; arquivos de repo (`.gitignore`, `.graphifyignore`, `README.md`, `CLAUDE.md`/`AGENTS.md`/`GEMINI.md`) e graphify inicial.
- Sem breaking changes: repo novo; MAUI, RN e Backend ficam intactos.

## Capabilities

### New Capabilities
- `project-foundation`: workspace Nx, apps `native` e `web`, libs compartilhadas, TypeScript strict, camadas verificáveis, DI com adapters por plataforma, Vitest.
- `app-shell`: Angular Router nativo com 4 abas + gatilho ⚙, tema PlantumHibis, ícones Lucide, BottomSheet próprio e sheet de Configurações.
- `ui-state-store`: NgRx Store + Effects para UI/sessão/eventos e regra de não guardar entidades no Store.
- `local-database`: módulo Database (expo-sqlite), migrations, colunas base, repositórios reativos por signal e adapter IndexedDB na web.
- `network-and-storage`: `HttpClient` nativo, interceptor `X-Device-Id`, erros tipados, `SecureStorage`/`Storage` e equivalentes web.
- `logging-and-error-handling`: logger em arquivo + console, `ErrorHandler` global, tela de erro e toast padrão.
- `notifications-foundation`: canal Android, toast global e estrutura de background task.
- `pwa-shell`: shell Angular web com islands, `@angular/service-worker`, manifest e abertura offline.
- `docker-build-and-e2e`: imagens de build do APK, E2E Android (Maestro + emulador) e E2E web (Playwright), com smoke tests.
- `repo-bootstrap`: `.gitignore`, `.graphifyignore`, README, instruções de agentes, graphify inicial, branches e worktree.

### Modified Capabilities
<!-- Nenhuma: repo novo, sem specs existentes. -->

## Impact

- **Repo novo:** `GreenKeeper.Frontend.NgNative` (GitHub `wesleyxavier/GreenKeeper.Frontend.NgNative`), vazio; `develop` precisa ser criada a partir de um commit inicial.
- **Dependências novas (npm):** `@angular/*` 22, `@ng-native/*` (platform, components, router, device, expo, icons, tailwind, testing, web, fabric, metro), `expo` SDK 57, `react-native` 0.86, `@ngrx/store` e `@ngrx/effects`, `@ng-icons/lucide`, `tailwindcss`, `@angular/service-worker`, `vitest`, `@playwright/test`; Maestro (CLI, só na imagem Docker); Nx.
- **Infra:** imagens Docker (`Dockerfile.build`, `Dockerfile.e2e`, Playwright) e `docker-compose.yml`; nenhum SDK no host.
- **Riscos principais:** NG Native alpha; compatibilidade do NgRx com o renderer; falta de live query, BottomSheet e ErrorBoundary prontos; router nativo sem versão web; KVM no Windows. Todos validados por spikes no início das tasks.
- **Fora de escopo:** módulos de negócio, sync com o Backend, geração de alertas, deploy do PWA, iOS, i18n completa, alterações no Backend/MAUI/RN.
