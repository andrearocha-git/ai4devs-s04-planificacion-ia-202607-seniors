# Poke-holes — US4 (Crear tarea)

> Ejercicio: pedirle a la IA que critique una de sus propias user stories del backlog (`output.md`). Story analizada: **US4 — Crear tarea**. No se reescribió la story, sólo se listan gaps, supuestos implícitos, escenarios faltantes y riesgos.

```
### US4 — Crear tarea
COMO usuario autenticado
QUIERO crear una tarea indicando al menos un título
PARA registrar mis pendientes en FlowSync

Criterios de Aceptación:
1. Given estoy autenticado, When creo una tarea indicando sólo el título, Then la tarea se guarda con estado `pending`, And aparece en mi listado de tareas.
2. Given estoy autenticado, When intento crear una tarea sin título, Then la creación es rechazada, And se muestra un mensaje de error comprensible indicando que el título es obligatorio.
3. Given estoy autenticado, When creo una tarea indicando título, descripción y fecha límite, Then la tarea se guarda con esos tres datos, And su estado inicial es `pending`.
4. Given tengo mi cuenta de Google conectada, When creo una tarea con fecha límite, Then se dispara la sincronización que crea el evento correspondiente en mi Google Calendar (ver US12).
```

## Edge cases no cubiertos
- Título compuesto sólo por espacios en blanco (`"   "`) — ¿cuenta como "sin título" (AC2) o se acepta tal cual?
- Longitud máxima de título/descripción no definida — ¿qué pasa con un título de 5000 caracteres?
- Fecha límite en el pasado — ¿se permite crear una tarea ya "vencida"? Ningún AC lo contempla.
- Doble envío del formulario (doble clic / reintento de red) — riesgo de tareas duplicadas, sin AC de idempotencia.
- Título con HTML/markdown/emojis — no se especifica sanitización, relevante porque el frontend es React y podría renderizar contenido sin escapar (riesgo XSS).
- Fecha límite con formato inválido o fuera de rango (ej. año 9999) — no cubierto.

## Supuestos implícitos
- Se asume que "estoy autenticado" es una precondición dada, pero ningún AC cubre el caso de un usuario NO autenticado intentando crear una tarea (debería rechazarse con 401, análogo a lo que sí se hizo en US3).
- Se asume que la creación es exitosa e inmediata; no se define contrato de respuesta (¿se devuelve la tarea creada? ¿un ID?).
- Se asume que "fecha límite" es solo fecha (sin hora) o fecha+hora — el PRD (sección 7, riesgo de zonas horarias) señala esto como punto crítico y no está resuelto aquí ni en el AC4.
- Se asume que no hay restricción de unicidad de título (se pueden crear tareas duplicadas) — no está explícito en ningún AC.

## Escenarios faltantes
- ¿Qué pasa si se crea una tarea con fecha límite pero el usuario NO tiene Google conectado? (falta el AC "negativo" complementario al AC4 — la tarea debería guardarse igual sin intentar sincronizar).
- Feedback visual/UX de creación exitosa (confirmación, redirección al listado) no está verificado en ningún AC.
- Comportamiento cerca del límite operativo mencionado en el PRD (NFR: hasta 200 tareas) — no hay AC que valide creación en ese escenario.
- Creación de tarea sin descripción explícitamente vacía vs. `null` — no se distingue.

## Dependencias y riesgos no mencionados
- AC4 depende de US11 (conexión Google) y US12 (lógica de sincronización), pero no menciona qué pasa si el token de Google expiró o es inválido justo en el momento de creación — ¿la tarea se crea igual y queda "pendiente de sync" o falla toda la operación? Ese comportamiento está descrito en otra US (US12 AC4) pero no se referencia aquí como dependencia explícita.
- Riesgo de zona horaria (PRD sección 7): la creación del evento en Calendar (AC4) hereda toda la ambigüedad de cómo se traduce "fecha límite" a un horario concreto — este AC oculta ese riesgo detrás de "se dispara la sincronización" sin marcarlo como abierto.
- Race condition: el usuario crea la tarea y desconecta Google casi simultáneamente — no está definido qué versión del estado de conexión "gana".
- No hay AC de rendimiento/latencia para la operación de creación (el PRD sí define uno para el listado, sección 4, pero no para creación).
