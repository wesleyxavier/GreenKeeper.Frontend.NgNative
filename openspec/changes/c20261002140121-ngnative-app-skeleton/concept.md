## Visão

**Problema:** Wesley quer a mesma base de app GreenKeeper/PlantumHibis em Angular (stack que domina), em vez de React, mantendo Android 100% nativo, build e E2E reproduzíveis por Docker e um PWA instalável. O NG Native (Angular 22 renderizando views nativas via Fabric/Expo) é a candidata, mas está em **alpha**, então a base precisa ser construída com spikes que validem a viabilidade antes de migrar módulos.
**Solução:** App GreenKeeper/PlantumHibis em **Angular 22 + NG Native + Expo SDK 57**, com os mesmos princípios do esqueleto React Native: sem módulos de negócio, camadas Clean Architecture, tema, 4 abas + sheet, banco local, **NgRx** só para estado de UI/sessão/eventos, logs, erros, notificações base, PWA, build do APK e E2E (Maestro + Playwright) via Docker. O PWA é um **shell Angular web** (Angular Router web + `@angular/service-worker`) que monta os componentes nativos como islands, compartilhando `domain` e `application` com o app nativo. Roadmap de módulos idêntico ao RN.
**Usuários-alvo:** Wesley (dev) nesta change; usuários de plantas em Android e no PWA a partir das changes de módulos.

## Mapa Funcional

```mermaid
mindmap
  root((GreenKeeper NgNative - Esqueleto))
    Projeto
      Angular 22 + NG Native + Expo SDK 57
      TypeScript strict, Node 22.18+
      Template @ng-native/template
      Workspace Nx
      ESLint + Prettier
    Camadas
      domain - TS puro
      application - casos de uso e ports
      infrastructure - adapters
      ui - componentes nativos
    UI Base
      Tema PlantumHibis - Tailwind + CSS vars
      ColorScheme signal light e dark
      Ícones ng-icons lucide
      BottomSheet próprio
      Footer 5 ícones
        Início
        Tarefas
        Plantas
        Alertas
        Configurações - gatilho de sheet
    Navegação
      Angular Router nativo
      native-tabs-outlet
      Deep links e botão voltar
    Estado
      NgRx Store - UI, sessão, eventos
      NgRx Effects
      Signals via selectSignal
    Dados e Reatividade
      Módulo Database - expo-sqlite
      Migrations por user_version
      Repositórios com signals e invalidação
      Web - IndexedDB atrás de port
    Rede
      HttpClient nativo
      Interceptor X-Device-Id
      Config de URL
    Storage
      SecureStorage - token
      Storage - preferências
    Notificações
      Módulo notifications
      Canal Android
      Background task no-op
    Erros e Logs
      ErrorHandler global
      Tela de erro por signal
      Toast global
      Logger arquivo + console
    PWA
      Shell Angular web - Vite
      Islands com componentes nativos
      Angular service worker
      Manifest instalável
    Infra Docker
      Build APK - JDK 17
      E2E Android - emulador + Maestro
      E2E web - Playwright
    Qualidade
      Vitest + ng-native/testing
      Teste de arquitetura
```

## Jornadas de Usuário

As jornadas de **Dev**, **Usuário Android** e **Usuário PWA** são as mesmas do esqueleto RN (`GreenKeeper.Frontend.Reactnative`), com estas diferenças: `npm test` roda Vitest; o erro fatal passa pelo `ErrorHandler` do Angular e abre a tela de erro; o PWA carrega o shell Angular web, o Angular service worker cacheia os assets e as islands montam os componentes nativos.

### Dev

```mermaid
flowchart TD
    A([Dev: clona o repo]) --> B[npm install]
    B --> C[docker compose run build: prebuild + gradle assembleRelease]
    C --> D[APK gerado em artifacts/]
    D --> E[docker compose run e2e: sobe emulador headless]
    E --> F[Maestro instala APK e roda smoke test]
    F --> G{Smoke passou?}
    G -->|Sim| H[npm test: Vitest]
    G -->|Não| I[Lê log e screenshot em artifacts/]
    I --> C
    H --> J[docker compose run web-e2e: Playwright no PWA]
    J --> K([Dev: pronto pra implementar módulos])
```

### Usuário Android

