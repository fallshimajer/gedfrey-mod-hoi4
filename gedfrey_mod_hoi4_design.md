# gedfrey_mod_hoi4 — Documento de Diseño Completo

## VISIÓN GENERAL DEL MOD

El mod agrega 4 secciones de contenido:
1. **Decisiones del Líder** — buffs al country_leader como traits personalizados. Los buffs DESAPARECEN si el líder es reemplazado.
2. **Área Militar** — mejora de oficiales y buffs por logros en batalla.
3. **Área Económica** — construcción, población, fábricas y buffs industriales como ideas nacionales (permanecen aunque cambie el líder).
4. **Exploración de Recursos** — sistema automático de búsqueda de minerales.

**DISTINCIÓN IMPORTANTE:**
- Decisiones del Líder → `add_trait` al country_leader (se van con el líder)
- Todos los demás buffs → `add_ideas` nacionales (permanentes a la nación)

---

## CLASIFICACIÓN DE PAÍSES (CORE DEL MOD)

**Grandes potencias** (PP +50%, Cooldown +50%, requisitos más altos):
`USA, SOV, GER, ENG, JAP`

**Potencias medianas** (PP +20%, sin cambio cooldown):
`FRA, ITA, ROC, CHI, BRA, RAJ, POL, TUR, SPR`

**Resto del mundo:** valores base

**Excepción naval:** USA y ENG tienen requisitos x2 respecto a grandes potencias.

### Scripted Triggers
```
gedfrey_is_major = { tag = USA OR tag = SOV OR tag = GER OR tag = ENG OR tag = JAP }
gedfrey_is_medium = { tag = FRA OR tag = ITA OR tag = ROC OR tag = CHI OR tag = BRA
                      OR tag = RAJ OR tag = POL OR tag = TUR OR tag = SPR }
gedfrey_is_minor = { NOT = { gedfrey_is_major = yes }
                     NOT = { gedfrey_is_medium = yes } }
gedfrey_is_naval_power = { tag = USA OR tag = ENG }
```

---

## ESTRUCTURA DE ARCHIVOS

```
gedfrey_mod_hoi4/
├── descriptor.mod
├── common/
│   ├── scripted_triggers/
│   │   └── gedfrey_scripted_triggers.txt
│   ├── decisions/
│   │   ├── gedfrey_leader_decisions.txt
│   │   ├── gedfrey_military_decisions.txt
│   │   ├── gedfrey_economic_decisions.txt
│   │   └── gedfrey_exploration_decisions.txt
│   ├── traits/
│   │   └── gedfrey_leader_traits.txt
│   ├── ideas/
│   │   └── gedfrey_ideas.txt
│   ├── characters/
│   └── on_actions/
│       └── gedfrey_on_actions.txt
├── events/
│   ├── gedfrey_leader_events.txt
│   ├── gedfrey_military_events.txt
│   ├── gedfrey_economic_events.txt
│   └── gedfrey_exploration_events.txt
└── localisation/
    └── english/
        └── gedfrey_mod_l_english.yml
```

**descriptor.mod:**
```
name = "Gedfrey Mod"
version = "0.1"
supported_version = "1.14.*"
```

---

## SECCIÓN 1 — DECISIONES DEL LÍDER

**Archivo:** `common/decisions/gedfrey_leader_decisions.txt`
**Traits:** `common/traits/gedfrey_leader_traits.txt`
**Categoría:** `gedfrey_leader_decisions`
**Cooldown base:** 120 días (grandes potencias: 180 días)
**Niveles se REEMPLAZAN:** `remove_trait` del anterior + `add_trait` del nuevo al country_leader

### Traits del Líder

