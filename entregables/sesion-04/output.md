# Backlog FlowSync MVP — User Stories

> Descomposición del PRD.md de FlowSync, limitada estrictamente a las funcionalidades listadas en "Qué incluye el MVP" (sección 1). No se incluyen funcionalidades del apartado "Qué NO incluye el MVP", no se estima, y no se propone arquitectura.
> Marcas usadas: **(asumido)** = supuesto no evidenciado explícitamente en el PRD. **(ambiguo)** = el propio PRD deja el punto sin definir o con doble interpretación, con referencia a la sección correspondiente.

---

## Módulo 1: Autenticación y gestión de cuenta

### US1 — Registro de cuenta
**COMO** visitante
**QUIERO** crear una cuenta con mi email y contraseña
**PARA** poder acceder a FlowSync y empezar a gestionar mis tareas

**Criterios de Aceptación:**
1. Given soy un visitante sin cuenta, When me registro con un email no utilizado previamente y una contraseña de al menos 8 caracteres, Then la cuenta se crea, And quedo autenticado, And soy dirigido a la pantalla de bienvenida.
2. Given soy un visitante, When intento registrarme con una contraseña de menos de 8 caracteres, Then el registro es rechazado, And se muestra un mensaje de error comprensible indicando el requisito de longitud mínima.
3. Given el email que intento usar ya está registrado, When envío el formulario de registro, Then el sistema indica que el email ya está en uso, And me ofrece la opción de ir a la pantalla de inicio de sesión.
4. Given completo el registro exitosamente, When llego a la pantalla de bienvenida, Then veo una frase que explica qué hace FlowSync, And una invitación a crear mi primera tarea.
5. (asumido — el PRD 3.1 no especifica el formato de validación de email) Given intento registrarme con un email con formato inválido, When envío el formulario, Then el registro es rechazado, And se muestra un mensaje de error comprensible.

### US2 — Inicio de sesión
**COMO** usuario registrado
**QUIERO** iniciar sesión con mi email y contraseña
**PARA** acceder a FlowSync y gestionar mis tareas

**Criterios de Aceptación:**
1. Given mi cuenta está registrada, When inicio sesión con mi email y contraseña correctos, Then quedo autenticado, And soy dirigido al listado de mis tareas.
2. Given mi cuenta está registrada, When inicio sesión con la contraseña incorrecta, Then el acceso es rechazado, And se muestra un mensaje indicando que las credenciales no son válidas.
3. Given intento iniciar sesión con un email no registrado, When envío el formulario, Then el acceso es rechazado, And se muestra el mismo mensaje genérico de credenciales inválidas (asumido — el PRD no especifica si el mensaje distingue email inexistente de contraseña incorrecta).
4. (asumido — el PRD no define expiración/duración del token de acceso) Given mi sesión se mantiene mediante un token de acceso, When reabro FlowSync dentro de la vigencia del token, Then continúo autenticado sin volver a iniciar sesión.

### US3 — Cierre de sesión
**COMO** usuario autenticado
**QUIERO** cerrar mi sesión
**PARA** proteger el acceso a mis tareas cuando termino de usar FlowSync

**Criterios de Aceptación:**
1. Given estoy autenticado, When solicito cerrar sesión, Then mi token de acceso deja de ser válido, And soy redirigido a la pantalla de inicio de sesión.
2. Given cerré sesión, When intento acceder a una ruta protegida (por ejemplo, mi listado de tareas) usando el token anterior, Then el acceso es rechazado, And soy redirigido a iniciar sesión.

---

## Módulo 2: Gestión de tareas (CRUD)

### US4 — Crear tarea
**COMO** usuario autenticado
**QUIERO** crear una tarea indicando al menos un título
**PARA** registrar mis pendientes en FlowSync

**Criterios de Aceptación:**
1. Given estoy autenticado, When creo una tarea indicando sólo el título, Then la tarea se guarda con estado `pending`, And aparece en mi listado de tareas.
2. Given estoy autenticado, When intento crear una tarea sin título, Then la creación es rechazada, And se muestra un mensaje de error comprensible indicando que el título es obligatorio.
3. Given estoy autenticado, When creo una tarea indicando título, descripción y fecha límite, Then la tarea se guarda con esos tres datos, And su estado inicial es `pending`.
4. Given tengo mi cuenta de Google conectada, When creo una tarea con fecha límite, Then se dispara la sincronización que crea el evento correspondiente en mi Google Calendar (ver US12).

