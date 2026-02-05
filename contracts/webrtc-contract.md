# Especificação WebRTC e Mídia com Opção de Proxy

### Nota Importante: Esta documentação é específica do nosso servidor WebRTC

Este documento descreve as especificações, configurações e comportamentos do **nosso servidor WebRTC**.

**Não** reflete necessariamente o comportamento exato da integração direta com provedores de terceiros, como a **Cloud API Calling do WhatsApp (Meta)**.

Para dúvidas técnicas específicas sobre o WebRTC da Meta (ex: requisitos de mídia, codecs suportados, ICE, DTLS, renegociação, etc.), consulte diretamente a documentação oficial do provedor:

- **Meta / WhatsApp Business API**  
  [Cloud API Calling | Perguntas frequentes sobre WebRTC/mídia](https://developers.facebook.com/documentation/business-messaging/whatsapp/calling/faq#webrtc--media-faq)

Usar a ColmeIA apenas como serviço de sinalização não altera as regras do provedor de mídia. Sempre valide o fluxo completo (sinalização + mídia) com a documentação do provedor escolhido.


## 1. Visão Geral

A **Colmeia Calling API** utiliza WebRTC para encaminhamento de áudio em chamadas de voz, atuando como **endpoint intermediário** entre o cliente e a plataforma de comunicação que origina a chamada.

## 2. ICE e Sinalização

- **Modo ICE:** Full  
- **Trickle ICE:** Desabilitado  
  - Todos os candidatos ICE são negociados exclusivamente na oferta/resposta inicial (dentro do SDP Offer/Answer)  
  - Não há envio ou processamento de candidatos adicionais após o SDP inicial  
- O cliente deve incluir todos os candidatos no SDP inicial e **não enviar candidatos extras**  
- Negociação de sessão ocorre **uma única vez por chamada**  
- **Sem suporte** a renegociação de mídia, ICE restart ou adição/remocão dinâmica de candidatos  
- Qualquer mudança de parâmetros (ex: adição de vídeo, mudança de codec) **exige nova chamada completa**

## 3. Codec de Áudio

- **Codec:** Opus (fixo e obrigatório)  
- Outros codecs **não são aceitos**  
- Garante interoperabilidade, baixa latência e compatibilidade com pipelines de voz/IA

## 4. Características de Áudio (Sample Rate, Bitrate e Comportamento)

### Sample Rate

- O **sample rate é configurável na plataforma Colmeia**, antes do início da chamada  
- Valores suportados:
  - **8 kHz:** indicado para telefonia tradicional, VoIP básico e cenários de baixa largura de banda
  - **16 kHz:** indicado para pipelines de IA e processamento de voz em tempo real
  - **48 kHz:** indicado para pipelines de IA e cenários de maior fidelidade de áudio
- O sample rate **é fixo durante a chamada**
- Essa configuração não aumenta a qualidade se a origem dos dados estiver num valor abaixo do configurado, apenas nivela.

---

### Bitrate

- O bitrate de áudio é **ajustado automaticamente**
- Não há valor fixo, mínimo ou máximo garantido
- O cliente **não pode configurar ou forçar** o bitrate
- O consumo de banda varia conforme:
  - conteúdo de áudio
  - perfil de frequência configurado
  - condições de rede

---

### Packetization Time (ptime)

- O áudio é transmitido em **quadros de duração fixa**, compatíveis com comunicações de voz em tempo quase real
- A duração dos quadros é **aproximadamente 20 ms**
- Esse comportamento:
  - não é negociável
  - não é configurável pelo cliente
  - não pode ser alterado durante a chamada

---

### Transmissão em Silêncio (DTX)

- A plataforma **não utiliza Discontinuous Transmission (DTX)**
- O áudio é transmitido continuamente, inclusive durante períodos de silêncio
- Não há supressão automática de envio de pacotes
- Esse comportamento garante fluxo contínuo e previsível de mídia

---

### Recuperação de Perda de Pacotes (FEC)

- A plataforma **não utiliza Forward Error Correction (FEC)**
- Não há redundância de pacotes para recuperação de perdas
- Em caso de perda de pacotes, aplicam-se apenas mecanismos padrão de ocultação de perdas

---

### Canais de Áudio (Mono)

- O áudio é transmitido **exclusivamente em mono**
- Áudio estéreo **não é suportado**
- Esse comportamento é fixo e não configurável

## 5. Fluxo de Mídia

- Áudio é encaminhado continuamente enquanto a chamada estiver ativa  
- **RTP/RTCP Ports:** Range 50000–60000  
- **RTCP-mux:** Ativado (RTCP multiplexado no mesmo porto RTP)
- **DTLS/SRTP:** Obrigatório (a=setup:actpass; fingerprint sha-256)  
- **Transport:** UDP/TLS/RTP/SAVPF (padrão seguro WebRTC)

## 6. Responsabilidades

| Camada | Responsabilidade |
|------|------------------|
| Cliente | Implementar WebRTC compatível, respeitar modelo ICE e parâmetros negociados |
| Colmeia | Estabelecer sessão WebRTC, encaminhar mídia e manter estabilidade durante a chamada |
| Plataforma de comunicação | Originação e encerramento da chamada |

## 7. Exemplo de SDP Offer Colmeia

```bash
v=0
o=- 1770293065220059 1 IN IP4 127.0.0.0
s=ColmeiaAudioSession-tlfdHAKXyTrzAYftmSpsxRGdh3PBRZZN
t=0 0
a=group:BUNDLE 0
a=fingerprint:sha-256 1A:6F:B3:48:BB:AA:C6:56:42:07:E3:55:72:D4:AF:27:8B:79:4D:20:48:0B:F0:96:0F:38:07:BF:89:13:A1:4E
a=extmap-allow-mixed
a=msid-semantic: WMS *
m=audio 9 UDP/TLS/RTP/SAVPF 111
c=IN IP4 127.0.0.0
a=sendrecv
a=mid:0
a=rtcp-mux
a=ice-ufrag:kymS
a=ice-pwd:mFMjZkYYBEbHN8uScVuQiZ
a=setup:actpass
a=rtpmap:111 opus/48000/2
a=rtcp-fb:111 transport-cc
a=extmap:1 urn:ietf:params:rtp-hdrext:sdes:mid
a=extmap:2 urn:ietf:params:rtp-hdrext:ssrc-audio-level
a=fmtp:111 0 maxplaybackrate=8000; stereo=0; sprop-stereo=0; useinbandfec=0
a=msid:ColmeiaAudioStream cm-audio-1
a=ssrc:3107864622 cname:ColmeiaAudioStreamSync
a=candidate:1 1 udp 2015363327 10.0.1.23 52314 typ host
a=candidate:2 1 udp 2015363327 18.230.50.12 52314 typ srflx
a=candidate:3 1 udp 2015363327 3.221.101.55 52314 typ srflx
a=candidate:4 1 udp 2015363327 54.202.33.11 52314 typ srflx
a=end-of-candidates
```

## 8. Observações

- Alterações internas que não modifiquem o comportamento descrito **não exigem atualização** desta especificação  
