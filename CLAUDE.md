# Gedfrey Mod — HOI4 Mod Project

## Contexto general
Mod para Hearts of Iron IV 1.14.x (Götterdämmerung) que agrega 4 sistemas:
1. **Decisiones del Líder** — traits al country_leader (se van si el líder cambia)
2. **Área Militar** — mejora de oficiales e ideas nacionales por logros en batalla
3. **Área Económica** — construcción, población, fábricas e ideas nacionales permanentes
4. **Exploración de Recursos** — sistema automático de búsqueda de minerales

**Distinción clave:** Decisiones del Líder → `add_country_leader_trait` (temporal). Todo lo demás → `add_ideas` (permanente a la nación).

---

## Clasificación de países

```
gedfrey_is_major    → USA, SOV, GER, ENG, JAP      (PP +50%, cooldown +50%)
gedfrey_is_medium   → FRA, ITA, ROC, CHI, BRA, RAJ, POL, TUR, SPR  (PP +20%)
gedfrey_is_minor    → todos los demás               (valores base)
gedfrey_is_naval_power → USA, ENG                  (requisitos navales x2 vs majors)
```

---

## Estado actual — FASE 1 implementada y funcionando

### Archivos del mod
```
gedfrey-mod-hoi4/
├── CLAUDE.md                          ← este archivo
├── DEVLOG.md                          ← registro de bugs y fixes
├── descriptor.mod
├── common/
│   ├── decisions/
│   │   ├── categories/
│   │   │   └── gedfrey_categories.txt       ← HOI4 1.14: debe ir en decisions/categories/
│   │   ├── gedfrey_leader_decisions.txt
│   │   ├── gedfrey_military_decisions.txt
│   │   ├── gedfrey_economic_decisions.txt
│   │   └── gedfrey_exploration_decisions.txt (placeholder vacío)
│   ├── country_leader/
│   │   └── gedfrey_leader_traits.txt
│   ├── ideas/
│   │   └── gedfrey_ideas.txt                (placeholder vacío, fase 2+)
│   ├── on_actions/
│   │   └── gedfrey_on_actions.txt
│   ├── scripted_triggers/
│   │   └── gedfrey_scripted_triggers.txt
│   └── scripted_values/
│       └── gedfrey_scripted_values.txt
├── events/
│   ├── gedfrey_economic_events.txt
│   ├── gedfrey_exploration_events.txt
│   ├── gedfrey_leader_events.txt
│   └── gedfrey_military_events.txt
└── localisation/
    └── english/
        └── gedfrey_mod_l_english.yml
```

### Decisiones implementadas en Fase 1
- **Leader Directives**: Industrial Power I-III, Political Consolidation I-III, Royal Grace I-IV, Doctrine of War I-III
- **Military Development**: Individual Training, Military Reform, Special Talent, Supreme Command
- **Economic Development**: Construct Military Factory, Construct Civilian Factory, Construct Naval Shipyard

---

## LIMITACIONES CRÍTICAS DE HOI4 1.14 — leer antes de modificar archivos

### 1. Carpeta de categorías de decisiones
**Ruta correcta:** `common/decisions/categories/` (NO `common/decision_categories/`)

### 2. Operador `>=` roto en triggers de enteros y fechas
HOI4 1.14 tokeniza `>=` como dos tokens (`>` y `=`). **Usar siempre `> X-1`.**
- `>= 30` → `> 29`
- `>= 1936.1.31` → `> 1936.1.30`
- Floats como `stability >= 0.25` también fallan en esta instalación (conflicto con otros mods activos)

### 3. `cost` y `days_re_enable` no aceptan scripted values
El campo `cost` en decisiones **solo acepta números directos**. `cost = nombre_scripted_value` retorna 0 silenciosamente. Mismo problema con `days_re_enable`. **Usar números hardcodeados.**

### 4. Bloques inline en `cost` rompen el parser
`cost = { base = X modifier = { ... } }` hace que HOI4 trate el bloque como una sub-decisión. **No usar.**

### 5. Scripted values que referencian scripted triggers → devuelven 0
Los scripted values se cargan ANTES que los scripted triggers. Si un scripted value usa `gedfrey_is_medium = yes` en sus modifiers, falla silenciosamente. **Usar tags directos** (`OR = { tag = FRA tag = ITA ... }`).

### 6. `stability` y `war_support` como triggers inválidos
Probablemente por conflicto con mods activos (ugc_*.mod). En decisiones disponibles del jugador, **no usar estos triggers**. Removidos de las condiciones `available`.

