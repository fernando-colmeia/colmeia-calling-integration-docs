# Colmeia Calling API

Este documento descreve como integrar com a Colmeia Calling API.
A definição completa de contratos, eventos, enums, schemas e exemplos está disponível na especificação OpenAPI 3.1 (Swagger).

>**Nota:**<br>
O evento `connect` não inicia chamadas. Ele é enviado apenas via Webhook para notificar uma chamada recebida.

--------------------------------------------------

### Fluxo de Integração

A integração é orientada a eventos.

Diagrama do fluxo:<br>
→ [`sequence-diagrams/call-api-integration-flow.png`](sequence-diagrams/call-api-integration-flow.png)

##### Resumo do ciclo de vida da chamada:
1. Bot solicita chamada (`bot_req_call`)
2. Plataforma notifica chamada recebida (`connect`)
3. Cliente aceita ou rejeita (`accept` / `reject`)
4. Chamada é encerrada (`hangup`)

--------------------------------------------------

### Contratos e Eventos

Todos os contratos, eventos, enums e schemas estão definidos exclusivamente no OpenAPI:

→ [`calling-integration-api-oas.yaml`](calling-integration-api-oas.yaml)


--------------------------------------------------

### WebRTC

Regras e restrições específicas de WebRTC (SDP, ICE, codecs):

→ [`contracts/webrtc-contract.md`](contracts/webrtc-contract.md)

A estrutura SessionDescription referenciada no OAS segue a RFC 8866.

--------------------------------------------------

### Endpoints

Base URL por ambiente:

- https://dev-api.colmeia.cx/v1/rest
- https://beta-api.colmeia.cx/v1/rest
- https://api.colmeia.cx/v1/rest

Definição completa de endpoints, métodos, parâmetros, exemplos e respostas:

→ [`calling-integration-api-oas.yaml`](calling-integration-api-oas.yaml)

--------------------------------------------------

### Webhooks

Eventos entregues via Webhook estão documentados no OpenAPI:

→ [`calling-integration-api-oas.yaml`](calling-integration-api-oas.yaml)

##### O cliente DEVE:
- Responder rapidamente com **HTTP 200**
- Não bloquear o processamento do Webhook

--------------------------------------------------

### Boas Práticas

1. Persistir `idCall` e `idConversation`
2. Tratar o OpenAPI como fonte única de contrato
3. Não modificar SDP sem necessidade
4. Usar `botHalt` apenas quando necessário

--------------------------------------------------

### Referências

- OpenAPI (OAS 3.1): [`calling-integration-api-oas.yaml`](calling-integration-api-oas.yaml)
- WebRTC: [`contracts/webrtc-contract.md`](contracts/webrtc-contract.md)
- Diagramas: [`sequence-diagrams`](sequence-diagrams)

--------------------------------------------------

