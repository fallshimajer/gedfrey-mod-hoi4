# Referencia — Decisiones

## Estructura mínima

Categorías en `common/decisions/categories/*.txt`:
```
mi_categoria = { }
```
Decisiones en `common/decisions/*.txt`:
```
mi_categoria = {
    mi_decision = {
        cost = 50
        available = { has_war = yes }
        complete_effect = { add_political_power = -50 ... }
    }
}
```
El nombre del archivo es irrelevante para el juego; usa el tag por convención.

## Argumentos de decisión más usados

- `allowed = { }` — chequeo único (inicio/carga). Restricción por `tag`/`original_tag`/`has_dlc`.
- `visible = { }` — cada frame; aparece en menú.
- `available = { }` — cada frame; si falso se ve gris.
- `cost = <int|var>` — coste en PP. Acepta variable.
- `custom_cost_trigger` + `custom_cost_text` — coste personalizado (NO descuenta solo;
  hay que restar en `complete_effect`, normalmente con `hidden_effect`). Para que la IA
  ahorre PP usa `ai_hint_pp_cost = N` (constante, no variable).
- `days_re_enable = <int>` — cooldown. NO acepta variable.
- `fire_only_once = yes` — una sola vez por país (o por target en decisiones targeted).
- `days_remove = <int>` — timer; `-1` = nunca por timer.
- `remove_effect = { }` — al terminar timer. `remove_trigger = { }` — quita al cumplirse.
- `cancel_trigger` / `cancel_effect` — cancelar timer sin remove_effect.
- `modifier = { }` y `targeted_modifier = { }` — modificadores activos durante el timer.
- `fixed_random_seed = no` — hace que random_list sea realmente aleatorio.
- `priority = <int>` o forma larga con `base` + `modifier` — orden en el menú.
- `ai_will_do = { base = 0 modifier = { add = 10 ... } }` — sin esto la IA NUNCA la toma.
- `icon = nombre` — usa GFX_decision_nombre (categorías: GFX_decision_category_nombre).

## complete_effect vs remove_effect

- `complete_effect` se ejecuta al SELECCIONAR (inicia el timer si hay `days_remove`).
- `remove_effect` se ejecuta cuando el timer TERMINA.
- Para construir un edificio con espera: descuento/marcado en complete_effect, edificio en remove_effect.

## Decisiones targeted (hacia otro país)

Se vuelve targeted si hay `targets`, `target_array` o `target_trigger`.
- `targets = { TAG TAG }`, `targets_dynamic = yes`, `target_non_existing = yes`.
- `target_root_trigger = { }` — chequea solo ROOT a diario (optimización).
- `target_trigger = { }` — chequea ROOT (default) y FROM (el target). OJO: FROM es el target.
- Guerra: `war_with_target_on_complete = yes` (equivale a `war_with_on_complete = FROM`).

## Decisiones targeted a estado

`state_target = yes | any | any_owned_state | any_controlled_state | <continente>`.
- FROM pasa a ser scope de estado.
- `on_map_mode = map_only | decision_view_only | map_and_decisions_view`.

## Missions (variante de decisión)

- `days_mission_timeout = N` convierte la decisión en misión.
- `timeout_effect = { }` — al expirar el timer.
- `activation = { }` — triggers para que aparezca (diario). `visible` NO sirve en misiones.
- `selectable_mission = yes` — requiere que el jugador pulse; si no, dispara al cumplir `available`.
- Mejor activarlas con `activate_mission = nombre` que dejarlas automáticas.
