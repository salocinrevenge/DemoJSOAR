# Guia Rápido - Sistema de Subgoaling

## Em Uma Frase
**O robô agora escaneia o ambiente, lembra onde estão os items, e decide ir buscar jóias (se tem energia) ou comida (se está fraco).**

---

## Os 3 Subgoals

### 1. 🔄 WANDER (Escaneamento)
**Quando**: Sempre disponível (fallback)
**O que faz**: Robô gira sobre si mesmo
**Por quê**: Encontra items e armazena em memória

```
Velocidades:
  VelR = 3  (roda direita rápida)
  VelL = 0  (roda esquerda parada)
  → Resultado: Rotação no lugar
```

**Itens encontrados durante wander**:
```
Visual Sensor detecta FOOD/JEWEL
        ↓
Adicionado a: SCANNING-ITEMS.FOODS ou SCANNING-ITEMS.JEWELS
        ↓
Permanece em memória mesmo quando sai da visão
```

---

### 2. 💎 GET-JEWEL (Buscar Jóia)
**Quando**: combustível >= 300 **E** há jóias em memória
**Prioridade**: 10 (alta)
**O que faz**: Move para jóia mais próxima → Pega

```
Pseudocódigo:
if (fuel >= 300) AND (scanning-items.jewels.list != empty) then
    MOVING-TO-JEWEL do
        target = closest-jewel-in-memory
        move-to(target.x, target.y)
        if distance < 30 then
            execute GET(jewel-name)
```

---

### 3. 🍎 EAT-FOOD (Buscar Comida)
**Quando**: combustível < 300 **E** há comidas em memória
**Prioridade**: 9 (média-alta)
**O que faz**: Move para comida mais próxima → Come

```
Pseudocódigo:
if (fuel < 300) AND (scanning-items.foods.list != empty) then
    MOVING-TO-FOOD do
        target = closest-food-in-memory
        move-to(target.x, target.y)
        if distance < 30 then
            execute EAT(food-name)
```

---

## Árvore de Decisão

```
┌─ Iniciar ciclo
│
├─ Ler combustível atual
│
└─ Decidir operador:
   │
   ├─ SE (fuel >= 300) E (há jóias em memória)
   │  └─► GET-JEWEL ⭐ Prioridade 10
   │
   ├─ SE (fuel < 300) E (há comidas em memória)
   │  └─► EAT-FOOD ⭐ Prioridade 9
   │
   └─ SENÃO (nenhuma condição atendida)
      └─► WANDER ⭐ Prioridade 1 (fallback)
```

---

## Exemplo de Execução Passo-a-Passo

### Ciclo 1-50: Exploração
```
Combustível: 500/500
Ação: WANDER (girar, escanear)
Encontrados: 3 jóias, 0 comidas
Memória:
  - JEWEL_1 (x=100, y=150)
  - JEWEL_2 (x=200, y=100)
  - JEWEL_3 (x=50, y=200)
```

### Ciclo 51: Mudança de Estratégia
```
Combustível: 450/500
Condição: fuel >= 300 ✓ E há jóias ✓
Decisão: MUDA para GET-JEWEL
Elegido: JEWEL_2 (mais próximo)
```

### Ciclo 51-80: Buscar Jóia
```
Ciclo 60: MOVING para JEWEL_2 (200, 100)
Ciclo 70: Distância = 42 unidades
Ciclo 71: Distância = 28 unidades ← DENTRO DO ALCANCE!
Ciclo 71: EXECUTA GET JEWEL_2
Combustível: 420/500 (consumido durante movimento)
```

### Ciclo 81: Volta ao Scan?
```
Combustível: 420/500
Ação: WANDER (combustível ainda >= 300)
Próximo destino: JEWEL_1 ou JEWEL_3
```

### Ciclo 200: Combustível Baixo
```
Combustível: 240/500
Condição: fuel < 300 ✓ E há comidas? ✗
Decisão: WANDER (buscar por comida)
Encontrados: FOOD_1 (x=300, y=50)
SCANNING-ITEMS.FOODS: [FOOD_1]
```

### Ciclo 201: Prioridade Muda
```
Combustível: 235/500
Condição: fuel < 300 ✓ E há comidas ✓
Decisão: MUDA para EAT-FOOD
Elegido: FOOD_1
```

---

## Estrutura de Memória Simplificada

### Antes (sem subgoaling):
```
CREATURE
├── SENSOR.VISUAL.ENTITY[...] ← Só itens visíveis NOW!
└── MEMORY
```

### Depois (com subgoaling):
```
CREATURE
├── SENSOR.VISUAL.ENTITY[...] ← Itens visíveis NOW
├── MEMORY
└── SCANNING-ITEMS ← NOVO: Histórico de items encontrados
    ├── JEWELS.list = [
    │     {name: "JEWEL_1", x: 100, y: 150, color: "red"},
    │     {name: "JEWEL_2", x: 200, y: 100, color: "blue"},
    │     ...
    │   ]
    └── FOODS.list = [
          {name: "FOOD_1", x: 300, y: 50, color: "green"},
          ...
        ]
```

---

## Modificações no Código

### Arquivo: `soar-rules-ativ1.soar`
- ✏️ Substituído completamente
- 🆕 Estrutura `scanned-items` inicializada
- 🆕 3 operadores principais com prioridades
- 🆕 Regras para armazenar items durante wander

### Arquivo: `SoarBridge.java`
- 🆕 `Identifier creatureScanningItems` adicionado
- 🆕 Inicialização de `SCANNING-ITEMS` em `prepareInputLink()`

---

## Como Validar

### ✅ Compilação:
```bash
cd /home/nicolas/Programas/DemoJSOAR
./gradlew build -x test
# Resultado: BUILD SUCCESSFUL
```

### ✅ Verificação Visual no MindView:
- Monitor o campo "SCANNING-ITEMS"
- Veja items sendo adicionados ao escanear
- Acompanhe decisões de operador no campo "current-subgoal"

---

## Regras de Funcionamento

| Combustível | Jóias em Memória | Comidas em Memória | Ação         | Prioridade |
|------------|-----------------|-------------------|---------|-----------|
| ≥ 300      | SIM             | -                 | GET-JEWEL | 10        |
| < 300      | -               | SIM               | EAT-FOOD | 9         |
| Qualquer   | NÃO             | NÃO               | WANDER   | 1         |

---

## FAQ Rápido

**P: O robô continua escaneando enquanto busca comida/jóia?**
R: Sim! Durante `get-jewel` e `eat-food`, ele continua agregando itens visuais à memória.

**P: Items velhos são removidos da memória?**
R: Não, por enquanto permanecem indefinidamente. Possível extensão futura.

**P: Por que jóia tem prioridade 10 e comida 9?**
R: Porque jóias geralmente são mais valiosas, então coletá-las enquanto há energia é preferível.

**P: O que acontece se não acham item em memória?**
R: Volta a WANDER (prioridade 1, fallback) e continua escaneando.

---

**Versão**: 1.0 | **Data**: 2 de Abril de 2026 | **Status**: ✅ Pronto para Uso
