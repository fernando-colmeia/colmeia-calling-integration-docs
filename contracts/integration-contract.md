# Contrato de Integração Externa

## 1. Objetivo

Este documento define o contrato externo de integração da **Colmeia Calling API**, descrevendo o modelo de comunicação, a sequência lógica de eventos e as garantias operacionais observáveis por clientes integradores.

O objetivo é estabelecer expectativas claras sobre comportamento, ordem de eventos, latência e responsabilidades em chamadas de voz *near-realtime*, sem expor detalhes de implementação interna.

## 2. Escopo

- Modelo de integração em alto nível  
- Papéis e responsabilidades das partes  
- Sequência obrigatória de eventos de chamada  
- Processamento de sinalização e mídia (visão externa)  
- Garantias e limitações operacionais  

## 3. Componentes

| Componente | Descrição |
|----------|----------|
| Cliente | Sistema integrador que consome a Colmeia Calling API |
| Colmeia Calling API | Plataforma intermediária de sinalização e, opcionalmente, de mídia |
| Origem da Comunicação | Sistema responsável por iniciar sessões de comunicação e emitir eventos iniciais |

**Notas:**

- A Origem da Comunicação pode ser um sistema externo ou a própria Colmeia, dependendo do canal e do fluxo de comunicação.  
- O cliente interage exclusivamente com a Colmeia Calling API.  

## 4. Visão Geral do Modelo

- A Colmeia emite eventos de chamada ao cliente  
- O cliente reage a esses eventos por meio das APIs expostas  
- A Colmeia atua como ponto intermediário de sinalização e, se configurado, também de mídia  
- A Colmeia participa do caminho crítico da chamada enquanto ela estiver ativa  

## 5. Sequência de Eventos de Chamada (Contrato)

Para uma mesma chamada (`idCall`), os eventos seguem uma sequência lógica obrigatória.  
Essa sequência define pré-condições contratuais para cada ação do cliente.

### Sequência Invariável

``` bot_req_call → connect → accept ```


### Definição dos Eventos

**bot_req_call**

- Evento inicial de solicitação de chamada  
- Indica que o cliente está autorizado a iniciar o processo de chamada  
- Nenhuma ação de aceitação ou mídia é válida antes deste evento  

**connect**

- Indica que o contexto de chamada foi estabelecido  
- A chamada está pronta para ser aceita  
- O cliente não deve enviar `accept` antes deste evento  

**accept**

- Enviado pelo cliente para aceitar a chamada  
- Deve ser enviado exclusivamente após o evento `connect`  
- Deve conter o SDP *answer* necessário para o estabelecimento da mídia  

### Regras de Validade

- Eventos enviados fora da sequência definida:
  - podem ser rejeitados  
  - podem ser ignorados  
  - podem resultar em erros  

- A Colmeia não garante processamento de ações fora das pré-condições definidas  

## 6. Processamento de Mídia (Visão Externa)

- A Colmeia atua como endpoint intermediário de mídia  
- O fluxo de áudio passa pela Colmeia durante a chamada  
- A Colmeia encaminha mídia entre as partes envolvidas  
- A Colmeia participa do caminho crítico da chamada  

A Colmeia não origina chamadas e não mantém mídia ativa após o encerramento da chamada.

## 7. Near-Realtime e Latência

- Eventos e mídia são processados em *near-realtime*  
- A latência observada depende de:
  - condições de rede  
  - carga da plataforma  
  - comportamento do cliente  

- Eventos são tipicamente propagados em centenas de milissegundos  
- Não há garantia de *hard real-time* nem de latência máxima  

## 8. Entrega de Eventos

- Eventos são entregues de forma assíncrona  
- Eventos podem:
  - ser duplicados  
  - sofrer tentativas de reentrega  

**Obrigatório:**  
Clientes devem tratar eventos de forma **idempotente**.

A sequência lógica por chamada é preservada conforme definido neste contrato.

## 9. Eventos de Chamada

Estados propagados pela API:

- `bot_req_call`  
- `connect`  
- `accept`  
- `reject`  
- `hangup`  

A disponibilidade e a transição entre estados seguem o comportamento da plataforma.

## 10. Encerramento e Falhas

Uma chamada pode ser encerrada por:

- Ação do usuário  
- Timeout  
- Falha técnica  

Após o encerramento:

- O fluxo de mídia é finalizado  
- Um evento final é emitido ao cliente  
- A chamada não pode ser retomada  

## 11. Limitações Conhecidas

- Não há garantia de ordenação absoluta entre chamadas distintas  
- Não há garantia de latência máxima  
- Eventos podem ser perdidos em cenários extremos  

## 12. Responsabilidades

| Parte | Responsabilidade |
|------|------------------|
| Cliente | Consumo correto de eventos, envio de ações válidas, idempotência |
| Colmeia | Emissão de eventos, encaminhamento de mídia, aplicação das regras de sequência |

## 13. Considerações Finais

Este documento define o contrato externo de integração da Colmeia Calling API.

Alterações internas que não modifiquem os comportamentos descritos aqui não exigem atualização deste documento.