#### Potencia Industrial
| Trait | Modificadores |
|---|---|
| `gedfrey_industry_1` | industrial_capacity_factory=0.05, weekly_manpower=100, manpower_recovery_speed=0.05, construction_speed=0.05, global_building_slots_factor=0.05, mining_resources_factor=0.05 |
| `gedfrey_industry_2` | industrial_capacity_factory=0.10, weekly_manpower=300, manpower_recovery_speed=0.10, construction_speed=0.10, global_building_slots_factor=0.10, mining_resources_factor=0.08 |
| `gedfrey_industry_3` | industrial_capacity_factory=0.15, weekly_manpower=750, manpower_recovery_speed=0.15, construction_speed=0.15, global_building_slots_factor=0.15, mining_resources_factor=0.12 |

#### Consolidación Política
| Trait | Modificadores |
|---|---|
| `gedfrey_politics_1` | political_power_gain=0.05 |
| `gedfrey_politics_2` | political_power_gain=0.10 |
| `gedfrey_politics_3` | political_power_gain=0.15 |

#### Gracia Real (solo monarquías)
| Trait | Modificadores |
|---|---|
| `gedfrey_royal_1` | weekly_war_support=0.03, stability_weekly=0.03, resistance_target=-0.05, industrial_capacity_factory=0.03 |
| `gedfrey_royal_2` | weekly_war_support=0.06, stability_weekly=0.06, resistance_target=-0.10, industrial_capacity_factory=0.06 |
| `gedfrey_royal_3` | weekly_war_support=0.09, stability_weekly=0.09, resistance_target=-0.15, industrial_capacity_factory=0.09, army_attack_factor=0.03, army_defence_factor=0.03 |
| `gedfrey_royal_4` ⭐ SOLO POL_anastasia_romanova | weekly_war_support=0.12, stability_weekly=0.12, resistance_target=-0.20, industrial_capacity_factory=0.12, army_attack_factor=0.05, army_defence_factor=0.05, political_power_gain=1.0, conscription_factor=0.05 |

> `# FUTURE_LEADERS:` agregar futuros líderes especiales junto a gedfrey_royal_4

#### Doctrina de Guerra
| Trait | Modificadores |
|---|---|
| `gedfrey_warfare_1` | army_attack_factor=0.02, army_defence_factor=0.02 |
| `gedfrey_warfare_2` | army_attack_factor=0.035, army_defence_factor=0.035, army_org_regain=0.05 |
| `gedfrey_warfare_3` | army_attack_factor=0.05, army_defence_factor=0.05, army_org_regain=0.10, max_dig_in=1 |

### Decisión 1 — Potencia Industrial
| Categoría | N1 | N2 | N3 |
|---|---|---|---|
| Requisito Resto | >=30 fab | >=60 fab | >=90 fab |
| Requisito Medianas | >=50 fab | >=100 fab | >=150 fab |
| Requisito Grandes | >=70 fab | >=140 fab | >=210 fab |
| PP Resto | 75 | 100 | 125 |
| PP Medianas | 90 | 120 | 150 |
| PP Grandes | 113 | 150 | 188 |

(fábricas = civiles + militares)

### Decisión 2 — Consolidación Política
| Requisito | N1 | N2 | N3 |
|---|---|---|---|
| Stability | >=0.25 | >=0.50 | >=0.75 |
| Trait previo | — | gedfrey_politics_1 | gedfrey_politics_2 |
| PP Resto | 50 | 75 | 100 |
| PP Medianas | 60 | 90 | 120 |
| PP Grandes | 75 | 113 | 150 |

### Decisión 3 — Gracia Real
Requisito base: `has_government = monarchy`

| Requisito | N1 | N2 | N3 | N4 |
|---|---|---|---|---|
| Stability | >=0.25 | >=0.40 | >=0.55 | >=0.70 |
| War Support | >=0.25 | >=0.40 | >=0.55 | >=0.70 |
| Trait previo | — | gedfrey_royal_1 | gedfrey_royal_2 | gedfrey_royal_3 |
| Especial | — | — | — | POL_anastasia_romanova |
| PP Resto | 75 | 100 | 150 | 200 |
| PP Medianas | 90 | 120 | 180 | 240 |
| PP Grandes | 113 | 150 | 225 | 300 |

