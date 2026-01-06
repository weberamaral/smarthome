# Manifesto da Arquitetura de Casa Inteligente
**Plataforma principal: SmartThings**

Versão: **1.1**  
Status: **Vivo**  
Última atualização: **2026-01-04**

---

## 1. Propósito do Manifesto

Este documento define a **arquitetura alvo da casa inteligente**, estabelecendo princípios, decisões técnicas e responsabilidades claras entre plataformas, hubs e integrações.

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
   Atuadores (ação contínua) e sensores sleepy (evento pontual) possuem características operacionais distintas e devem seguir caminhos diferentes.

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

### SmartThings
- Plataforma principal de automação e rotinas
- Integra dispositivos via Matter e integrações diretas
- Orquestra cenas e automações
- **Não deve** controlar dispositivos que já possuem outro “dono” definido

### Hub Nova Digital (Tuya)
- Bridge Zigbee → Matter para **atuadores**
- Fonte de verdade para tomadas, interruptores, lâmpadas e fechaduras Zigbee Tuya
- **Proibido** para sensores sleepy

### Home Assistant (Supervised)
- Camada de integração local
- Hospeda Zigbee2MQTT, MQTT e LocalTuya
- Observabilidade, troubleshooting e bridges especializadas
- **Não é** a plataforma principal de automação da casa

### Zigbee2MQTT
- Controle exclusivo de sensores sleepy e cortinas a bateria
- Fonte de verdade Zigbee para esses dispositivos

### MQTT
- Backbone de eventos desacoplado
- Responsável pela comunicação entre sensores e bridges

### Bridge Matter (ex.: Matterbridge)
- Exposição de sensores do domínio MQTT/Z2M ao SmartThings
- Opera exclusivamente em LAN com mDNS/multicast funcional

---

## 5. Canais de Integração no SmartThings

### Canal 1 — Matter via Hub Nova Digital
- Atuadores Zigbee Tuya
- Comunicação Matter local
- Alta interoperabilidade

### Canal 2 — Matter via Bridge (Z2M → MQTT → Bridge)
- Sensores de porta, presença e cortinas a bateria
- Evita dependência de bridges cloud instáveis

### Canal 3 — Integrações diretas no SmartThings
- Matter over Wi-Fi
- Integrações não-Matter (Edge / Cloud)
- Classificação obrigatória:
  - **Local / Edge**
  - **Cloud-based**

---

## 6. Matriz de Decisão por Tipo de Dispositivo

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

## 7. Catálogo de Integrações Diretas no SmartThings

Registro vivo de dispositivos integrados diretamente no ST.

| Fabricante | Tipo | Canal | Natureza | Observações |
|-----------|------|-------|----------|-------------|
| Tapo | Tomada | ST direto | Cloud | Estável |
| — | — | — | — | — |

---

## 8. Decisões Arquiteturais (ADR-lite)

### ADR-01 — Sensores sleepy fora do Hub Nova Digital
**Decisão:** Sensores de porta, presença e cortinas a bateria não devem ser integrados via Hub Nova Digital.  
**Motivo:** Instabilidade pós-queda de energia.  
**Consequência:** Arquitetura mais complexa, porém mais resiliente.

---

### ADR-02 — MQTT como backbone de sensores
**Decisão:** Utilizar MQTT entre Zigbee2MQTT e Bridge Matter.  
**Motivo:** Desacoplamento, resiliência e flexibilidade.  
**Consequência:** Dependência de broker sempre disponível.

---

### ADR-03 — ST-first para integrações maduras
**Decisão:** Priorizar integração direta no SmartThings quando nativa e estável.  
**Motivo:** Simplicidade operacional.  
**Consequência:** Convivência com múltiplos canais de integração.

---

## 9. Runbook Operacional

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

## 10. Checklist Oficial — Integração de Novo Dispositivo

### 1. Identificação
- [ ] Tipo (atuador / sensor / cortina / outro)
- [ ] Alimentação (energia / bateria)
- [ ] Dispositivo crítico? (sim/não)

### 2. Integração nativa no SmartThings?
- [ ] Sim → integrar direto no ST
- [ ] Não → continuar avaliação

### 3. Suporte a Matter?
- [ ] Sim → avaliar ST direto ou Hub Nova Digital
- [ ] Não → continuar avaliação

### 4. Sensor sleepy ou a bateria?
- [ ] Sim → Z2M → MQTT → Bridge Matter
- [ ] Não → avaliar LocalTuya / integração local

### 5. Source of Truth definido?
- [ ] Plataforma dona definida
- [ ] Garantido que não será pareado em outro hub

### 6. Classificação da integração
- [ ] Local / Edge
- [ ] Cloud
- [ ] Dependência de internet aceita

### 7. Registro
- [ ] Atualizar Catálogo de Integrações
- [ ] Atualizar diagrama (se necessário)
- [ ] Registrar ADR se decisão for excepcional

---

## 11. Governança e Evolução

Este manifesto é um **documento vivo** e deve ser revisado:
- Após mudanças relevantes de infraestrutura
- Com a evolução do padrão Matter
- Quando falhas recorrentes indicarem desalinhamento arquitetural

Qualquer decisão que viole princípios ou contratos aqui definidos deve ser tratada como **quebra arquitetural** e registrada formalmente.

---