```mermaid
flowchart TD
    A([Usuário: instala o APK]) --> B[Abre o app]
    B --> C[Splash PlantumHibis]
    C --> D[Migrations do Database rodam no 1º boot]
    D --> E{Falha na inicialização?}
    E -->|Sim| F[Tela de erro padrão + log em arquivo]
    E -->|Não| G[Aba Início]
    G --> H[Footer: Início, Tarefas, Plantas, Alertas, Configurações]
    H --> I[Troca de aba: tela vazia com título]
    H --> J[Toca no ícone Configurações: abre bottom sheet]
    J --> K[Fecha o sheet]
    G --> L{Sistema em dark mode?}
    L -->|Sim| M[Tema escuro]
    L -->|Não| N[Tema claro]
    I --> O{Erro em runtime?}
    O -->|Sim| P[Toast de erro + log em arquivo]
    O -->|Não| Q([App estável])
    K --> Q
```

### Usuário PWA

```mermaid
flowchart TD
    A([Usuário abre a URL no Chrome/Edge]) --> B[Shell Angular web carrega, Angular service worker cacheia]
    B --> C[Islands montam os componentes nativos]
    C --> D[Banco local IndexedDB inicializa]
    D --> E{Storage web OK?}
    E -->|Não| F[Tela de erro padrão + log no console]
    E -->|Sim| G[Início]
    G --> H[Navegador oferece Instalar app]
    H --> I[Janela própria, ícone no Windows]
    I --> J{Sem internet?}
    J -->|Sim| K[Abre do cache, dados do IndexedDB]
    J -->|Não| L[App normal]
    K --> M([Funcional offline])
    L --> M
```

## Arquitetura Macro

```mermaid
graph LR
    subgraph Alvos
        AND([Android APK - Expo/Fabric])
        WEB([PWA - Chrome/Edge/Windows])
    end

    subgraph Shared["Código compartilhado (libs TypeScript)"]
        DOM[domain]
        APP[application: casos de uso + ports]
    end

    subgraph Native["apps/native (Angular + NG Native)"]
        UIN[ui: componentes nativos + Angular Router nativo]
        STN[NgRx Store - UI, sessão, eventos]
        INN[infrastructure native]
    end

    subgraph Web["apps/web (Angular + Vite)"]
        UIW[Angular Router web + islands]
        STW[NgRx Store]
        INW[infrastructure web]
        SW[Angular service worker + manifest]
    end

    subgraph NativeAdapters["Adapters nativos"]
        DB[(Database - expo-sqlite)]
        SEC[SecureStorage / Storage]
        LOG[Logger: arquivo + console]
        NOT[notifications + background-task]
        HTTP[HttpClient nativo]
    end

    subgraph WebAdapters["Adapters web"]
        DBW[(IndexedDB)]
        SECW[web storage]
        HTTPW[HttpClient web]
    end

    BE[Backend FastAPI - inalterado]
    EXT[APIs externas]

    AND --> UIN
    WEB --> UIW
    UIN --> STN --> APP
    UIW --> STW --> APP
    UIN --> APP
    UIW --> APP
    APP --> DOM
    APP --> INN
    APP --> INW
    INN --> DB
    INN --> SEC
    INN --> LOG
    INN --> NOT
    INN --> HTTP
    INW --> DBW
    INW --> SECW
    INW --> HTTPW
    UIW --> SW
    HTTP --> BE
    HTTPW --> BE
    HTTP --> EXT

    subgraph Docker
        BLD[build: Node 22 + JDK17 + Android SDK]
        E2EA[e2e Android: emulador + Maestro]
        E2EW[e2e web: Playwright]
    end
    BLD -.gera.-> AND
    E2EA -.testa.-> AND
    E2EW -.testa.-> WEB
```

## Funcionalidades por Área