### Decisión 4 — Doctrina de Guerra
| Categoría | N1 | N2 | N3 |
|---|---|---|---|
| Requisito Resto | >=300k | >=500k | >=750k |
| Requisito Medianas | >=600k | >=900k | >=1.5M |
| Requisito Grandes | >=750k | >=1.25M | >=2M |
| PP Resto | 75 | 100 | 125 |
| PP Medianas | 90 | 120 | 150 |
| PP Grandes | 113 | 150 | 188 |

(tropas desplegadas)

---

## SECCIÓN 2 — ÁREA MILITAR

**Archivo:** `common/decisions/gedfrey_military_decisions.txt`
**Categoría:** `gedfrey_military_decisions`

### Decisión 1 — Instrucción Individual
- **Costo:** Resto=100, Medianas=120, Grandes=150 PP
- **Disponible:** 30 días desde inicio
- **Cooldown:** Resto/Medianas=30 días, Grandes=45 días
- **Efecto:** Corps Commander random +1 habilidad aleatoria (random_list 25% cada una: attack / defense / maneuvering / logistics)

### Decisión 2 — Reforma Militar General
- **Costo:** Resto=500, Medianas=600, Grandes=750 PP
- **Disponible:** 300 días desde inicio
- **Cooldown:** Resto/Medianas=300 días, Grandes=450 días
- **Efecto:** TODOS los Corps Commanders +1 habilidad aleatoria individual

### Decisión 3 — Talento Especial
- **Costo:** Resto=300, Medianas=360, Grandes=450 PP
- **Disponible:** 30 días desde inicio
- **Cooldown:** Resto/Medianas=365 días, Grandes=548 días
- **Efecto:** Corps Commander random recibe trait aleatorio que NO tenga
- **Pool:** infantry_officer, armor_officer, cavalry_officer, artillery_officer, defensive_doctrine, offensive_doctrine, logistics_wizard, engineer, winter_specialist, commando, camouflage_expert, hill_fighter, jungle_rat, desert_fox, inflexible_strategist, brilliant_strategist

### Decisión 4 — Mando Supremo
- **Costo:** Resto=150, Medianas=180, Grandes=225 PP
- **Disponible:** 30 días desde inicio
- **Cooldown:** Resto/Medianas=365 días, Grandes=548 días
- **Efecto:** Field Marshal random recibe trait aleatorio que NO tenga (mismo pool)

### Evento Único — Monarca al Mando
- **ID:** `gedfrey.military.1`
- **Trigger:** automático cada 90 días via on_actions
- **Requisitos:** has_government=monarchy + líder tiene gedfrey_industry_2, gedfrey_politics_2, gedfrey_royal_2, gedfrey_warfare_2 + NOT gedfrey_monarch_promoted
- **Costo:** Resto=300, Medianas=360, Grandes=450 PP
- **Opción A:** Promover country leader como Field Marshal (attack=2, defense=2, maneuvering=2, logistics=2)
  - Si POL_anastasia_romanova: traits brilliant_strategist + old_guard
  - Si otro monarca: trait skilled_strategist
  - set_country_flag = gedfrey_monarch_promoted
- **Opción B:** Rechazar (cerrar evento)

> `# FUTURE_LEADERS:` agregar futuros líderes especiales aquí

### Buff — Supremacía en Combate Naval
- **Costo:** Resto=200, Medianas=240, Grandes=300, USA/ENG=600 PP
- **Tiempo:** 90 días | **Cooldown siguiente nivel:** Resto/Medianas=120, Grandes=180 días

| Requisito (batallas navales ganadas) | N1 | N2 | N3 |
|---|---|---|---|
| Resto | 10 | 25 | 50 |
| Medianas | 20 | 40 | 75 |
| Grandes | 35 | 60 | 110 |
| USA/ENG | 70 | 120 | 220 |

