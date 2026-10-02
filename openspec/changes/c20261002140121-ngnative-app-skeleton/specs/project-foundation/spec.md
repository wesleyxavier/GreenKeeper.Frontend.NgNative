## ADDED Requirements

### Requirement: Workspace Nx com app nativo, shell web e libs compartilhadas
O repo SHALL ser um workspace Nx com `apps/native` (Angular 22 + NG Native + Expo SDK 57), `apps/web` (shell Angular + Vite) e `libs/domain` e `libs/application` compartilhadas entre os dois apps, em TypeScript com `strict: true`, com ESLint e Prettier e targets `lint`, `typecheck` e `test`.

#### Scenario: Verificações estáticas passam
- **WHEN** o dev roda `nx run-many -t lint typecheck`
- **THEN** todos os projetos terminam sem erros

#### Scenario: Libs compartilhadas
- **WHEN** `apps/native` e `apps/web` importam de `libs/domain` e `libs/application`
- **THEN** ambos compilam sem duplicar código de domínio ou de casos de uso

### Requirement: Camadas com dependência unidirecional
O código SHALL ser organizado em `domain`, `application`, `infrastructure` e `ui`, onde `ui` depende de `application`, `application` depende de `domain`, `infrastructure` implementa os ports de `application` e `domain` não importa nenhuma outra camada, Angular, NG Native ou Expo.

#### Scenario: Domain não importa de outras camadas
- **WHEN** um teste de arquitetura varre os imports de `libs/domain`
- **THEN** nenhum import aponta para `application`, `infrastructure`, `ui`, `@angular/*`, `@ng-native/*` ou `expo-*`

#### Scenario: UI não importa infrastructure direto
- **WHEN** um teste de arquitetura varre os imports de `ui`
- **THEN** nenhum import aponta para `infrastructure`, exceto o arquivo de providers do app

### Requirement: Adapters por plataforma via DI do Angular
Cada app SHALL registrar, em seu arquivo de providers, o adapter de cada port de plataforma (banco, storage seguro, preferências, logger, notificações, HTTP), usando injection tokens definidos em `application`.

#### Scenario: Adapter nativo no Android
- **WHEN** o app nativo inicia
- **THEN** os providers entregam os adapters nativos (Database, SecureStorage, Storage, logger em arquivo)

#### Scenario: Adapter web no PWA
- **WHEN** o shell web inicia
- **THEN** os providers entregam os adapters web e os recursos sem suporte ficam explicitamente desligados

### Requirement: Testes unitários executáveis
O projeto SHALL ter Vitest com `@ng-native/testing` (fake Fabric) configurado no app nativo e Vitest no web e nas libs, com ao menos um teste de domain e um de componente.

#### Scenario: Suíte passa
- **WHEN** o dev roda `nx run-many -t test`
- **THEN** a suíte termina verde

### Requirement: Versões fixadas e validadas por spike
As versões de Angular, `@ng-native/*`, Expo, React Native, Node e JDK SHALL ser fixadas e registradas após os spikes iniciais, dado o status alpha do NG Native.

#### Scenario: Registro de versões
- **WHEN** os spikes terminam
- **THEN** o design.md lista as versões exatas usadas e o `package.json` as fixa sem `^`
