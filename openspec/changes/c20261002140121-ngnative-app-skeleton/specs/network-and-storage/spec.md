## ADDED Requirements

### Requirement: Cliente HTTP base
O app nativo SHALL usar `HttpClient` configurado por `provideNativeHttpClient()` (e o shell web por `provideHttpClient()`), com URL base por ambiente, timeout, tratamento de erro padronizado e interceptor que adiciona `X-Device-Id` a toda chamada ao backend.

#### Scenario: Header de device
- **WHEN** o cliente faz uma chamada ao backend
- **THEN** a requisição inclui `X-Device-Id` com o identificador estável do device

#### Scenario: Erro de rede
- **WHEN** a chamada falha por falta de conexão
- **THEN** o cliente lança um erro tipado de rede, que a UI converte em toast

#### Scenario: Timeout
- **WHEN** a chamada excede o timeout configurado
- **THEN** o cliente lança um erro tipado de timeout

### Requirement: Identificador estável de device
O app SHALL gerar um id de device uma vez e persisti-lo, retornando o mesmo valor nas execuções seguintes.

#### Scenario: Reabrir o app
- **WHEN** o app é fechado e reaberto
- **THEN** o id de device é o mesmo

### Requirement: Storage seguro e preferências atrás de ports
O token SHALL ser guardado pelo port de storage seguro (`SecureStorage`, expo-secure-store, no Android) e as preferências pelo port de preferências (`Storage`, AsyncStorage, no Android), com equivalentes web, sem que `ui` ou `application` conheçam a implementação. Como o `Storage` do NG Native descarta escritas na web, o shell web SHALL usar adapters próprios.

#### Scenario: Salvar e ler token
- **WHEN** um valor é salvo no storage seguro e lido em seguida
- **THEN** o valor lido é igual ao salvo, e após apagar a leitura retorna nulo

#### Scenario: Web não descarta escritas
- **WHEN** uma preferência é gravada no shell web e o app é recarregado
- **THEN** o valor persiste