| Funcionalidade | Ator | Prioridade | Complexidade |
|---|---|---|---|
| Workspace Nx + app nativo (NG Native) + shell web + libs compartilhadas | Dev | Alta | G |
| TypeScript strict, ESLint/Prettier, fronteiras de camadas | Dev | Alta | M |
| Angular Router nativo com 4 abas e ícone ⚙ como gatilho de sheet | Usuário | Alta | M |
| Tema PlantumHibis light/dark (Tailwind + CSS vars + ColorScheme) | Usuário | Alta | M |
| Ícones ng-icons Lucide | Usuário | Média | P |
| BottomSheet próprio (`modal` + animação) e sheet de Configurações | Usuário | Alta | G |
| NgRx Store (UI/sessão/eventos) + regra "sem entidades" | Dev | Alta | M |
| Database (expo-sqlite) com migrations e colunas base | Dev | Alta | G |
| Repositórios reativos por signal com invalidação | Dev | Alta | G |
| Adapter IndexedDB atrás do mesmo port (web) | Dev | Alta | G |
| HttpClient nativo + interceptor `X-Device-Id` + erros tipados | Dev | Alta | M |
| SecureStorage / Storage e equivalentes web atrás de port | Dev | Média | M |
| Logger em arquivo (Android) + console | Dev | Alta | M |
| ErrorHandler global + tela de erro + toast padrão | Usuário | Alta | M |
| Base de notificações (canal, toast global, background task) | Dev | Média | M |
| PWA: shell Angular web, islands, service worker, manifest | Usuário | Alta | G |
| Docker build do APK (JDK 17 + Android SDK) | Dev | Alta | G |
| Docker E2E Android (emulador + Maestro) | Dev | Alta | G |
| Docker E2E web (Playwright) | Dev | Média | M |
| Vitest + @ng-native/testing + teste de arquitetura | Dev | Alta | M |
| .gitignore, .graphifyignore, README, CLAUDE/AGENTS/GEMINI, graphify | Dev | Alta | P |

## Viabilidade

- [x] A solução resolve o problema declarado
- [x] Não duplica funcionalidade já existente (repo novo; MAUI e RN seguem como referência)
- [x] Stack definida (Angular 22, NG Native, Expo SDK 57, RN 0.86, NgRx, Vitest, Maestro, Playwright)
- [x] Escopo é delimitável (esqueleto sem negócio; módulos em changes sequenciais)
- [ ] NG Native alpha sustenta o esqueleto (Router, tabs, modal, Database, ícones) em Android real
- [ ] NgRx funciona com o renderer nativo e `provideNativeHttpClient()`
- [ ] Emulador Android em Docker roda neste ambiente (KVM; WSL2 com virtualização aninhada)
- [ ] Islands renderizam o shell web sem depender do router nativo
- [ ] Node 22.18+, JDK 17 e SDK Android compatíveis com Expo SDK 57
- [ ] Estimativa de esforço aceitável (esqueleto maior que o do RN: dois apps + libs)

## Riscos Iniciais

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| NG Native é alpha (APIs 0.x mudam, bugs, poucos contribuidores) | Alta | Alto | Spikes antes de construir; versões fixas; isolar o framework atrás de ports; projeto tratado como avaliação paralela ao RN |
| Sem BottomSheet e sem ErrorBoundary prontos | Alta | Médio | BottomSheet próprio (`modal` + Reanimated); `ErrorHandler` do Angular + tela de erro por signal |
| Sem live query no módulo Database | Alta | Alto | Repositórios com signal de invalidação disparada nas escritas; teste de reatividade por tela; avaliar `addDatabaseChangeListener` do expo-sqlite |
| Router nativo não existe na web | Alta | Alto | Shell web Angular padrão + islands; compartilhar só domain/application/estado |
| Compatibilidade NgRx/bibliotecas Angular com o renderer não documentada | Média | Alto | Spike; fallback: store próprio com signals mantendo as mesmas regras |
| Emulador Android em Docker sem KVM no Windows | Alta | Alto | WSL2 com nested virtualization; plano B runner Linux |
| Expo SDK 57 / RN 0.86 mais novos que o resto do ecossistema (módulos sem suporte) | Média | Médio | Fixar versões do template; verificar cada módulo no spike |
| Esqueleto grande (2 apps + libs + Docker) | Média | Médio | Tasks granuladas, commit por grupo, spikes primeiro |

## Fora de Escopo

- Telas e regras de negócio: auth, casas, áreas, plantas, tarefas, alertas, perfil, sync, dashboard (changes 2 a 10).
- Sync com o Backend (só o cliente HTTP base).
- Lógica de alertas e permissão de notificação (só a base).
- Hospedagem/deploy do PWA.
- iOS e builds nativos Windows.
- Alterações no Backend FastAPI, no repo MAUI e no repo React Native.
- Toggle manual de tema.
- i18n completa (suporte parcial no NG Native), `@angular/animations` e SSR/hidratação (não suportados).
- Publicação na Play Store e assinatura de release.
