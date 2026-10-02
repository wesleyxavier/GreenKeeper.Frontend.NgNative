## ADDED Requirements

### Requirement: Navegação por abas com Angular Router nativo
O app nativo SHALL usar o Angular Router nativo (`native-tabs-outlet`) com um footer de 5 ícones Lucide (sem rótulo visível, com `accessibilityLabel`) na ordem Início, Tarefas, Plantas, Alertas e Configurações. Início, Tarefas, Plantas e Alertas são rotas com título e estado vazio ("Em breve"); o ícone de Configurações é um gatilho que abre o sheet, sem rota própria.

#### Scenario: Troca de abas
- **WHEN** o usuário toca em Tarefas, Plantas e Alertas
- **THEN** a tela correspondente é exibida com seu título e a aba fica marcada como ativa

#### Scenario: Configurações não é rota
- **WHEN** o usuário toca no ícone de Configurações
- **THEN** o sheet abre e a rota atual não muda

#### Scenario: Aba inicial
- **WHEN** o app abre
- **THEN** a aba Início é exibida

#### Scenario: Botão voltar
- **WHEN** o usuário aciona o botão voltar do Android na aba Início sem sheet aberto
- **THEN** o comportamento padrão do sistema é aplicado e nenhum erro é lançado

### Requirement: Tema PlantumHibis light e dark
O app SHALL aplicar tokens de cor, espaçamento e tipografia do tema PlantumHibis via Tailwind e CSS variables, escolhendo light ou dark pelo signal `ColorScheme`, sem cores hardcoded nos componentes.

#### Scenario: Sistema em dark mode
- **WHEN** o sistema está em modo escuro
- **THEN** fundo e texto usam os tokens do tema escuro e o texto permanece legível

#### Scenario: Sistema em light mode
- **WHEN** o sistema está em modo claro
- **THEN** fundo e texto usam os tokens do tema claro

#### Scenario: Mudança em runtime
- **WHEN** o sistema alterna entre claro e escuro com o app aberto
- **THEN** o tema troca sem reiniciar

### Requirement: Ícones ng-icons Lucide
Os ícones SHALL ser renderizados por `@ng-native/icons` com `@ng-icons/lucide`, no Android e no shell web, recoloridos por `color` a partir dos tokens do tema.

#### Scenario: Ícones do footer
- **WHEN** o footer é exibido
- **THEN** cada item mostra seu ícone Lucide nos estados ativo e inativo

### Requirement: BottomSheet reutilizável
O app SHALL oferecer um componente BottomSheet próprio (o NG Native não fornece), baseado em `modal` com animação, que abre e fecha por gesto, botão, backdrop ou botão voltar do Android; o ícone de Configurações SHALL abri-lo, listando Perfil, Sincronização, Casas, Áreas e Sobre o App desabilitados com selo "em breve".

#### Scenario: Abrir e fechar
- **WHEN** o usuário toca no ícone de Configurações e depois fecha o sheet
- **THEN** o sheet abre e fecha sem trocar a aba de fundo

#### Scenario: Botão voltar com sheet aberto
- **WHEN** o sheet está aberto e o usuário aciona o botão voltar do Android
- **THEN** só o sheet fecha e o app continua aberto