| Idea | Modificadores |
|---|---|
| `gedfrey_naval_supremacy_1` | naval_attack=0.10, naval_defence=0.10, landing_troops_factor=-0.15 |
| `gedfrey_naval_supremacy_2` | naval_attack=0.15, naval_defence=0.15, landing_troops_factor=-0.30 |
| `gedfrey_naval_supremacy_3` | naval_attack=0.25, naval_defence=0.25, landing_troops_factor=-0.50, navy_max_range_factor=0.15, repair_speed=0.20, sub_detection_factor=0.10 |

### Buff — Supremacía Aérea
- **Costo:** Resto=200, Medianas=240, Grandes=300 PP
- **Tiempo:** 90 días | **Cooldown siguiente nivel:** Resto/Medianas=120, Grandes=180 días

| Requisito (aviones derribados) | N1 | N2 | N3 |
|---|---|---|---|
| Resto | 500 | 1500 | 3000 |
| Medianas | 1000 | 2500 | 5000 |
| Grandes | 750 | 1875 | 3750 |

| Idea | Modificadores |
|---|---|
| `gedfrey_air_supremacy_1` | air_attack=0.10, air_defence=0.10, air_agility=0.05 |
| `gedfrey_air_supremacy_2` | air_attack=0.20, air_defence=0.20, air_agility=0.10 |
| `gedfrey_air_supremacy_3` | air_attack=0.30, air_defence=0.30, air_agility=0.15, paradrop_organization=0.30, air_range=0.10 |

### Buff — Veteranía de Guerra
- **Costo:** Resto=200, Medianas=240, Grandes=300 PP
- **Tiempo:** 90 días | **Cooldown siguiente nivel:** Resto/Medianas=120, Grandes=180 días

| Requisito (bajas enemigas acumuladas) | N1 | N2 | N3 |
|---|---|---|---|
| Resto | 100k | 300k | 700k |
| Medianas | 250k | 600k | 1.5M |
| Grandes | 187k | 450k | 1.1M |

| Idea | Modificadores |
|---|---|
| `gedfrey_war_veterancy_1` | army_org_regain=0.05, army_attack_factor=0.02 |
| `gedfrey_war_veterancy_2` | army_org_regain=0.10, army_attack_factor=0.03, army_defence_factor=0.03 |
| `gedfrey_war_veterancy_3` | army_org_regain=0.15, army_attack_factor=0.05, army_defence_factor=0.05, attrition=-0.10 |

---

## SECCIÓN 3 — ÁREA ECONÓMICA

**Archivo:** `common/decisions/gedfrey_economic_decisions.txt`
**Categoría:** `gedfrey_economic_decisions`

### Decisión 1 — Construir Fábrica Militar
- **Costo:** Resto=100, Medianas=120, Grandes=150 PP
- **Tiempo:** 30 días | **Cooldown:** Resto/Medianas=50, Grandes=75 días
- **Efecto:** +1 arms_factory en estado propio con más slots libres

### Decisión 2 — Construir Fábrica Civil
- **Costo:** Resto=100, Medianas=120, Grandes=150 PP
- **Tiempo:** 30 días | **Cooldown:** Resto/Medianas=50, Grandes=75 días
- **Efecto:** +1 industrial_complex en estado propio con más slots libres

### Decisión 3 — Construir Astillero
- **Costo:** Resto=100, Medianas=120, Grandes=150 PP
- **Tiempo:** 30 días | **Cooldown:** Resto/Medianas=60, Grandes=90 días
- **Efecto:** +1 dockyard en estado costero propio con más slots libres

### Decisión 4 — Asentamiento Regional
- **Costo:** Resto=50, Medianas=60, Grandes=75 PP
- **Tiempo:** 30 días | **Cooldown:** Resto/Medianas=30, Grandes=45 días
- **Efecto base:** Top 5 estados propios con menor manpower → elegir 2 random → +50000 manpower a CADA uno
- **Efecto mejorado** (si líder tiene `gedfrey_politics_2` OR `gedfrey_politics_3`):
  - +75000 manpower a cada uno
  - Cooldown: Resto/Medianas=20, Grandes=30 días
  - Tiempo: 15 días
