# Referencia — Localización

## Reglas básicas

- Archivos en `localisation/<idioma>/<nombre>_l_<idioma>.yml`.
- Primera línea: `l_english:` (con dos puntos). Codificación UTF-8 con BOM.
- Cada entrada: ` clave:0 "Texto"` (espacio inicial, el `:0` es la versión, opcional pero recomendado).
- Toda ID visible necesita clave (decisión, idea, trait, evento, opción), o sale `[clave]`.
- Descripción = clave + `_desc`.

```
l_english:
 gedfrey_industry_1_minor:0 "Potencia Industrial I"
 gedfrey_industry_1_minor_desc:0 "Otorga al líder el rasgo de potencia industrial (nivel 1)."
```

## Claves por tipo

- Decisión: `<id>` y `<id>_desc`. Categoría: `<cat>` y `<cat>_desc`.
- Idea: `<id>` y `<id>_desc`.
- Trait de líder: `<id>` y `<id>_desc`.
- Evento: `<id>.t` (título), `<id>.d` (descripción), `<id>.a` (opción A), `<id>.b`, etc.

## Si triplicas decisiones

Crea clave para CADA variante aunque el texto sea igual:
```
 gedfrey_build_civ_minor:0 "Construir Fábrica Civil"
 gedfrey_build_civ_minor_desc:0 "Construye una fábrica civil en el mejor estado disponible."
 gedfrey_build_civ_medium:0 "Construir Fábrica Civil"
 gedfrey_build_civ_medium_desc:0 "Construye una fábrica civil en el mejor estado disponible."
 gedfrey_build_civ_major:0 "Construir Fábrica Civil"
 gedfrey_build_civ_major_desc:0 "Construye una fábrica civil en el mejor estado disponible."
```

## Text icons y variables en texto

- Iconos: `£political_power`, `£command_power`, etc. dentro del string.
- Color: `§Y...§!` (amarillo), `§R...§!` (rojo), `§G...§!` (verde).
- Variables: `[?gedfrey_total_factories]` muestra el valor de la variable.
- custom_cost_text necesita 3 claves: `<clave>`, `<clave>_blocked`, `<clave>_tooltip`.

## Cambios 1.15+

Desde 1.15 cambió el funcionamiento de la localización scripted, reduciendo la cantidad
de loc manual necesaria para algunos tooltips dinámicos. Si un mod viejo definía mucha loc
scripted a mano, parte puede simplificarse. Revisa el comportamiento en la versión objetivo
antes de copiar patrones de mods antiguos.
