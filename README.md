# Manifesto da Arquitetura de Casa Inteligente
**Plataforma principal: SmartThings**

Versão: **1.2**  
Status: **Vivo**  
Última atualização: **2026-01-04**

---

## 1. Propósito do Manifesto

Este documento define a **arquitetura alvo da casa inteligente**, estabelecendo princípios, decisões técnicas e **contratos arquiteturais** claros entre plataformas, hubs e integrações.

Seus objetivos são:
- Garantir **estabilidade operacional**, especialmente após quedas de energia
- Evitar **duplicidade de controle** e estados inconsistentes
- Servir como **referência viva** para evolução futura
- Reduzir decisões improvisadas ao integrar novos dispositivos

Este manifesto **não** tem como objetivo:
- Centralizar toda a automação em uma única plataforma
- Eliminar integrações cloud quando estas forem maduras e estáveis
- Padronizar marcas ou fabricantes específicos

---

## 2. Princípios Arquiteturais

1. **ST-first (SmartThings como plataforma principal)**  
   Sempre que existir uma integração nativa, estável e suportada pelo SmartThings, o dispositivo deve ser integrado diretamente ao ST.

2. **Single Source of Truth por dispositivo**  
   Cada dispositivo deve possuir **um único dono operacional**, evitando controle simultâneo por múltiplas plataformas.

3. **Separação por natureza do dispositivo**  
   Atuadores (ação contínua) e sensores sleepy (evento pontual) possuem características operacionais distintas.

4. **Sensores sleepy não dependem de bridges cloud instáveis**  
   Sensores de porta, presença e cortinas a bateria devem evitar bridges conhecidas por instabilidade pós-reboot.

5. **Preferência por integrações locais ou edge para funções críticas**  
   Funções críticas (segurança, acesso, presença) devem priorizar integrações locais sempre que possível.

6. **Matter como padrão de interoperabilidade, não como obrigação universal**  
   Matter é adotado quando agrega interoperabilidade e estabilidade, não como requisito absoluto.

7. **Arquitetura resiliente a reboot e falhas de conectividade**  
   O ambiente deve se recuperar de forma previsível após quedas de energia ou rede.

---

## 3. Visão Geral da Arquitetura

A arquitetura é organizada em **canais de integração convergindo no SmartThings**, respeitando o tipo de dispositivo, sua criticidade e comportamento operacional.

### Canais principais

1. **Matter via Hub Nova Digital**  
   Atuadores Zigbee Tuya (tomadas, interruptores, lâmpadas, fechaduras).

2. **Matter via Bridge (Zigbee2MQTT → MQTT → Bridge Matter)**  
   Sensores de porta, presença e cortinas a bateria.

3. **Integrações diretas no SmartThings**  
   - Dispositivos Matter over Wi-Fi  
   - Dispositivos não-Matter com integração nativa (ex.: Tapo)

4. **Integrações locais via Home Assistant**  
   LocalTuya, MQTT, observabilidade e suporte técnico.

> 📌 O SmartThings atua como **plataforma principal de automação**, enquanto o Home Assistant atua como **camada de integração local e suporte arquitetural**.

---

## 4. Contrato Arquitetural — Papéis das Plataformas

### 4.1 SmartThings — Contrato de Responsabilidade e Controle

O SmartThings é definido como a **plataforma principal de orquestração da casa inteligente**, responsável por automações, rotinas e experiência do usuário.

Entretanto, o SmartThings **não é necessariamente o controlador primário de todos os dispositivos**.

#### Regra de Ouro — Source of Truth

> **O SmartThings não deve parear, controlar ou manter o estado primário de dispositivos que já possuam outro sistema definido como “Source of Truth” (fonte de verdade).**

Um dispositivo possui **Source of Truth** quando existe outro sistema responsável por:
- pareamento inicial
- manutenção do estado do dispositivo
- reconciliação após falhas (reboot, queda de energia, perda de rede)
- envio de comandos diretos ao dispositivo

Nesses casos, o SmartThings atua **exclusivamente** como:
- consumidor de eventos
- orquestrador de automações
- camada de integração entre domínios

#### Exemplos Práticos

