## ADDED Requirements

### Requirement: Store NgRx único
Cada app SHALL ter um único Store NgRx (`provideStore`, `provideEffects`) registrado nos providers, com um estado de exemplo de UI/sessão e selectors expostos como signals (`selectSignal`).

#### Scenario: Action chega ao reducer
- **WHEN** uma action do estado de exemplo é despachada
- **THEN** o estado reflete a mudança e o signal do selector retorna o novo valor

### Requirement: Store sem entidades de domínio
O Store SHALL guardar somente estado de UI/sessão e eventos, nunca entidades de domínio nem listas delas; o banco local SHALL ser a fonte de dados.

#### Scenario: Teste de reflexão sobre o estado
- **WHEN** um teste inspeciona o estado inicial de todas as features do Store
- **THEN** nenhum valor é uma entidade do `domain` nem array de entidades

### Requirement: Effects para eventos cross-cutting
O app SHALL reagir a actions por NgRx Effects, sem acoplar componentes a quem emite o evento.

#### Scenario: Evento dispara reação
- **WHEN** uma action observada por um effect é despachada
- **THEN** o effect executa uma vez

### Requirement: Compatibilidade validada com o renderer nativo
O uso do NgRx no app nativo SHALL ser validado em Android real (spike); se incompatível, o Store SHALL ser substituído por um store próprio baseado em signals mantendo as mesmas regras e API de selectors.

#### Scenario: Store no dispositivo
- **WHEN** o app nativo roda no emulador com o Store ativo
- **THEN** actions, reducers, selectors e effects funcionam sem erros no log

### Requirement: Dados locais por consulta reativa, não por estado global
Listagens locais SHALL vir de repositórios reativos (signal) do banco, e chamadas HTTP SHALL usar `HttpClient` com signals/`resource`, sem duplicar dados do banco no Store.

#### Scenario: Lista local não usa o Store
- **WHEN** uma tela lista dados do banco local
- **THEN** a fonte é o signal do repositório reativo, não um selector do Store