### US5 — Consultar listado de tareas
**COMO** usuario autenticado
**QUIERO** ver el listado de mis tareas
**PARA** saber qué tengo pendiente

**Criterios de Aceptación:**
1. Given tengo tareas creadas, When accedo a mi listado, Then veo todas mis tareas con título, estado y fecha límite (cuando la tiene).
2. Given no tengo ninguna tarea (cuenta nueva o todas archivadas), When accedo a mi listado, Then veo un estado vacío con una invitación a crear mi primera tarea.
3. (ambiguo — PRD 3.3: "el criterio exacto de ordenación queda a decisión del equipo durante el refinamiento") Given tengo varias tareas con distintas fechas límite, When accedo a mi listado sin aplicar filtros, Then las tareas se muestran en un orden donde las más relevantes para "hoy" aparecen primero, sin que el PRD defina el criterio exacto de ordenación.
4. Given mis tareas pertenecen únicamente a mi cuenta, When accedo a mi listado, Then no veo tareas de ningún otro usuario (privacidad, sección 4).

### US6 — Editar tarea
**COMO** usuario autenticado
**QUIERO** editar cualquier campo de una tarea existente
**PARA** mantener mis datos actualizados

**Criterios de Aceptación:**
1. Given tengo una tarea existente, When edito su título, descripción o fecha límite, Then los cambios se guardan, And se reflejan en el listado.
2. Given edito el título de una tarea dejándolo vacío, When intento guardar, Then la edición es rechazada, And se muestra un mensaje de error indicando que el título es obligatorio.
3. Given cambio la fecha límite de una tarea y tengo Google conectado, When guardo el cambio, Then el evento correspondiente en Google Calendar se actualiza (ver US12).
4. (asumido — el PRD no detalla el código/mensaje de error exacto) Given intento editar una tarea que no me pertenece, When envío la edición, Then la operación es rechazada, en cumplimiento de la privacidad entre usuarios (sección 4).

### US7 — Borrar tarea
**COMO** usuario autenticado
**QUIERO** borrar una tarea
**PARA** eliminar pendientes que ya no necesito

**Criterios de Aceptación:**
1. Given tengo una tarea existente, When la borro, Then desaparece de mi listado de tareas.
2. (ambiguo — PRD 3.5: "el evento correspondiente en Google Calendar se elimina o se marca según corresponda", sin especificar cuál de las dos opciones aplica al borrado) Given la tarea borrada tenía un evento asociado en Google Calendar y tengo Google conectado, When la borro, Then el evento correspondiente se elimina o se marca según corresponda.
3. Given intento borrar una tarea que no me pertenece, When envío la solicitud, Then la operación es rechazada (privacidad, sección 4).

### US8 — Cambiar el estado de una tarea
**COMO** usuario autenticado
**QUIERO** cambiar el estado de una tarea
**PARA** reflejar si está pendiente, completada o archivada

**Criterios de Aceptación:**
1. Given tengo una tarea en `pending`, When la marco como `completed`, Then su estado cambia, And se refleja en el listado.
2. Given tengo una tarea en `pending` o `completed`, When la marco como `archived`, Then su estado cambia a `archived`.
3. (ambiguo — misma referencia que US7, PRD 3.5) Given completo una tarea que tiene un evento asociado en Google Calendar y tengo Google conectado, When cambio su estado a `completed`, Then el evento se elimina o se marca según corresponda.
4. (asumido — el PRD no detalla el manejo de estados fuera del enum definido, se infiere de la validación estándar) Given intento asignar un estado distinto de `pending`, `completed` o `archived`, When envío la solicitud, Then la operación es rechazada, And se muestra un error de validación.

---

## Módulo 3: Organización y filtrado

### US9 — Filtrar tareas por estado
**COMO** usuario autenticado
**QUIERO** filtrar mis tareas por estado
**PARA** ver sólo las que me interesan en cada momento

**Criterios de Aceptación:**
1. Given tengo tareas en distintos estados, When filtro por `pending`, Then sólo veo las tareas pendientes.
2. Given tengo tareas en distintos estados, When filtro por `completed`, Then sólo veo las tareas completadas.
3. Given tengo tareas en distintos estados, When filtro por `archived`, Then sólo veo las tareas archivadas.
4. (asumido — PRD 3.3 sólo describe el estado vacío general "sin ninguna tarea", no el caso de un filtro sin resultados) Given aplico un filtro y no tengo tareas en ese estado, When veo el resultado, Then veo un estado vacío correspondiente a ese filtro.

