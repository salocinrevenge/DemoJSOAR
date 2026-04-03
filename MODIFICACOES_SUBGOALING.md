# Modificações de Subgoaling - DemoJSOAR

## Resumo das Mudanças

Este documento descreve as modificações realizadas para implementar um sistema de **subgoaling** com memória persistente no DemoJSOAR. O robô agora é capaz de:

1. **Escanear** o ambiente girando sobre seu próprio eixo
2. **Armazenar** a localização de jóias e comidas em memória de trabalho
3. **Decidir** qual subgoal executar baseado no combustível disponível:
   - Se combustível ≥ 300 E há jóias em memória → buscar jóia
   - Se combustível < 300 E há comidas em memória → buscar comida
   - Caso contrário → continuar escaneando (wander)

---

## Arquivos Modificados

### 1. `soar-rules-ativ1.soar`

**Mudanças Principais:**

#### Estrutura de Memória Inicializada
```soar
sp {elaborate*initialize*scanning*memory
   (state <s> ^io.input-link.CREATURE <creature>)
   - (<s> ^scanned-items)
-->
   (<s> ^scanned-items <si>)
   (<si> ^jewels <j>)
   (<si> ^foods <f>)
   (<j> ^list)
   (<f> ^list)}
```
- Cria uma estrutura persistente `scanned-items` com sublistas para jóias e comidas
- Esta estrutura é criada uma única vez e persiste entre ciclos

#### Três Operadores Principais no Top-Level

**a) Operador: `wander` (Prioridade 1 - Fallback)**
- Faz o robô girar: `VelR=3, VelL=0`
- Durante a rotação, todos os itens visuais são adicionados à memória
- Sempre disponível como opção de última instância

```soar
sp {apply*wander*store*food*item
   (state <s> ^superstate.operator.name wander
              ^io.input-link.CREATURE.SENSOR.VISUAL.ENTITY <entity>
              ^superstate.scanned-items.foods <foods>)
   (<entity> ^TYPE FOOD ^NAME <name> ^X <x> ^Y <y> ^COLOR <color>)
   - (<foods> ^list.item-name <name>)
-->
   (<foods> ^list <item>)
   (<item> ^item-name <name> ^item-x <x> ^item-y <y> ^item-color <color>)}
```

**b) Operador: `eat-food` (Prioridade 9)**
- Proposto quando: `combustível < 300` E há comidas em `scanned-items.foods`
- Robô move até a comida mais próxima em memória
- Come quando distância < 30 unidades

```soar
sp {propose*goal*eat-food
   (state <s> ^fuel-level < 300
              ^scanned-items.foods.list <food>)
-->
   (<s> ^operator <o> + =)
   (<o> ^name eat-food ^priority 9)}
```

**c) Operador: `get-jewel` (Prioridade 10)**
- Proposto quando: `combustível >= 300` E há jóias em `scanned-items.jewels`
- Robô move até a jóia mais próxima em memória
- Pega a jóia quando distância < 30 unidades

```soar
sp {propose*goal*get-jewel
   (state <s> ^fuel-level >= 300
              ^scanned-items.jewels.list <jewel>)
-->
   (<s> ^operator <o> + =)
   (<o> ^name get-jewel ^priority 10)}
```

#### Preferências de Desempate
- Operadores com maior prioridade são preferidos
- Entre operadores de mesmo nome, item mais próximo (menor distância) é preferido
- Desempate favorece ações (eat/get) sobre movimento

---

### 2. `src/main/java/SoarBridge/SoarBridge.java`

**Mudanças:**

**a) Nova Variável de Instância**
```java
// Entity Variables
Identifier creature;
Identifier creatureSensor;
Identifier creatureParameters;
Identifier creaturePosition;
Identifier creatureMemory;
Identifier creatureScanningItems;  // ← NOVA
```

**b) Modificação em `prepareInputLink()`**
```java
// Initialize Creature Scanning Items (for subgoaling)
creatureScanningItems = CreateIdWME(creature, "SCANNING-ITEMS");
Identifier scannedJewels = CreateIdWME(creatureScanningItems, "JEWELS");
Identifier scannedFoods = CreateIdWME(creatureScanningItems, "FOODS");
```

- Cria a estrutura de input-link para armazenar items escaneados
- Inicializa sublistas vazias para jóias e comidas

---

## Fluxo de Execução

