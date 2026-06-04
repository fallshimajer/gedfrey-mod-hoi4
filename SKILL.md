---
name: hoi4-modding
description: Guía técnica para crear y editar mods de Hearts of Iron IV (HOI4), enfocada en decisiones, ideas nacionales, traits de líder, eventos, variables, scripted_triggers, on_actions y localización. Úsala SIEMPRE que el trabajo toque archivos en common/decisions, common/ideas, common/traits, common/scripted_triggers, common/on_actions, events/ o localisation/ de un mod de HOI4, o cuando el usuario hable de "decisión", "idea nacional", "trait", "cooldown", "PP", "descriptor.mod", "país mayor/mediano/menor" o pida generar código de paradox script. Aplica a la versión 1.18 "Peace for Our Time" y posteriores.
---

# HOI4 Modding (Paradox Script)

Skill para escribir Paradox Script correcto y evitar los errores que más rompen mods.
Mantén SKILL.md como referencia rápida; los detalles largos están en `references/`.

## Versión del juego (verificar siempre)

A mayo de 2026 la versión actual es **1.18 "Peace for Our Time"** (abril 2026).
Línea de versiones: 1.14 (mar 2024) → 1.15 (nov 2024) → 1.16 (mar 2025) → 1.17
"No Compromise, No Surrender" → 1.18 "Peace for Our Time".

- En `descriptor.mod`, `supported_version` debe coincidir con la versión donde se prueba.
  Usa el comodín de rama, p. ej. `supported_version = "1.18.*"`.
- Un `supported_version` viejo (ej. 1.14) NO impide cargar el mod, pero el launcher lo
  marca como desactualizado. Actualízalo al probar en una versión nueva.

## REGLA CRÍTICA: campos estáticos vs. variables en decisiones

Este es el error #1 al generar decisiones. Dentro de una decisión:

| Campo | ¿Acepta variable? | Controla |
|---|---|---|
| `cost` | SÍ (`cost = var:mi_var`) | Coste en PP |
| `days_remove` | NO — entero fijo | Timer de ejecución |
| `days_re_enable` | NO — entero fijo | Cooldown de reaparición |

`days_remove` y `days_re_enable` se parsean al CARGAR el mod, así que ninguna variable
funciona ahí. Si necesitas que el coste/cooldown/tiempo varíe por tipo de país, NO uses
variables: **triplica la decisión** (ver patrón abajo).

## REGLA CRÍTICA: operadores de check_variable

`check_variable` solo acepta `>`, `<` y `=`. **NO existe `>=` ni `<=`** (da error de parseo).
Para enteros, expresa "mayor o igual a N" como `> (N-1)`:

```
# "fábricas >= 30"  ->  correcto:
check_variable = { gedfrey_total_factories > 29 }
```

Esto solo es exacto con enteros. Para decimales (stability, war_support 0–1), usa el
valor justo, p. ej. `> 0.249` para representar `>= 0.25`.

## Patrón de triplicación por clasificación de país

Cuando coste/cooldown/tiempo difieren por tipo de país, crea 3 versiones con valores
hardcodeados y filtra con `allowed`. El cuerpo (efectos, requisitos, traits) es idéntico;
solo cambian `cost`, `days_remove`, `days_re_enable`.

```
mi_categoria = {
    decision_minor = {
        allowed       = { gedfrey_is_minor = yes }
        cost          = 100
        days_re_enable = 50
        complete_effect = { }   # IDÉNTICO en las 3
    }
    decision_medium = {
        allowed       = { gedfrey_is_medium = yes }
        cost          = 120
        days_re_enable = 50
        complete_effect = { }
    }
    decision_major = {
        allowed       = { gedfrey_is_major = yes }
        cost          = 150
        days_re_enable = 75
        complete_effect = { }
    }
}
```

- Usa `allowed` (se evalúa solo al inicio/carga; la clasificación por tag no cambia → eficiente).
- Usa `visible` SOLO si el país puede cambiar de tag a mitad de partida (golpe/formación).
- Si una decisión NO varía entre tipos, déjala como UNA sola (no tripliques de más).

## Triggers de decisión (cuándo se evalúa cada uno)

- `allowed` — una vez, al inicio y al cargar save. Para restringir por tag/DLC. Si es
  falso, la decisión queda deshabilitada para siempre (hasta recargar).
- `visible` — cada frame; controla si aparece en el menú.
- `available` — cada frame; si es falso la decisión se ve gris (no se puede tomar).

