# Gedfrey Mod — Dev Log
Registro de cambios aplicados, errores encontrados e intentos fallidos durante el desarrollo.
HOI4 versión objetivo: 1.14.x (Götterdämmerung)

---

## PROBLEMAS RAÍZ ENCONTRADOS

### 1. Carpeta de categorías de decisiones incorrecta
**Archivo:** `common/decision_categories/gedfrey_categories.txt`  
**Error:** HOI4 1.14 carga las categorías desde `common/decisions/categories/`, NO desde `common/decision_categories/`.  
**Síntoma:** Todas las decisiones aparecían como "Unknown category" y eran ignoradas. El mod cargaba pero ninguna decisión funcionaba.  
**Fix:** Crear `common/decisions/categories/gedfrey_categories.txt` y mover el contenido ahí.

---

### 2. Operador `>=` roto para triggers en HOI4 1.14
**Archivos:** `common/decisions/*.txt`, `common/scripted_triggers/*.txt`  
**Error:** HOI4 tokeniza `>=` como dos tokens separados (`>` y `=`) en comparaciones de enteros y fechas. El parser falla silenciosamente o lanza errores en cascada.  
**Síntoma:** `num_of_civilian_factories >= 30` → `Invalid trigger '30'` + `Invalid trigger '='`. `date >= 1936.1.31` → `Wrong format in trigger; Y.M.D should be used`.  
**Fix:** Reemplazar `>= X` por `> X-1` en todos los archivos. Ejemplo: `>= 30` → `> 29`, `>= 1936.1.31` → `> 1936.1.30`.  
**Nota:** `>=` SÍ funciona para valores float en algunos contextos (ej. `stability >= 0.25` en vanilla), pero en esta instalación también falla.

---

### 3. `cost = nombre_scripted_value` no funciona en HOI4 1.14
**Archivos:** Todos los archivos de decisiones  
**Error:** El campo `cost` en decisiones de HOI4 1.14 solo acepta números directos. Las referencias a scripted values se ignoran silenciosamente → costo = 0.  
**Síntoma:** Todas las decisiones aparecían sin costo de PP (gratis).  
**Fix:** Hardcodear el costo directamente: `cost = 100`.

---

### 4. `days_re_enable = nombre_scripted_value` no funciona en HOI4 1.14
**Archivos:** Todos los archivos de decisiones  
**Error:** Mismo problema que el campo `cost` — el campo `days_re_enable` ignora referencias a scripted values → cooldown = 0 días.  
**Síntoma:** Las decisiones se podían tomar infinitamente sin cooldown.  
**Fix:** Hardcodear el valor directamente: `days_re_enable = 30`.

---

