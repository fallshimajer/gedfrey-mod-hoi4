# Plantilla — Decisiones con costo/cooldown/tiempo fijos por clasificación de país (gedfrey_mod_hoi4)

> Pásale este archivo a Claude Code como referencia de patrón. Resuelve el problema de que
> los costos, cooldowns y tiempos NO pueden ser dinámicos por país en una sola decisión.

---

## 1. LA REGLA (causa del problema)

En HOI4, dentro de una decisión:

| Campo | ¿Acepta variable? | Qué controla |
|---|---|---|
| `cost` | ✅ SÍ (`cost = var:mi_var`) | Coste en PP |
| `days_remove` | ❌ NO (entero fijo) | Tiempo de ejecución/timer |
| `days_re_enable` | ❌ NO (entero fijo) | Cooldown para reaparecer |

`days_remove` y `days_re_enable` se parsean al CARGAR el mod, por eso ninguna variable
funciona ahí. **Conclusión: no se puede una sola decisión con cooldown distinto por país.**

### Solución: triplicar cada decisión

Por cada decisión lógica se crean 3 versiones con valores HARDCODEADOS:
`_minor`, `_medium`, `_major`. Solo cambian `cost`, `days_remove` y `days_re_enable`.
Todo lo demás (efectos, requisitos, traits) es IDÉNTICO entre las tres.

Se filtra con `allowed` por clasificación, así solo se carga la versión correcta y no
se ensucia el menú.

- Usar **`allowed`** (se evalúa al inicio/carga, eficiente, ideal porque el tag no cambia).
- Usar **`visible`** SOLO si el país puede cambiar de tag a mitad de partida (golpe/formación).

---

## 2. CLASIFICACIÓN (ya definida en scripted_triggers)

```
gedfrey_is_minor  = yes   # resto del mundo  -> valores base
gedfrey_is_medium = yes   # FRA ITA ROC CHI BRA RAJ POL TUR SPR
gedfrey_is_major  = yes   # USA SOV GER ENG JAP
```

---

## 3. PATRÓN BASE (sin niveles, sin traits) — ejemplo: Construir Fábrica Civil

Diseño: Costo 100/120/150 · Tiempo 30 días (igual) · Cooldown 50/50/75.

```
gedfrey_economic_decisions = {

    gedfrey_build_civ_minor = {
        allowed       = { gedfrey_is_minor = yes }
        cost          = 100
        days_remove   = 30
        days_re_enable = 50
        remove_effect = {
            # +1 industrial_complex en el estado propio con más slots libres
            # (este bloque es IDÉNTICO en las 3 versiones)
        }
    }

    gedfrey_build_civ_medium = {
        allowed       = { gedfrey_is_medium = yes }
        cost          = 120
        days_remove   = 30
        days_re_enable = 50
        remove_effect = { }   # mismo efecto
    }

    gedfrey_build_civ_major = {
        allowed       = { gedfrey_is_major = yes }
        cost          = 150
        days_remove   = 30
        days_re_enable = 75
        remove_effect = { }   # mismo efecto
    }
}
```

> Lo único distinto entre las tres: `cost` y `days_re_enable`. `days_remove` aquí es igual.

---

## 4. PATRÓN CON NIVELES + TRAITS DEL LÍDER — ejemplo: Potencia Industrial

Reglas del diseño:
- 3 niveles que se REEMPLAZAN (remove del anterior + add del nuevo al country_leader).
- Requisito por fábricas (civiles + militares).
- Cooldown 120 días (grandes 180).
- N×clasificación = 3 niveles × 3 clasificaciones = **9 decisiones**.

| | Req. fab | PP | | Req. fab | PP | | Req. fab | PP |
|---|---|---|---|---|---|---|---|---|
| **minor N1** | 30 | 75 | **medium N1** | 50 | 90 | **major N1** | 70 | 113 |
| **minor N2** | 60 | 100 | **medium N2** | 100 | 120 | **major N2** | 140 | 150 |
| **minor N3** | 90 | 125 | **medium N3** | 150 | 150 | **major N3** | 210 | 188 |

Cooldown: minor/medium = 120 · major = 180.