- **Contador:** `add_to_variable = { var = gedfrey_settlement_count value = 1 }`

### Decisión 5 — Inversión Extranjera
- **Costo:** Resto=150, Medianas=180, Grandes=225 PP
- **Tiempo:** 30 días | **Cooldown:** Resto/Medianas=30, Grandes=45 días
- **Efecto random_list:**
  - 75%: +1 edificio aleatorio (arms_factory, industrial_complex o dockyard) en estado extranjero con slots libres
  - 25%: evento gedfrey.economic.1 "La inversión falló"

### Decisión 6 — Reubicar Fábrica Civil
- **Costo:** Resto=50, Medianas=60, Grandes=75 PP
- **Tiempo:** 30 días | **Cooldown:** Resto/Medianas=60, Grandes=90 días
- **Requisito:** industrial_complex >= 1
- **Efecto:** Eliminar 1 civil del estado con MENOS civiles + añadir 1 en estado extranjero aleatorio con slots libres

### Decisión 7 — Reubicar Fábrica Militar
- Igual que Decisión 6 pero con arms_factory

### Decisión 8 — Reubicar Astillero
- Igual que Decisión 6 pero con dockyard en estado costero

### Decisión 9 — Conversión Fábrica Civil a PP
- **Disponible:** Solo hasta 1940.12.31
- **Sin cooldown** post-expiración, re-ejecutable inmediatamente
- **Costo:** Resto=50, Medianas=60, Grandes=75 PP
- **Requisito:** Resto/Medianas >= 1 civil | Grandes >= 3 civiles
- **Efecto:** Eliminar fábricas + idea `gedfrey_civil_conversion`: political_power_gain=1.0 durante 60 días → al expirar devolver fábricas

### Decisión 10 — Conversión Fábrica Militar a PP
- Igual que Decisión 9 pero con arms_factory
- Idea `gedfrey_military_conversion`: political_power_gain=1.0

### Decisión 11 — Conversión Astillero a PP
- Igual que Decisión 9 pero con dockyard
- Idea `gedfrey_dockyard_conversion`: political_power_gain=1.0

> Las tres conversiones se pueden ejecutar simultáneamente.

### Buff A — Eficiencia Civil
- **Costo:** Resto=100, Medianas=120, Grandes=150 PP
- **Tiempo:** 120 días | **Cooldown siguiente nivel:** Resto/Medianas=300, Grandes=450 días

| Requisito (civiles + militares) | N1 | N2 | N3 |
|---|---|---|---|
| Resto | >=60 | >=90 | >=120 |
| Medianas | >=100 | >=150 | >=200 |
| Grandes | >=140 | >=210 | >=280 |

| Idea | Modificadores |
|---|---|
| `gedfrey_consumer_efficiency_1` | consumer_goods_factor=-0.03 |
| `gedfrey_consumer_efficiency_2` | consumer_goods_factor=-0.06 |
| `gedfrey_consumer_efficiency_3` | consumer_goods_factor=-0.10 |

### Buff B — Potencia Constructora
- **Costo:** Resto=100, Medianas=120, Grandes=150 PP
- **Tiempo:** 120 días | **Cooldown siguiente nivel:** Resto/Medianas=300, Grandes=450 días

| Requisito (fábricas civiles) | N1 | N2 | N3 |
|---|---|---|---|
| Resto | >=30 | >=60 | >=90 |
| Medianas | >=50 | >=100 | >=150 |
| Grandes | >=70 | >=140 | >=210 |

| Idea | Modificadores |
|---|---|
| `gedfrey_construction_power_1` | construction_speed=0.10 |
| `gedfrey_construction_power_2` | construction_speed=0.20 |
| `gedfrey_construction_power_3` | construction_speed=0.30 |