| Dispositivo | Source of Truth | Papel do SmartThings |
|------------|----------------|----------------------|
| Sensor de porta Zigbee | Zigbee2MQTT | Consumir eventos |
| Sensor de presença | Zigbee2MQTT | Orquestrar automações |
| Cortina a bateria | Zigbee2MQTT | Orquestrar |
| Lâmpada Tuya Zigbee | Hub Nova Digital | Controlar via Matter |
| Tomada Tapo | SmartThings | Dono e controlador |
| Tuya Wi-Fi | LocalTuya (HA) | Consumidor opcional |

#### Ações Explicitamente Proibidas

Para preservar a integridade da arquitetura, o SmartThings **não deve**:
- Parear dispositivos Zigbee cujo Source of Truth seja o Zigbee2MQTT
- Controlar dispositivos já gerenciados por outro hub (Nova Digital, LocalTuya)
- Manter múltiplos caminhos de controle para o mesmo dispositivo
- Substituir o controlador primário definido neste contrato

Qualquer violação dessas regras é considerada **quebra arquitetural**.

#### Regra Mental de Validação

Antes de integrar um novo dispositivo ao SmartThings, deve-se responder:

> **“Quem é responsável por acordar, reconectar e reconciliar esse dispositivo após uma queda de energia?”**

Se a resposta **não for SmartThings**, então o SmartThings **não deve ser o Source of Truth** desse dispositivo.

---

### 4.2 Hub Nova Digital (Tuya)
- Bridge Zigbee → Matter para **atuadores**
- Fonte de verdade para tomadas, interruptores, lâmpadas e fechaduras Zigbee Tuya
- **Proibido** para sensores sleepy

### 4.3 Home Assistant (Supervised)
- Camada de integração local
- Hospeda Zigbee2MQTT, MQTT e LocalTuya
- Observabilidade, troubleshooting e bridges especializadas
- **Não é** a plataforma principal de automação da casa

### 4.4 Zigbee2MQTT
- Controle exclusivo de sensores sleepy e cortinas a bateria
- Fonte de verdade Zigbee para esses dispositivos

### 4.5 MQTT
- Backbone de eventos desacoplado

### 4.6 Bridge Matter (ex.: Matterbridge)
- Exposição de sensores do domínio MQTT/Z2M ao SmartThings
- Opera exclusivamente em LAN com mDNS/multicast funcional

---

## 5. Matriz de Decisão por Tipo de Dispositivo

| Tipo de Dispositivo | Canal Preferencial | Observações |
|--------------------|------------------|-------------|
| Tomadas | ST direto / Matter Nova Digital | Avaliar cloud vs local |
| Interruptores | Matter Nova Digital | Atuador contínuo |
| Lâmpadas | Matter Nova Digital | |
| Fechaduras | Matter Nova Digital | Dispositivo crítico |
| Sensores de porta | Z2M → MQTT → Bridge Matter | Sleepy |
| Sensores de presença | Z2M → MQTT → Bridge Matter | |
| Cortinas a bateria | Z2M → MQTT → Bridge Matter | |
| Tuya Wi-Fi | LocalTuya (HA) | Opcional expor ao ST |
| Tapo (exemplo) | ST direto | Integração cloud estável |

---

## 6. Catálogo de Integrações Diretas no SmartThings

| Fabricante | Tipo | Canal | Natureza | Observações |
|-----------|------|-------|----------|-------------|
| Tapo | Tomada | ST direto | Cloud | Estável |
| — | — | — | — | — |

---

## 7. Decisões Arquiteturais (ADR-lite)

### ADR-01 — Sensores sleepy fora do Hub Nova Digital
Motivo: instabilidade pós-queda de energia.

### ADR-02 — MQTT como backbone de sensores
Motivo: desacoplamento e resiliência.

### ADR-03 — ST-first para integrações maduras
Motivo: simplicidade operacional.

---

## 8. Runbook Operacional

### Ordem de boot pós-queda de energia
1. Roteador / Internet
2. Home Assistant + MQTT
3. Bridge Matter
4. Hub Nova Digital
5. SmartThings

### Regras de ouro
- Nunca parear o mesmo dispositivo em dois hubs
- Nunca misturar sensores sleepy em bridges cloud instáveis
- Validar mDNS/multicast após alterações de rede

---

## 9. Checklist Oficial — Integração de Novo Dispositivo

*(mantido conforme versão anterior)*

---

## 10. Governança e Evolução

Este manifesto é um **documento vivo** e deve ser revisado sempre que houver mudanças relevantes de infraestrutura ou integração.

Qualquer decisão que viole princípios ou contratos aqui definidos deve ser tratada como **quebra arquitetural** e registrada.

---