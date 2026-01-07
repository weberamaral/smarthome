# Manifesto da Arquitetura de Casa Inteligente
**Plataforma principal: SmartThings**

Versão: **1.5**  
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
   Cada dispositivo deve possuir apenas **um único dono operacional**, evitando controle simultâneo por múltiplas plataformas.

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

### Diagrama da Arquitetura Atual

A figura abaixo representa a **arquitetura atual conforme este manifesto**, evidenciando:
- o SmartThings como núcleo de orquestração
- os diferentes canais de entrada (Matter, Bridge, integrações diretas)
- a separação clara de responsabilidades (Source of Truth)

![Arquitetura alvo da casa inteligente](./docs/arquitetura/arquitetura-alvo.png)

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

O SmartThings **não deve**:
- Parear dispositivos Zigbee cujo Source of Truth seja o Zigbee2MQTT
- Controlar dispositivos já gerenciados por outro hub (Nova Digital, LocalTuya)
- Manter múltiplos caminhos de controle para o mesmo dispositivo
- Substituir o controlador primário definido neste contrato

Qualquer violação dessas regras é considerada **quebra arquitetural**.

---

### 4.2 Hub Nova Digital (Tuya)
- Bridge Zigbee → Matter para **atuadores**
- Fonte de verdade para atuadores Tuya Zigbee
- **Proibido** para sensores sleepy

### 4.3 Home Assistant (Supervised)
- Camada de integração local
- Hospeda Zigbee2MQTT, MQTT e LocalTuya
- Observabilidade e bridges especializadas
- **Não é** a plataforma principal de automação

### 4.4 Zigbee2MQTT
- Controle exclusivo de sensores sleepy e cortinas a bateria

### 4.5 MQTT
- Backbone de eventos desacoplado

### 4.6 Bridge Matter
- Exposição de sensores Z2M/MQTT ao SmartThings
- Operação exclusiva em LAN com mDNS/multicast funcional

---

## 5. Matriz de Decisão por Tipo de Dispositivo

| Tipo | Canal Preferencial | Observações |
|----|------------------|-------------|
| Tomadas | ST direto / Matter ND | Avaliar cloud vs local |
| Interruptores | Matter ND | Atuador contínuo |
| Lâmpadas | Matter ND | |
| Fechaduras | Matter ND | Crítico |
| Sensores de porta | Z2M → MQTT → Bridge | Sleepy |
| Presença | Z2M → MQTT → Bridge | |
| Cortinas bateria | Z2M → MQTT → Bridge | |
| Tuya Wi-Fi | LocalTuya (HA) | Opcional |
| Tapo | ST direto | Cloud estável |

---

## 6. Catálogo de Integrações Diretas no SmartThings

### Objetivo do Capítulo

Este capítulo registra conscientemente todas as integrações realizadas **diretamente no SmartThings**, fora dos fluxos padronizados, garantindo controle arquitetural e rastreabilidade de decisões.

> 📌 Todo dispositivo integrado diretamente no SmartThings **deve constar neste catálogo**.

### Catálogo

| Fabricante | Tipo | Canal | Natureza | Observações |
|-----------|------|-------|----------|-------------|
| Tapo | Tomada | ST direto | Cloud | Estável |
| — | — | — | — | — |

---

## 7. Decisões Arquiteturais (ADR-lite)

- **ADR-01:** Sensores sleepy fora do Hub Nova Digital  
- **ADR-02:** MQTT como backbone de sensores  
- **ADR-03:** ST-first para integrações maduras  

---

## 8. Runbook Operacional

### Ordem de boot pós-queda de energia
1. Roteador / Internet  
2. Home Assistant + MQTT  
3. Bridge Matter  
4. Hub Nova Digital  
5. SmartThings  

### Regras de ouro
- Nunca parear um dispositivo em dois hubs
- Nunca misturar sensores sleepy em bridges cloud
- Validar mDNS/multicast após alterações de rede

---

## 9. Checklist Oficial — Integração de Novo Dispositivo

### 9.1 Identificação
- [ ] Tipo do dispositivo
- [ ] Atuador ou sensor sleepy
- [ ] Alimentação (energia / bateria)
- [ ] Dispositivo crítico? (sim/não)

### 9.2 Integração nativa no SmartThings?
- [ ] Sim → integrar direto no ST
- [ ] Não → continuar avaliação

### 9.3 Suporte a Matter?
- [ ] Sim → avaliar ST direto ou Hub Nova Digital
- [ ] Não → continuar avaliação

### 9.4 Sensor sleepy ou a bateria?
- [ ] Sim → Z2M → MQTT → Bridge Matter
- [ ] Não → avaliar LocalTuya / integração local

### 9.5 Source of Truth
- [ ] Dono definido
- [ ] Garantido que não será pareado em outro hub

### 9.6 Classificação da integração
- [ ] Local / Edge
- [ ] Cloud
- [ ] Dependência de internet aceita

### 9.7 Registro
- [ ] Atualizar catálogo
- [ ] Atualizar diagrama
- [ ] Registrar ADR se necessário

---

## 10. Governança e Evolução

Este manifesto é um **documento vivo** e deve ser revisado:
- Após mudanças relevantes de infraestrutura
- Com a evolução do padrão Matter
- Quando falhas recorrentes indicarem desalinhamento arquitetural

Decisões que violem este manifesto são consideradas **quebra arquitetural** e devem ser registradas.

---