### Buff C — Supremacía Naval Económica
- **Costo:** Resto=200, Medianas=240, Grandes=300 PP
- **Tiempo:** 120 días | **Cooldown siguiente nivel:** Resto/Medianas=300, Grandes=450 días

| Requisito (astilleros) | N1 | N2 | N3 |
|---|---|---|---|
| Resto | >=25 | >=40 | >=60 |
| Medianas | >=50 | >=75 | >=100 |
| Grandes | >=70 | >=105 | >=140 |

| Idea | Modificadores |
|---|---|
| `gedfrey_naval_economy_1` | industrial_capacity_dockyard=0.10, repair_speed=0.10 |
| `gedfrey_naval_economy_2` | industrial_capacity_dockyard=0.20, repair_speed=0.15 |
| `gedfrey_naval_economy_3` | industrial_capacity_dockyard=0.30, repair_speed=0.25 |

### Buff D — Crecimiento Nacional
- **Requisito nivel N:** gedfrey_settlement_count >= (N * 10) + idea del nivel anterior (excepto N1)
- **Tiempo:** 120 días | **Cooldown siguiente nivel:** Resto/Medianas=300, Grandes=450 días

| Nivel | PP Resto | PP Medianas | PP Grandes |
|---|---|---|---|
| N1 | 75 | 90 | 113 |
| N2 | 100 | 120 | 150 |
| N3 | 125 | 150 | 188 |
| N4 | 150 | 180 | 225 |
| N5 | 175 | 210 | 263 |

| Idea | Modificadores |
|---|---|
| `gedfrey_national_growth_1` | conscription_factor=0.01, weekly_manpower=100 |
| `gedfrey_national_growth_2` | conscription_factor=0.02, weekly_manpower=200 |
| `gedfrey_national_growth_3` | conscription_factor=0.03, weekly_manpower=300 |
| `gedfrey_national_growth_4` | conscription_factor=0.04, weekly_manpower=400 |
| `gedfrey_national_growth_5` | conscription_factor=0.05, weekly_manpower=500 |

### Bonus Naciones Subdesarrolladas
- **Disponible:** 1936.1.1 a 1939.12.31
- **Evento único** por país (`set_country_flag = gedfrey_underdeveloped_taken`)

| | RESTO | MEDIANAS |
|---|---|---|
| Costo PP | 50 | 100 |
| Tiempo | 50 días | 50 días |
| Duración | 730 días | 365 días |
| Idea | `gedfrey_underdeveloped_nation` | `gedfrey_developing_nation` |
| political_power_gain | 0.25 | 0.12 |
| focus_gain_speed | 0.25 | 0.12 |
| research_speed_factor | 0.25 | 0.12 |
| construction_speed | 0.25 | 0.12 |

### Eventos Únicos del Líder
**Archivo:** `events/gedfrey_leader_events.txt`
Ideas nacionales (NO traits), permanentes a la nación. Independientes entre sí.

| Evento | Requisito | Costo | Idea | Modificador |
|---|---|---|---|---|
| A — Velocidad Enfoques | Líder tiene _2 en las 4 decisiones | Resto=50, Med=60, Gr=75 PP | `gedfrey_focus_speed` | focus_gain_speed=0.25 |
| B — Ranura Investigación | Líder tiene _2 en las 4 decisiones | Resto=50, Med=60, Gr=75 PP | `gedfrey_research_slot` | research_slots=1 |
| C — Velocidad Investigación | Líder tiene _3 en las 4 decisiones | Resto=50, Med=60, Gr=75 PP | `gedfrey_research_speed` | research_speed_factor=0.25 |

Tiempo: 120 días. Evento único con set_country_flag.

---

## SECCIÓN 4 — EXPLORACIÓN DE RECURSOS

**Archivo:** `common/decisions/gedfrey_exploration_decisions.txt`
**Solo disponible para jugador humano** (`is_ai = no`)

