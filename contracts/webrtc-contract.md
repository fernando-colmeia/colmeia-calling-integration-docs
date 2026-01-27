# Especificação WebRTC e Mídia com Opção de Proxy

## 1. Visão Geral

A **Colmeia Calling API** utiliza WebRTC para encaminhamento de áudio em chamadas de voz, atuando como **endpoint intermediário** entre o cliente e a plataforma de comunicação que origina a chamada.

## 2. ICE e Sinalização

- **Modo ICE:** Full  
- **Trickle ICE:** desabilitado (*halt-trickle*)  
- Todos os candidatos são negociados na **oferta/resposta inicial**  
- O cliente **não deve enviar candidatos adicionais** após a conexão  
- Negociação de sessão ocorre **uma única vez por chamada**  
- **Sem suporte** a renegociação de mídia ou *ICE restart*  
- Mudanças de parâmetros **exigem nova chamada**

## 3. Codec de Áudio

- **Codec:** Opus (fixo e obrigatório)  
- Outros codecs **não são aceitos**  
- Garante interoperabilidade, baixa latência e compatibilidade com pipelines de voz/IA

## 4. Sample Rate e Bitrate

- **Sample rate:** configurável na plataforma Colmeia  
  - **8 kHz:** indicado para telefonia tradicional, VoIP básico e cenários de baixa largura de banda
  - **16 kHz:** indicado para pipelines de IA e processamento de voz em tempo real
- **Imutável durante a chamada**; alterações exigem nova chamada  
- **Bitrate:** negociado automaticamente via SDP  
- Não exposto nem configurável pelo cliente

## 5. Fluxo de Mídia

- Áudio é encaminhado continuamente enquanto a chamada estiver ativa
- Interrupção do fluxo de mídia **implica encerramento da chamada**

## 6. Latência e Comportamento Near-Realtime

- Sistema opera em **near-realtime**  
- Latência fim-a-fim depende da plataforma de origem, Colmeia, cliente e rede  
- **Sem garantia** de *hard real-time* ou latência máxima absoluta

## 7. Responsabilidades

| Camada | Responsabilidade |
|------|------------------|
| Cliente | Implementar WebRTC compatível, respeitar modelo ICE e parâmetros negociados |
| Colmeia | Estabelecer sessão WebRTC, encaminhar mídia e manter estabilidade durante a chamada |
| Plataforma de comunicação | Originação e encerramento da chamada |

## 8. Observações

- Alterações internas que não modifiquem o comportamento descrito **não exigem atualização** desta especificação  