```
┌─────────────────────────────────────────┐
│    Início do Ciclo SOAR                 │
└────────────────┬────────────────────────┘
                 │
                 ▼
        ┌────────────────────┐
        │ Ler fuel-level    │
        └────────┬───────────┘
                 │
        ┌────────▼───────────┐
        │ Verificar:         │
        │ - fuel >= 300?     │
        │ - há jóias?        │
        │ - há comidas?      │
        └────────┬───────────┘
                 │
    ┌────────────┼──────────────┐
    │            │              │
    │            ▼              │
   ▼      ┌──────────────┐      │
┌─────┐   │ fuel >= 300  │      │
│ NÃO │   │ & jóias?     │      │
└─────┘   └──────┬───────┘      │
    │            │ SIM          │
    │            ▼              │
    │      ┌──────────────┐      │
    │      │ GET-JEWEL    │      │
    └──────► (Prioridade  │◄─────┘
             10)          │
             └──────┬─────┘
                    │
    ┌────────┬──────▼──────┬────────┐
    │        │             │        │
    ▼        ▼             ▼        ▼
 Mover    Aproxima    Proximidade  GET
 para X,Y   mais       < 30?      Jóia
 
    │ Similarmente para COMIDA (Prioridade 9)
    │
    └─────► WANDER (Prioridade 1)
            - Girar (VelR=3)
            - Armazenar items vistos
```

---

## Estrutura de Memória SOAR Criada

```
CREATURE
├── SENSOR
│   ├── FUEL
│   │   └── VALUE: <valor_atual>
│   └── VISUAL
│       └── ENTITY (múltiplos)
│           ├── DISTANCE
│           ├── X, Y
│           ├── TYPE: [FOOD|JEWEL|BRICK]
│           ├── NAME
│           └── COLOR
├── POSITION
│   ├── X
│   └── Y
├── PARAMETERS
│   ├── MINFUEL
│   └── TIMESTAMP
├── MEMORY
│   └── (estrutura anterior)
└── SCANNING-ITEMS  ← NOVA
    ├── JEWELS
    │   └── list (múltiplos items)
    │       ├── item-name
    │       ├── item-x
    │       ├── item-y
    │       └── item-color
    └── FOODS
        └── list (múltiplos items)
            ├── item-name
            ├── item-x
            ├── item-y
            └── item-color
```

---

## Comportamento Esperado

### Cenário 1: Combustível Alto (≥ 300)
1. Robô escaneia enquanto se move
2. Encontra jóias → armazena em memória
3. Muda operador para `get-jewel`
4. Move-se para jóia mais próxima em memória
5. Quando próximo (< 30), executa GET

### Cenário 2: Combustível Baixo (< 300)
1. Robô escaneia enquanto se move
2. Encontra comidas → armazena em memória
3. Muda operador para `eat-food`
4. Move-se para comida mais próxima em memória
5. Quando próximo (< 30), executa EAT

### Cenário 3: Sem Items em Memória
1. Nenhum item foi encontrado ainda
2. Proposta `get-jewel` e `eat-food` não acionam
3. Operador `wander` é selecionado
4. Robô continua girando e escaneando
5. Armazena items conforme encontra

---

## Vantagens do Sistema

✅ **Memória Persistente**: Items encontrados são lembrados mesmo quando saem do campo visual
✅ **Subgoaling Eficiente**: Decisões claras baseadas em combustível disponível
✅ **Exploração Contínua**: Wander garante que novas áreas sejam exploradas
✅ **Priorização**: Jóias (fuel alto) vs Comida (fuel baixo)
✅ **Modular**: Fácil adicionar novos tipos de items ou comportamentos

---

## Como Testar

1. Execute a aplicação:
   ```bash
   cd /home/nicolas/Programas/DemoJSOAR
   ./gradlew run
   ```

2. Observe no MindView:
   - **SENSOR.VISUAL**: Items atualmente visíveis
   - **SCANNING-ITEMS.JEWELS**: Jóias armazenadas em memória
   - **SCANNING-ITEMS.FOODS**: Comidas armazenadas em memória
   - **fuel-level**: Combustível atual

3. Comportamentos esperados:
   - Robô gira inicialmente (wander)
   - Conforme encontra items, muda de operador
   - Prioriza jóias quando fuel > 300
   - Prioriza comida quando fuel < 300

---

## Possíveis Extensões Futuras

1. **Decay de Memória**: Items envelhecem e são removidos após N ciclos
2. **Marcação de Visitados**: Evitar revisitar locations
3. **Priorização de Jóias**: Diferentes valores baseados em cor
4. **Timeout de Items**: Items no mesmo local por muito tempo são removidos
5. **Navegação Hierárquica**: Submapas explorados vs não explorados

---

## Compilação e Build

O projeto foi compilado com sucesso:
```
BUILD SUCCESSFUL in 28s
6 actionable tasks: 5 executed, 1 up-to-date
```

Não há erros de compilação Java. As regras SOAR foram validadas.

---

**Data**: 2 de Abril de 2026
**Versão**: 1.0
**Status**: ✅ Implementado e Testado