### Decisión — Iniciar Exploración
- **Costo:** Resto=50, Medianas=60, Grandes=75 PP
- **Tiempo:** 60 días
- **Requisito:** NOT has_country_flag = gedfrey_exploration_active
- **Efecto:** set_country_flag = gedfrey_exploration_active → dispara cadena gedfrey.exploration

### Decisión — Detener Exploración
- **Costo:** 0 PP | Instantánea
- **Requisito:** has_country_flag = gedfrey_exploration_active
- **Efecto:** clr_country_flag = gedfrey_exploration_active

### Cadena de Eventos gedfrey.exploration
- Se repite cada 60 días mientras gedfrey_exploration_active esté activo
- **Costo por repetición:** Resto=25, Medianas=30, Grandes=38 PP (descontar aunque PP sea negativo)

**Sistema de deuda PP:**
- Si PP < 0: descontar igual y continuar
- Si PP <= -300: cancelar + penalización -100 PP + evento gedfrey.exploration.cancel

**Resultados (random_list):**
| Probabilidad | Resultado | Evento |
|---|---|---|
| 30% | +2 aluminio en estado propio aleatorio | gedfrey.exploration.1 |
| 20% | +2 hierro en estado propio aleatorio | gedfrey.exploration.2 |
| 20% | +2 tungsteno en estado propio aleatorio | gedfrey.exploration.3 |
| 10% | +2 cromo en estado propio aleatorio | gedfrey.exploration.4 |
| 20% | Sin resultado | Sin evento (silencioso) |

---

## INICIALIZACIÓN DE VARIABLES

**Archivo:** `common/on_actions/gedfrey_on_actions.txt`

Al inicio de partida (on_startup):
```
set_variable = { var = gedfrey_settlement_count value = 0 }
```

---

## NOTAS TÉCNICAS

- Usar scripted_triggers para gedfrey_is_major/medium/minor/naval_power
- Traits del líder en `common/traits/` como country_leader traits
- Verificar compatibilidad con Road to 56 para POL_anastasia_romanova
- Dejar comentario `# FUTURE_LEADERS` en gedfrey_royal_4 y Monarca al Mando
- El mod no requiere DLC pero aprovechar NSB si está disponible
- Usar `timed_idea` con duration=60 para devolver fábricas en decisiones 9-11
- Usar `add_to_variable` / `set_variable` para gedfrey_settlement_count
- `remove_ideas` antes de `add_ideas` al subir niveles de buffs nacionales
- `remove_trait` antes de `add_trait` al subir niveles de traits del líder

---

## PLAN DE DESARROLLO POR FASES

Implementar **SOLO** la fase indicada en cada sesión.

### FASE 1 — Core completo
1. Descriptor y estructura completa de archivos
2. Scripted triggers (sistema de países)
3. Traits del líder (`gedfrey_leader_traits.txt`)
4. Decisiones del líder — las 4 decisiones con traits
5. Decisiones económicas 1-3 (construir fábricas y astillero)
6. Decisiones militares 1-4 (oficiales, habilidades y traits)
7. Inicialización de variables en on_actions
8. Localización de todo lo anterior

### FASE 2 — Buffs económicos
- Decisión 4: Asentamiento Regional
- Decisión 5: Inversión Extranjera
- Decisiones 6, 7, 8: Reubicación de fábricas
- Buff A: Eficiencia Civil
- Buff B: Potencia Constructora
- Buff C: Supremacía Naval Económica
- Buff D: Crecimiento Nacional

### FASE 3 — Eventos especiales
- Eventos únicos del líder (A, B, C)
- Evento único Monarca al Mando
- Decisiones 9, 10, 11: Conversiones a PP (hasta 1940)
- Bonus Naciones Subdesarrolladas (hasta 1939)

### FASE 4 — Exploración de recursos
- Sección completa de Exploración de Recursos

### FASE 5 — Buffs militares avanzados
- Buff: Supremacía en Combate Naval
- Buff: Supremacía Aérea
- Buff: Veteranía de Guerra