```
gedfrey_leader_decisions = {

    # ---------- NIVEL 1 ----------
    gedfrey_industry_1_minor = {
        allowed   = { gedfrey_is_minor = yes }
        available = {
            check_variable = { gedfrey_total_factories > 29 }   # >= 30
            NOT = { has_country_leader_with_trait = gedfrey_industry_1 }
            NOT = { has_country_leader_with_trait = gedfrey_industry_2 }
            NOT = { has_country_leader_with_trait = gedfrey_industry_3 }
        }
        cost           = 75
        days_re_enable = 120
        complete_effect = { add_country_leader_trait = gedfrey_industry_1 }
    }
    gedfrey_industry_1_medium = {
        allowed   = { gedfrey_is_medium = yes }
        available = {
            check_variable = { gedfrey_total_factories > 49 }   # >= 50
            NOT = { has_country_leader_with_trait = gedfrey_industry_1 }
            NOT = { has_country_leader_with_trait = gedfrey_industry_2 }
            NOT = { has_country_leader_with_trait = gedfrey_industry_3 }
        }
        cost           = 90
        days_re_enable = 120
        complete_effect = { add_country_leader_trait = gedfrey_industry_1 }
    }
    gedfrey_industry_1_major = {
        allowed   = { gedfrey_is_major = yes }
        available = {
            check_variable = { gedfrey_total_factories > 69 }   # >= 70
            NOT = { has_country_leader_with_trait = gedfrey_industry_1 }
            NOT = { has_country_leader_with_trait = gedfrey_industry_2 }
            NOT = { has_country_leader_with_trait = gedfrey_industry_3 }
        }
        cost           = 113
        days_re_enable = 180
        complete_effect = { add_country_leader_trait = gedfrey_industry_1 }
    }

    # ---------- NIVEL 2 (requiere trait _1, lo reemplaza) ----------
    gedfrey_industry_2_minor = {
        allowed   = { gedfrey_is_minor = yes }
        available = {
            check_variable = { gedfrey_total_factories > 59 }   # >= 60
            has_country_leader_with_trait = gedfrey_industry_1
        }
        cost           = 100
        days_re_enable = 120
        complete_effect = {
            remove_country_leader_trait = gedfrey_industry_1
            add_country_leader_trait    = gedfrey_industry_2
        }
    }
    # gedfrey_industry_2_medium  -> req >=100, cost 120, cooldown 120
    # gedfrey_industry_2_major   -> req >=140, cost 150, cooldown 180
    # (mismo cuerpo, solo cambian allowed / check_variable / cost / days_re_enable)

    # ---------- NIVEL 3 (requiere trait _2, lo reemplaza) ----------
    # gedfrey_industry_3_minor   -> req >=90,  cost 125, cooldown 120
    # gedfrey_industry_3_medium  -> req >=150, cost 150, cooldown 120
    # gedfrey_industry_3_major   -> req >=210, cost 188, cooldown 180
    # complete_effect: remove gedfrey_industry_2 + add gedfrey_industry_3
}
```

### Nota sobre el requisito "fábricas = civiles + militares"

No existe un trigger único de suma. Mantener una variable actualizada a diario y
chequearla en `available` con `check_variable`:

```
# common/on_actions/gedfrey_on_actions.txt
on_startup = {
    effect = { set_variable = { gedfrey_settlement_count = 0 } }
}
on_daily = {
    effect = {
        every_country = {
            set_variable    = { gedfrey_total_factories = num_of_civilian_factories }
            add_to_variable = { gedfrey_total_factories = num_of_military_factories }
        }
    }
}
```

> Verificar que `num_of_civilian_factories` / `num_of_military_factories` sean legibles
> como variable en la versión 1.14; si no, usar `set_variable = { x = modifier@... }`
> o el game-variable equivalente.

---

## 5. LOCALIZACIÓN

Cada una de las 9 (o 3) variantes necesita su clave. Para evitar repetir texto,
todas las variantes de un mismo nivel pueden apuntar al mismo texto visible
reutilizando la descripción, pero la CLAVE debe existir por cada ID:

```
l_english:
 gedfrey_industry_1_minor:      "Potencia Industrial I"
 gedfrey_industry_1_minor_desc: "Otorga al líder el rasgo de potencia industrial (nivel 1)."
 gedfrey_industry_1_medium:      "Potencia Industrial I"
 gedfrey_industry_1_medium_desc: "Otorga al líder el rasgo de potencia industrial (nivel 1)."
 gedfrey_industry_1_major:      "Potencia Industrial I"
 gedfrey_industry_1_major_desc: "Otorga al líder el rasgo de potencia industrial (nivel 1)."
```

---

## 6. INSTRUCCIONES PARA CLAUDE CODE (aplicar a TODA la fase y siguientes)

1. **Nunca** uses variables en `days_remove` ni `days_re_enable`. Solo en `cost` (y aun
   así, preferimos triplicar para mantener un solo sistema).
2. Por cada decisión cuyo costo, cooldown o tiempo difiera entre clasificaciones, genera
   **3 variantes**: `_minor`, `_medium`, `_major`, filtradas con `allowed = { gedfrey_is_X = yes }`.
3. Define el cuerpo (`available`, `complete_effect`, `remove_effect`, `modifier`, traits)
   UNA sola vez y cópialo idéntico en las 3; cambia **solo** `cost`, `days_remove`,
   `days_re_enable` según las tablas del documento de diseño.
4. Si una decisión NO varía entre clasificaciones (mismo costo/cooldown/tiempo para todos),
   déjala como **una sola** decisión sin triplicar.
5. Si el costo difiere por nivel y/o clasificación, refléjalo en `cost` hardcodeado de cada
   variante (no uses una variable de multiplicador).
6. Traits del líder: `add_country_leader_trait` / `remove_country_leader_trait`;
   requisito de trait previo con `has_country_leader_with_trait`.
7. Crea claves de localización para CADA ID generado (incluidas las 3 variantes).
8. Respeta los sufijos de ID exactamente: `<base>_<nivel>_<clasificacion>`.

### Tabla de referencia rápida (rellenar desde el doc de diseño por cada decisión)

```
DECISIÓN: <nombre>
                    minor      medium     major
cost            =   ___        ___        ___
days_remove     =   ___        ___        ___   (omitir si no hay timer)
days_re_enable  =   ___        ___        ___
requisito       =   ___        ___        ___
efecto          =   (idéntico en las tres)
```