### 5. Scripted values referenciando scripted triggers — problema de orden de carga
**Archivo:** `common/scripted_values/gedfrey_scripted_values.txt`  
**Error:** HOI4 carga los scripted values ANTES que los scripted triggers. Las condiciones `gedfrey_is_medium = yes` dentro de los modifiers fallaban porque el trigger aún no existía. Cuando todos los modifiers de un valor fallan, el valor entero se descarta → retorna 0.  
**Síntoma:** Aunque los scripted values cargaban sin errores en el log, su resultado siempre era 0.  
**Solución intentada:** Reemplazar referencias a scripted triggers por tags directos (`OR = { tag = FRA tag = ITA ... }`). Funcionó sintácticamente pero el campo `cost` igualmente ignoraba el valor (problema #3).  
**Fix final:** Valores hardcodeados directamente en cada decisión.

---

### 6. Modifiers inválidos en leader traits
**Archivo:** `common/country_leader/gedfrey_leader_traits.txt`  

| Modifier inválido | Motivo | Reemplazo |
|---|---|---|
| `mining_resources_factor` | No existe en HOI4 | 6 factores individuales → luego `local_resources_factor` |
| `manpower_recovery_speed` | No válido en scope de leader traits | `monthly_population` |
| `construction_speed` | Nombre incorrecto | `production_speed_buildings_factor` |
| `oil_factor`, `steel_factor`, etc. | No válidos en scope de leader traits | `local_resources_factor` |
| `weekly_war_support` | No válido en leader traits | `war_support_factor` |

---

### 7. `has_government = monarchy` valor inválido
**Archivo:** `common/decisions/gedfrey_leader_decisions.txt`  
**Error:** `has_government` en HOI4 acepta ideology groups (`democratic`, `fascism`, `communism`, `neutrality`), no formas de gobierno como `monarchy`.  
**Síntoma:** Error `"invalid database object: monarchy"` en el log.  
**Fix:** Reemplazado por `has_government = neutrality` (aproximación para monarquías neutrales).

---

### 8. `add_maneuvering = 1` efecto inválido en HOI4 1.14
**Archivo:** `common/decisions/gedfrey_military_decisions.txt`  
**Error:** En HOI4 Götterdämmerung (1.14), la habilidad "maneuvering" de comandantes fue renombrada.  
**Síntoma:** `Unknown effect-type: add_maneuvering`.  
**Fix:** Reemplazado por `add_planning = 1`.

---

### 9. `stability` y `war_support` como triggers inválidos
**Archivo:** `common/decisions/gedfrey_leader_decisions.txt`  
**Error:** Causa desconocida — probablemente conflicto con otro mod activo (ugc_*.mod) que redefine estas variables, o cambio en HOI4 1.14. Ambos triggers se marcan como `Invalid trigger` aunque son válidos en vanilla.  
**Síntoma:** Political Consolidation y Royal Grace nunca disponibles.  
**Fix:** Eliminados los requisitos de `stability` y `war_support` de los bloques `available`. Las decisiones quedan disponibles solo con la cadena de traits previa.

---

### 10. `priority = { base = 100 }` inválido en decision categories
**Archivo:** `common/decisions/categories/gedfrey_categories.txt`  
**Error:** El campo `priority` en categorías de decisiones solo acepta números directos. La sintaxis de scripted value block es inválida ahí.  
**Síntoma:** Las categorías se parseaban pero no se registraban → todas las decisiones aparecían como "Unknown category".  
**Fix:** `priority = { base = 100 }` → `priority = 100`.

---

### 11. Archivo `.mod` externo faltante
**Archivo:** `gedfrey-mod-hoi4.mod` (en la carpeta `mod/`)  
**Error:** El launcher de HOI4 solo carga mods que tienen un archivo `.mod` externo apuntando a la carpeta. Sin ese archivo, el mod no era visible en el launcher.  
**Fix:** Crear `gedfrey-mod-hoi4.mod` con el path correcto.

---

## INTENTOS FALLIDOS

### Intento 1 — Inline blocks para costos escalados
**Código intentado:**
```
cost = {
    base = 75
    modifier = { factor = 1.2  gedfrey_is_medium = yes }
    modifier = { factor = 1.5  gedfrey_is_major = yes  }
}
```
**Por qué falló:** HOI4 no reconoce el bloque inline como el campo `cost` de la decisión. Lo interpreta como la definición de una sub-decisión llamada `cost`. Resultado: `days_re_enable` y `days_remove` aparecían como decisiones sueltas en el UI.

---

### Intento 2 — Factores de recursos individuales en leader traits
**Código intentado:** `oil_factor = 0.05`, `steel_factor = 0.05`, etc.  
**Por qué falló:** Estos modifiers son de scope de estado, no de scope de líder de país. HOI4 los marca como "Unknown modifier" y los ignora.

---

### Intento 3 — Scripted values con tags inline para costos
**Código intentado:** Reescribir `gedfrey_scripted_values.txt` reemplazando `gedfrey_is_medium = yes` por `OR = { tag = FRA tag = ITA ... }`.  
**Por qué falló:** El problema no era el orden de carga de los triggers, sino que el campo `cost` en HOI4 1.14 no resuelve referencias a nombres de scripted values en absoluto. La solución correcta para las referencias fue innecesaria — el problema raíz era diferente.

---

### Intento 4 — `stability > 0.24` como alternativa a `stability >= 0.25`
**Por qué falló:** El operador `>` tampoco funciona para el trigger `stability` en esta instalación. El trigger `stability` en sí mismo es marcado como inválido, independiente del operador.

---

## ESTADO ACTUAL DEL MOD

| Sistema | Estado |
|---|---|
| Leader Directives (Industrial Power, Political Consolidation, Doctrine of War) | ✓ Funciona |
| Royal Grace | ✓ Funciona (solo para gobiernos neutrality) |
| Military Development (Individual Training, Reform, Special Talent, Supreme Command) | ✓ Funciona — 10 días de ejecución + cooldown |
| Economic Development (Construir fábricas) | ✓ Funciona — 30 días construcción + cooldown |
| Costo PP por tier (minor/medium/major) | ✗ No implementado — `cost` solo acepta números directos en HOI4 1.14 |
| Cooldown por tier (minor/medium/major) | ✗ No implementado — mismo problema |
| Requisitos de estabilidad (Political Consolidation) | ✗ Removido — trigger `stability` falla en esta instalación |
| Resource exploration (Phase 4) | ✗ Pendiente de implementación |
