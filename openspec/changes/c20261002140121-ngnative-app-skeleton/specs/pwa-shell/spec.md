## ADDED Requirements

### Requirement: Shell web Angular com islands
O PWA SHALL ser um app Angular web (`apps/web`, Vite com `ngNativeWeb()`) com Angular Router web, que monta os componentes nativos de tela como **islands** (`ng-native-island`), sem usar o router nativo (que não tem versão web).

#### Scenario: Navegação web
- **WHEN** o usuário navega entre Início, Tarefas, Plantas e Alertas no navegador
- **THEN** a rota web muda e cada tela é montada por uma island com seu título

#### Scenario: Reuso de camadas
- **WHEN** o shell web compila
- **THEN** usa `libs/domain` e `libs/application` do mesmo workspace

### Requirement: PWA instalável
A build web SHALL ter manifest (nome PlantumHibis, ícones 192/512 e maskable, `display: standalone`, cores do tema) e `@angular/service-worker` configurado (`ngsw-config.json`), atendendo aos critérios de instalação do Chrome e do Edge.

#### Scenario: Instalação
- **WHEN** o usuário abre o PWA no Chrome ou Edge
- **THEN** o navegador oferece "Instalar app" e, após instalar, o app abre em janela própria

### Requirement: Abertura offline
Depois da primeira visita, o PWA SHALL abrir sem rede a partir do cache do service worker e usar o banco local IndexedDB.

#### Scenario: Sem internet
- **WHEN** o usuário abre o PWA instalado sem conexão
- **THEN** a aba Início é exibida e a navegação entre abas funciona

### Requirement: Mesmo footer, abas, tema e sheet
O PWA SHALL exibir o mesmo footer, as mesmas abas, o mesmo tema e o mesmo bottom sheet do app nativo, seguindo `prefers-color-scheme`.

#### Scenario: Tema escuro no navegador
- **WHEN** o navegador está em preferência escura
- **THEN** o PWA usa os tokens do tema escuro

### Requirement: Recursos nativos desligados na web
Notificações push, background task e demais recursos sem suporte na web SHALL ficar explicitamente desligados, sem lançar erros.

#### Scenario: Web sem background task
- **WHEN** o shell web inicia
- **THEN** nenhuma background task é registrada e nenhum erro é lançado
