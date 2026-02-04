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

## 4. Características de Áudio (Sample Rate, Bitrate e Comportamento)

### Sample Rate

- O **sample rate é configurável na plataforma Colmeia**, antes do início da chamada  
- Valores suportados:
  - **8 kHz:** indicado para telefonia tradicional, VoIP básico e cenários de baixa largura de banda
  - **16 kHz:** indicado para pipelines de IA e processamento de voz em tempo real
  - **48 kHz:** indicado para pipelines de IA e cenários de maior fidelidade de áudio
- O sample rate **é fixo durante a chamada**

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

## 6. Responsabilidades

| Camada | Responsabilidade |
|------|------------------|
| Cliente | Implementar WebRTC compatível, respeitar modelo ICE e parâmetros negociados |
| Colmeia | Estabelecer sessão WebRTC, encaminhar mídia e manter estabilidade durante a chamada |
| Plataforma de comunicação | Originação e encerramento da chamada |

## 7. Observações

- Alterações internas que não modifiquem o comportamento descrito **não exigem atualização** desta especificação  
