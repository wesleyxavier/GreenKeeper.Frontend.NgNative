## ADDED Requirements

### Requirement: Logger em arquivo e console
O app SHALL ter um logger com níveis (debug, info, warn, error) que escreve no console e, no Android, em arquivo rotativo no diretório do app (módulo `@ng-native/expo/file-system`), legível via `adb run-as`. Na web SHALL escrever só no console.

#### Scenario: Log no Android
- **WHEN** o app registra um erro
- **THEN** a linha aparece no console e no arquivo de log do app

#### Scenario: Rotação
- **WHEN** o arquivo de log passa do limite configurado
- **THEN** um novo arquivo é iniciado e os mais antigos além do limite são removidos

#### Scenario: Sem dados sensíveis
- **WHEN** uma requisição autenticada é logada
- **THEN** o token e demais segredos não aparecem no log

### Requirement: ErrorHandler global e tela de erro
O app SHALL substituir o `ErrorHandler` do Angular por um que loga o erro e aciona, por signal, a tela de erro padrão com opção "Tentar de novo" (o NG Native não tem ErrorBoundary); "Tentar de novo" reinicia a navegação e a árvore de telas.

#### Scenario: Erro de renderização ou de ação
- **WHEN** um componente ou handler lança um erro não tratado
- **THEN** a tela de erro padrão é exibida, o erro é logado e o app não fecha

#### Scenario: Tentar de novo
- **WHEN** o usuário toca em "Tentar de novo"
- **THEN** a árvore remonta e o app volta ao boot

### Requirement: Toast padrão de erro
O app SHALL oferecer um toast global para erros não fatais, acionável por qualquer camada via serviço/evento, com mensagem amigável.

#### Scenario: Erro não fatal
- **WHEN** uma chamada remota falha e o caso de uso emite o evento de erro
- **THEN** o toast é exibido sobre a tela atual e some após alguns segundos
