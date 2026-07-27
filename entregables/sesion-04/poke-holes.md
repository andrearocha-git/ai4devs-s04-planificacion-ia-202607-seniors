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
- Fecha límite en el pasado — ¿se permite crear una tarea ya "vencida"? Ningún AC lo contempla.
- Título con HTML/markdown/emojis — no se especifica sanitización, relevante porque el frontend es React y podría renderizar contenido sin escapar (riesgo XSS).

## Supuestos implícitos
- Se asume que "fecha límite" es solo fecha (sin hora) o fecha+hora — el PRD (sección 7, riesgo de zonas horarias) señala esto como punto crítico y no está resuelto aquí ni en el AC4.

## Escenarios faltantes
- ¿Qué pasa si se crea una tarea con fecha límite pero el usuario NO tiene Google conectado? (falta el AC "negativo" complementario al AC4 — la tarea debería guardarse igual sin intentar sincronizar).

## Dependencias y riesgos no mencionados
- Riesgo de zona horaria (PRD sección 7): la creación del evento en Calendar (AC4) hereda toda la ambigüedad de cómo se traduce "fecha límite" a un horario concreto — este AC oculta ese riesgo detrás de "se dispara la sincronización" sin marcarlo como abierto.