---

## Módulo 4: Exportación

### US10 — Exportar tareas a CSV
**COMO** usuario autenticado
**QUIERO** exportar mis tareas a un archivo CSV
**PARA** tener mis datos disponibles fuera de FlowSync

**Criterios de Aceptación:**
1. Given tengo tareas creadas, When solicito la exportación, Then recibo un archivo CSV que incluye, para cada tarea, título, descripción, estado y fecha límite.
2. (asumido — el PRD no especifica el comportamiento cuando no hay tareas) Given no tengo ninguna tarea, When solicito la exportación, Then recibo un archivo CSV válido que contiene sólo la fila de encabezados.
3. (asumido — no evidenciado en el PRD, es una práctica estándar de generación de CSV) Given mis tareas incluyen comas, comillas o saltos de línea en el título o la descripción, When exporto a CSV, Then esos caracteres quedan correctamente escapados y el archivo resultante sigue siendo un CSV válido.

---

## Módulo 5: Sincronización con Google Calendar

### US11 — Conectar y desconectar cuenta de Google
**COMO** usuario autenticado
**QUIERO** conectar y desconectar mi cuenta de Google
**PARA** controlar cuándo FlowSync sincroniza mis tareas con mi Google Calendar

**Criterios de Aceptación:**
1. Given estoy autenticado y no tengo Google conectado, When inicio y completo con éxito el flujo de autorización OAuth, Then mi cuenta de Google queda conectada a FlowSync.
2. (asumido — el PRD no detalla el mensaje exacto ante un rechazo de OAuth) Given estoy en medio del flujo OAuth, When cancelo o rechazo el permiso en la pantalla de Google, Then mi cuenta de Google no queda conectada, And se muestra un mensaje indicando que la conexión no se completó.
3. Given tengo mi cuenta de Google conectada, When me desconecto, Then FlowSync deja de sincronizar mis tareas con Google Calendar, And las tareas ya creadas en FlowSync no se borran.
4. Given desconecté mi cuenta de Google, When creo o edito una tarea con fecha límite, Then la tarea se guarda en FlowSync, And no se crea ni se actualiza ningún evento en Google Calendar.

### US12 — Sincronización de tareas como eventos en Google Calendar
**COMO** usuario con Google Calendar conectado
**QUIERO** que mis tareas con fecha límite se reflejen automáticamente como eventos
**PARA** no tener que copiar manualmente mis pendientes al calendario (Glosario, sección 8: Sincronización)

**Criterios de Aceptación:**
1. Given tengo Google conectado, When creo una tarea con fecha límite, Then se crea un evento correspondiente en mi Google Calendar.
2. Given tengo Google conectado y una tarea con un evento asociado, When cambio la fecha límite de la tarea, Then el evento en Google Calendar se actualiza a la nueva fecha.
3. (ambiguo — PRD 3.5 no especifica cuál de las dos acciones, eliminar o marcar, corresponde a cada caso) Given tengo Google conectado y una tarea con un evento asociado, When completo o borro la tarea, Then el evento correspondiente se elimina o se marca según corresponda.
4. (asumido — el PRD no define la política de reintentos: número de intentos, tiempos de espera, backoff) Given la API de Google Calendar no está disponible o devuelve un error al intentar sincronizar, When ocurre el fallo, Then la tarea igualmente queda guardada en FlowSync, And la operación de sincronización se reintenta más tarde.
5. Given se ejecuta una operación de sincronización (exitosa o fallida), When ocurre, Then queda registrada en logs para su diagnóstico (sección 4, Observabilidad).

---

## Resumen
- **12 User Stories** agrupadas en 5 módulos, todas derivadas exclusivamente de la sección "Qué incluye el MVP" del PRD.
- No se incluyen tareas compartidas/equipos, sincronización inversa (Google → FlowSync), notificaciones, app móvil nativa, etiquetas/proyectos/subtareas ni recordatorios configurables, por estar explícitamente fuera de alcance (sección "Qué NO incluye el MVP").
- Puntos marcados **(ambiguo)**: orden del listado (US5), y el criterio "eliminar vs. marcar" el evento de Calendar al completar/borrar una tarea (US7, US8, US12) — ambos remiten a redacciones del PRD que dejan la decisión abierta al equipo.
- No se incluyen estimaciones ni propuestas de arquitectura, conforme a las restricciones del prompt.
