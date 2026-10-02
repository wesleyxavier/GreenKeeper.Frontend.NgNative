## ADDED Requirements

### Requirement: Banco local com migrations versionadas
O app nativo SHALL usar o módulo `@ng-native/expo/database` (expo-sqlite) e aplicar as migrations pendentes na inicialização, por versão (`PRAGMA user_version`), cada uma em transação, antes de renderizar as abas.

#### Scenario: Primeiro boot
- **WHEN** o app inicia sem banco existente
- **THEN** as migrations rodam em ordem, o schema é criado e as abas são exibidas

#### Scenario: Falha de migration
- **WHEN** uma migration falha na inicialização
- **THEN** a migration faz rollback, a versão não muda, a tela de erro padrão é exibida, o erro é logado e o app não fecha

#### Scenario: Banco já atualizado
- **WHEN** o app inicia com o schema na versão mais recente
- **THEN** nenhuma migration executa

### Requirement: Colunas base de sincronização
Toda tabela de domínio SHALL ter `id`, `version`, `syncStatus` e `deletedAt` (soft delete), definidas por um helper de schema compartilhado.

#### Scenario: Registro novo
- **WHEN** uma entidade é criada localmente
- **THEN** `syncStatus` é `PendingSync`, `version` é 1 e `deletedAt` é nulo

### Requirement: Repositórios reativos por signal
Os repositórios SHALL expor listagens como signals que são recomputados quando as tabelas observadas mudam, inclusive por escritas feitas fora da tela (invalidação disparada pelo repositório, ou pelo listener de mudança do expo-sqlite se o spike o validar).

#### Scenario: Escrita externa atualiza a lista
- **WHEN** um serviço insere um registro enquanto a lista está montada
- **THEN** a lista exibe o novo registro sem recarga manual

#### Scenario: Consulta com join
- **WHEN** a consulta usa join entre duas tabelas e uma delas muda
- **THEN** a lista é atualizada

#### Scenario: Soft delete
- **WHEN** `deletedAt` de um registro é preenchido
- **THEN** o registro some da lista

### Requirement: Adapter web atrás do mesmo port
Na web, o banco SHALL ser entregue por um adapter IndexedDB atrás do mesmo port de repositório, sem alterar componentes ou casos de uso.

#### Scenario: Mesmo teste de contrato nos dois adapters
- **WHEN** a suíte de contrato de repositório roda contra o adapter nativo e o adapter web
- **THEN** ambos passam com o mesmo comportamento
