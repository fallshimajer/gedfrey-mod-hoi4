# Referencia — Efectos, Triggers y Variables

## Distinción clave

- **Triggers (condiciones)**: se evalúan, devuelven verdadero/falso. Van en `allowed`,
  `visible`, `available`, `limit`, `trigger`, etc. NO pueden ejecutar efectos.
- **Efectos (comandos)**: cambian el estado del juego una vez. Van en `complete_effect`,
  `remove_effect`, `completion_reward`, `option`, `effect`, etc.
- Muchos efectos aceptan variable donde el wiki marca `<variable>`, p. ej.
  `add_manpower = var:mi_var`.

## Variables

- `set_variable = { var = valor }` — fija. `add_to_variable`, `subtract_from_variable`,
  `multiply_variable`, `divide_variable`.
- `set_temp_variable` / `*_temp_variable` — variables temporales dentro del mismo efecto.
- Lectura en triggers: `check_variable = { var > N }` (operadores solo `>`, `<`, `=`).
- Puedes leer game-variables en una variable propia:
  `set_variable = { x = num_of_civilian_factories }` (verifica disponibilidad por versión).
- `has_variable = nombre` — comprueba que exista.

## Triggers de uso común

- País: `tag = USA`, `original_tag = POL`, `has_government = monarchy`, `has_dlc = "..."`.
- Estabilidad/apoyo: `has_stability > 0.5`, `has_war_support > 0.5` (decimales: usa el valor justo).
- Fábricas: `num_of_civilian_factories > N`, `num_of_military_factories > N`,
  `num_of_naval_factories > N` (NO hay trigger directo de suma civ+mil).
- Guerra: `has_war = yes`, `has_war_with = TAG`, `is_in_faction = yes`.
- Líder: `has_country_leader_with_trait = X`.
- Banderas: `has_country_flag = X`, `has_state_flag = X`.
- IA/humano: `is_ai = yes/no`.
- Fechas: `date > 1940.1.1`.

## Efectos de uso común

- PP: `add_political_power = N` (negativo para restar).
- Manpower: `add_manpower = N`.
- Ideas: `add_ideas = X`, `remove_ideas = X`, `add_timed_idea = { idea = X days = N }`.
- Traits de líder: `add_country_leader_trait = X`, `remove_country_leader_trait = X`.
- Banderas: `set_country_flag = X`, `clr_country_flag = X`, `set_country_flag = { flag = X days = N }`.
- Edificios: `add_building_construction = { type = industrial_complex level = 1 instant_build = yes }`.
- Recursos por estado: `add_resource = { type = aluminium amount = 2 state = 123 }`
  (en scope de estado suele ser `add_resource = { type = aluminium amount = 2 }`).
- Eventos: `country_event = { id = gedfrey.economic.1 days = 0 }`.

## Scopes y bucles

- `every_owned_state`, `random_owned_state`, `any_owned_state` (trigger).
- `every_country`, `random_country`, `all_countries`.
- `prioritize` dentro de un scope ordena la selección.
- En `random_owned_state` usa `limit = { ... }` para filtrar (p. ej. slots libres):
  `free_building_slots = { building = industrial_complex size > 0 }`.

## Scripted triggers y scripted effects

`common/scripted_triggers/*.txt`:
```
gedfrey_is_major = { OR = { tag = USA tag = SOV tag = GER tag = ENG tag = JAP } }
```
`common/scripted_effects/*.txt`: bloques reutilizables de efectos. Úsalos para no repetir
el mismo `complete_effect` en las 3 variantes de una decisión triplicada:
```
gedfrey_build_civ_effect = { random_owned_state = { limit = { ... } add_building_construction = { ... } } }
```
Luego en cada variante: `remove_effect = { gedfrey_build_civ_effect = yes }`.