### 7. `priority = { base = X }` inválido en categorías
En `decision_categories`, `priority` solo acepta número directo: `priority = 100`.

### 8. Modifiers inválidos en leader traits (scope de country_leader)
| Inválido | Reemplazo aplicado |
|---|---|
| `mining_resources_factor` | `local_resources_factor` |
| `manpower_recovery_speed` | `monthly_population` |
| `construction_speed` | `production_speed_buildings_factor` |
| `oil_factor`, `steel_factor`, etc. | `local_resources_factor` |
| `weekly_war_support` | `war_support_factor` |

### 9. `has_government = monarchy` inválido
Usar `has_government = neutrality` como aproximación.

### 10. `add_maneuvering = 1` renombrado en HOI4 1.14
Usar `add_planning = 1`.

---

## Costos y cooldowns actuales (hardcodeados)

### Leader Directives
| Decisión | Costo PP | Cooldown |
|---|---|---|
| Industrial Power I | 75 | 120 días |
| Industrial Power II | 100 | 120 días |
| Industrial Power III | 125 | 120 días |
| Political Consolidation I | 50 | 120 días |
| Political Consolidation II | 75 | 120 días |
| Political Consolidation III | 100 | 120 días |
| Royal Grace I | 75 | 120 días |
| Royal Grace II | 100 | 120 días |
| Royal Grace III | 150 | 120 días |
| Royal Grace IV | 200 | 120 días |
| Doctrine of War I | 75 | 120 días |
| Doctrine of War II | 100 | 120 días |
| Doctrine of War III | 125 | 120 días |

### Military Development
| Decisión | Costo PP | Timer ejecución | Cooldown |
|---|---|---|---|
| Individual Training | 100 | 10 días | 30 días |
| Military Reform | 500 | 10 días | 300 días |
| Special Talent | 300 | 10 días | 365 días |
| Supreme Command | 150 | 10 días | 365 días |

### Economic Development
| Decisión | Costo PP | Timer ejecución | Cooldown |
|---|---|---|---|
| Build Military Factory | 100 | 30 días | 50 días |
| Build Civilian Factory | 100 | 30 días | 50 días |
| Build Naval Shipyard | 100 | 30 días | 60 días |

> **Nota:** Los costos deberían escalar por tier (minor/medium/major) pero el campo `cost` no acepta scripted values en HOI4 1.14. Todos pagan el costo base (minor).

---

## Fases pendientes

### Fase 2
- Decisión 4: Asentamiento Regional (con gedfrey_settlement_count)
- Decisión 5: Inversión Extranjera
- Decisiones 6-8: Reubicación de fábricas
- Buff A: Eficiencia Civil (idea nacional)
- Buff B: Potencia Constructora (idea nacional)
- Buff C: Supremacía Naval Económica (idea nacional)
- Buff D: Crecimiento Nacional (idea nacional, 5 niveles)

### Fase 3
- Eventos únicos del líder (A: Focus Speed, B: Research Slot, C: Research Speed)
- Evento Monarca al Mando ⚠ BLOQUEADO — `add_field_marshal_role` no funciona en country leaders dinámicos. Requiere pre-definir personajes por país en `common/characters/`. Ver DEVLOG #12.
- Decisiones 9-11: Conversiones de fábricas a PP (disponible hasta 1940)
- Bonus Naciones Subdesarrolladas (disponible hasta 1939)

### Fase 4
- Exploración de Recursos (sistema automático con cadena de eventos cada 60 días)

### Fase 5 ✓ IMPLEMENTADA
- Buff: Supremacía en Combate Naval — 12 decisiones (minor/medium/major/naval), ideas gedfrey_naval_supremacy_1/2/3
- Buff: Supremacía Aérea — 9 decisiones (minor/medium/major), ideas gedfrey_air_supremacy_1/2/3
- Buff: Veteranía de Guerra — 9 decisiones (minor/medium/major), ideas gedfrey_war_veterancy_1/2/3

---

## Notas de implementación futura

- `gedfrey_settlement_count` se inicializa en `on_actions` con `set_variable` → incrementar en cada Asentamiento Regional
- Ideas nacionales que suben de nivel: usar `remove_ideas` antes de `add_ideas`
- Decisiones de conversión (Fase 3): usar `timed_idea` con `duration = 60` para devolver fábricas al expirar
- Exploración (Fase 4): usar `on_actions` con check periódico o evento que se repite a sí mismo
- POL_anna_andersson: verificar compatibilidad con Road to 56 antes de Fase 3
- Comentario `# FUTURE_LEADERS` en gedfrey_royal_4 para agregar líderes especiales futuros
