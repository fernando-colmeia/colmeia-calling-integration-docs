# Changelog

Todas as mudanças notáveis na Calling API serão documentadas neste arquivo.

O formato segue o [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/) e este projeto adota [Versionamento Semântico](https://semver.org/lang/pt-BR/).

## [1.1.0-beta] - 2026-02-11

### Added
- Campo opcional `callingTag` incluído nos eventos de webhook, definido na configuração da ação do bot e propagado no payload.
<br><br>
---

## [1.0.0-beta] - 2026-02-06

### Fixed
- Rejeição de chamadas: evento agora é corretamente enviado ao provedor.

### Changed
- Fluxo de rejeição: evento de hangup não é mais necessário em chamadas rejeitadas pela integração.  
  *(Comportamento anterior não estava documentado – desculpas pelo inconveniente)*

- Rotina de limpeza do estado “Halt” agora executada automaticamente em falhas ao processar eventos de conexão do provedor.

### Added
- Respostas de erro de validação documentadas na API REST (em respostas 400 Bad Request).
- Suporte a respostas HTTP para eventos e controle de concorrência:
  - **204 No Content** para eventos duplicados (mesmo evento do mesmo originador para a mesma chamada), garantindo processamento idempotente e evitando duplicidade.
  - **429 Too Many Requests** para controle de concorrência por chamada (limite de processamento simultâneo).

### Documentation
- Campo `idCallEvent` agora devidamente documentado no arquivo `calling-integration-api-oas.yaml`.