No pongas condiciones dinámicas en `allowed`; van en `visible`/`available`.

## Timer y reaparición

- `days_re_enable = N` — cooldown antes de reaparecer (por defecto reaparece al día siguiente).
- `fire_only_once = yes` — la decisión solo puede tomarse una vez por país.
- `days_remove = N` — días entre tomar la decisión (`complete_effect`) y `remove_effect`.
  `days_remove = -1` = nunca se quita por timer.
- `remove_effect = { }` — efectos al terminar el timer (p. ej. añadir el edificio construido).
- `cancel_trigger` / `cancel_effect` — cancelar el timer antes de tiempo sin disparar remove_effect.

## Aleatoriedad en decisiones

Las decisiones usan semilla FIJA por defecto: `random_list` elige siempre lo mismo.
Para que sea realmente aleatorio añade `fixed_random_seed = no` a la decisión.

## Traits de líder vs. ideas nacionales

- **Traits del país-líder** (se van si cambia el líder):
  - `add_country_leader_trait = X` / `remove_country_leader_trait = X`
  - trigger: `has_country_leader_with_trait = X`
  - Se definen en `common/traits/` como `leader_traits = { ... }`.
  - Al subir de nivel: `remove_country_leader_trait` del anterior + `add_country_leader_trait` del nuevo.
- **Ideas nacionales** (permanentes a la nación):
  - `add_ideas = X` / `remove_ideas = X`
  - Temporales: `add_timed_idea = { idea = X days = N }` (se quita sola al expirar).
  - Al subir de nivel: `remove_ideas` del anterior antes de `add_ideas` del nuevo.

## on_actions y variables

```
# common/on_actions/gedfrey_on_actions.txt
on_actions = {
    on_startup = { effect = { set_variable = { gedfrey_settlement_count = 0 } } }
    on_daily   = { effect = { every_country = { ... } } }   # también on_weekly / on_monthly
}
```

- `set_variable`, `add_to_variable`, `multiply_temp_variable`, etc. operan en efectos, no en triggers.
- Para comparar una suma (ej. civiles+militares) mantén una variable diaria y léela con `check_variable`.
- Verifica que game-variables como `num_of_civilian_factories` sean legibles en la versión
  actual; si no, usa el equivalente vía `modifier@...` o el game variable correspondiente.

## Localización

- Cada ID (decisión, idea, trait, evento, opción) necesita su clave, o aparece `[clave]` en pantalla.
- Las descripciones usan el sufijo `_desc`.
- Si triplicas decisiones, crea clave para CADA variante (`_minor`, `_medium`, `_major`),
  aunque el texto visible sea el mismo.
- Desde 1.15 cambió cómo funciona la localización scripted (se necesita menos loc manual);
  si generas tooltips dinámicos, revisa `references/localisation.md`.

## Flujo de trabajo recomendado

1. Confirmar la versión objetivo y el `supported_version`.
2. Definir/usar los scripted_triggers de clasificación (`gedfrey_is_minor/medium/major`).
3. Por cada decisión: decidir si varía por tipo de país → triplicar o no.
4. Hardcodear `cost`/`days_remove`/`days_re_enable` desde la tabla de diseño.
5. Escribir efectos una vez y copiarlos idénticos en las variantes.
6. Crear TODAS las claves de localización de los IDs generados.
7. Probar en el juego con el log de errores ("Error Dog") abierto; corregir y recargar.

## Errores frecuentes (checklist antes de entregar)

- [ ] ¿Metiste una variable en `days_remove`/`days_re_enable`? → prohibido, hardcodea o triplica.
- [ ] ¿Usaste `>=`/`<=` en `check_variable`? → cámbialo a `> (N-1)`.
- [ ] ¿Condiciones dinámicas en `allowed`? → muévelas a `visible`/`available`.
- [ ] ¿`random_list` en decisión sin `fixed_random_seed = no`? → añádelo.
- [ ] ¿Subiste nivel de idea/trait sin quitar el anterior? → `remove_*` antes de `add_*`.
- [ ] ¿Faltan claves de localización para algún ID? → créalas todas.
- [ ] ¿`supported_version` desactualizado? → ponlo en la rama de prueba.

## Referencias (leer cuando aplique)

- `references/decisions.md` — anatomía completa de decisiones, targeted/state decisions, missions.
- `references/effects_triggers.md` — efectos y triggers de uso común con sintaxis de variable.
- `references/localisation.md` — claves, text icons, loc scripted (cambios 1.15+).
