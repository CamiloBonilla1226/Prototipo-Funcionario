# CLAUDE.md — Panel del Funcionario Académico (ACBB)

Contexto para Claude Code al trabajar en esta carpeta (`funcionario.html`).

## 1. Qué es este proyecto

Prototipo de aplicación web para la gestión de los procesos académicos de
**Cancelación de Matrícula**, **Cancelación de Asignatura** y **Examen
Supletorio** — FIET, Universidad del Cauca. Trabajo de grado de Andersson
Camilo Bonilla Belalcázar.

Mockup HTML de archivo único, estático e interactivo. No hay backend real:
los datos (`DATA`) están hardcodeados, y el paso de una solicitud entre
`estudiante.html`, `funcionario.html` y `decano.html` se simula con
`localStorage` como puente de una sola vez.

Actores del sistema: **Estudiante**, **Funcionario Académico**, **Decano**.
Escribe siempre estos nombres con esta capitalización exacta.

Esta carpeta cubre **únicamente el panel del Funcionario Académico**. Los
paneles de Estudiante y Decano viven en otras carpetas, cada una con su
propio `CLAUDE.md`; no los edites desde aquí.

## 2. DECISIÓN CLAVE — la Resolución es un documento digital (PDF), solo para CM y CA

**Actualizado 2026-09-30**: esta sección reemplaza la decisión anterior ("la
Resolución es 100% física"). La regla de negocio cambió y fue validada con
el usuario (Andersson Camilo Bonilla Belalcázar): la Resolución de
Cancelación de Matrícula y Cancelación de Asignatura ahora **sí** es un
documento digital en PDF que el Funcionario **adjunta** desde la
aplicación (nunca lo redacta ni lo genera ahí), tanto si la decisión final
es **Aprobada** como si es **Rechazada**. Examen Supletorio sigue **sin
generar Resolución nunca**.

- **Cuándo se pide la Resolución** (nunca antes de que exista una decisión):
  - Al **rechazar directamente** una solicitud `Pendiente` (`abrirRechazo`,
    sin pasar por el Decano) — CM/CA únicamente.
  - Al **enviar la respuesta al Estudiante** después de la decisión del
    Decano (`abrirRespuesta`, estado `Pendiente Notificar`) — CM/CA
    únicamente, tanto si el Decano aprobó como si rechazó.
  - **Nunca** en "Remitir al Decano" (`abrirRemision`): en ese punto del
    trámite el Decano todavía no ha decidido, así que la Resolución no
    puede existir todavía — esto no cambió.
  - **Nunca** para Examen Supletorio, en ningún paso.
- **Cómo se aporta el documento**: únicamente adjuntando un archivo PDF ya
  existente (zona de carga, campo obligatorio, bloquea el envío si falta).
  No hay ninguna opción para redactar/generar la Resolución desde la
  aplicación — el documento siempre lo produce el Decano/Decanatura fuera
  del sistema. El nombre del archivo adjuntado se guarda en `r.resolucion`.
- **Detalle de la solicitud**: la tarjeta "Resolución" se muestra siempre
  que `r.resolucion` exista (CM/CA, aprobada o rechazada), con botones
  Visualizar/Descargar — igual que los demás documentos adjuntos del
  detalle. Para Examen Supletorio esa tarjeta nunca aparece.
- Los mensajes al Estudiante y los toasts de confirmación mencionan que la
  Resolución queda archivada en Decanatura y en DARCA, igual que antes,
  pero ahora como documento digital disponible para descarga, no solo como
  trámite físico.

## 3. Modelo de estados (relativos al Funcionario)

El sistema guarda dos campos internos — `Responsable Actual` y `Decisión` —
de los que se deriva la etiqueta visible.

| Estado | Significado | Acción esperada del Funcionario |
|---|---|---|
| Pendiente | La solicitud acaba de llegar del Estudiante | Revisar y remitir al Decano, o rechazar |
| En Gestión | No es su turno (espera del Decano o, en Examen Supletorio, espera de que el Estudiante pague) | Ninguna; solo seguimiento |
| Pendiente Notificar | El Decano ya decidió y devolvió el trámite | Enviar la respuesta al Estudiante (cierra como `Respondida`) |
| Enviar Recibo | *(solo Examen Supletorio)* El Decano aprobó | Enviar el recibo de pago al Estudiante (pasa a `En Gestión`, `esperaDe: 'Estudiante'`, a la espera del pago) |
| Recibo Pagado | *(solo Examen Supletorio)* El Estudiante subió el comprobante de pago | Verificar el comprobante: aprobar (notifica y cierra) o rechazar (exige motivo y cierra) |
| Respondida | Trámite cerrado | Ninguna; solo consulta |

- Cancelación de Matrícula y Cancelación de Asignatura: `Pendiente`,
  `En Gestión`, `Pendiente Notificar`, `Respondida`. Nunca pasan por
  `Enviar Recibo` ni `Recibo Pagado`.
- Examen Supletorio: `Pendiente`, `En Gestión`, `Respondida`, y en vez de un
  único `Pendiente Notificar` genérico se bifurca según la decisión del
  Decano: si **rechaza**, usa `Pendiente Notificar` igual que Cancelación;
  si **aprueba**, usa `Enviar Recibo` y luego `Recibo Pagado` (nunca cierra
  directamente a `Respondida` desde `Pendiente Notificar`).
- `Enviar Recibo` y `Recibo Pagado` **no existen** para los procesos de
  cancelación — el filtro por estado debe reflejar esta dependencia.
- `En Gestión` agrupa deliberadamente dos situaciones (espera del Decano y
  espera del Estudiante, esta última solo tras "Enviar Recibo" en Examen
  Supletorio); no la desambigües en la tabla, solo en el detalle
  (campo `esperaDe`).
- Al rechazar en el paso `Pendiente Notificar`, la observación se
  precarga con un texto predeterminado editable ("La solicitud no cumple
  con los requisitos establecidos."), igual que en el rechazo inicial del
  Funcionario.
- El paso `Enviar Recibo` exige adjuntar el recibo de pago en **PDF**
  (campo obligatorio, bloquea el envío si falta el archivo — mismo criterio
  de la sección 9). El nombre del archivo se guarda en `r.recibo` y se
  muestra en el detalle de la solicitud junto al comprobante que luego
  suba el Estudiante. Este recibo **no es la Resolución** (sección 2): es
  un documento distinto, propio del cobro del examen supletorio, y sí está
  permitido adjuntarlo/descargarlo digitalmente.

## 4. Menú (exactamente tres ítems, no agregar más)
1. **Mi Usuario**
2. **Solicitudes**
3. **Respuestas**

## 5. Vista Solicitudes

- Tabla única con todas las solicitudes activas (todo lo que no está en
  `Respondida`). Al pasar a `Respondida` desaparece de aquí y aparece en
  Respuestas.
- Columnas: Número de radicado (`AAAA-XX-NNNN`, siglas `CM`/`CA`/`ES`),
  Nombre del Estudiante, Número de documento, Tipo de proceso, Fecha de
  radicación, Estado (chip de color), Acciones.
- Filtros: Tipo de proceso (solo los asignados al Funcionario + "Todos"),
  Estado (nunca ofrece `Respondida`; no ofrece `Enviar Recibo` ni
  `Recibo Pagado` si el Funcionario no tiene asignado Examen Supletorio),
  Número de documento (búsqueda). Botón de limpiar filtros. Estado vacío
  con mensaje informativo, nunca como error.
- Acciones por estado: `Pendiente` → Ver detalle · Remitir al Decano ·
  Rechazar. `En Gestión` → solo Ver detalle (ninguna acción de gestión
  habilitada). `Pendiente Notificar` → Ver detalle · Enviar respuesta.
  `Enviar Recibo` → Ver detalle · Enviar recibo al Estudiante.
  `Recibo Pagado` → Ver detalle · Aprobar comprobante · Rechazar comprobante.

### 5.1. "Ver detalle" — regla obligatoria transversal (Solicitudes y Respuestas)

El Funcionario debe ver **todos los campos que el Estudiante diligenció**,
cada uno como su propio dato estructurado (label + valor) — nunca
comprimidos en un bloque de texto libre. Incluye datos generales, la
justificación y su soporte, todos los campos propios del tipo de proceso
(ver la sección 6), todos los anexos identificados por nombre/tipo, y el
comprobante de pago cuando aplica.

Esto es una corrección de alcance conocida: el modal `abrirDetalle` de
`funcionario.html` hoy solo muestra `just` y una lista plana de `anexos`; no
lee los campos estructurados que el formulario de Examen Supletorio ya
genera (`examen`, `fechaExamen`, `propuesta`, `causa`). Si trabajas en esto,
la estructura `DATA` debe ampliarse para conservar esos campos y el modal
debe renderizarlos todos.

## 6. Vista Respuestas

- Historial de **solo lectura** de todas las solicitudes en `Respondida`
  (aprobadas o rechazadas). Sin botones de editar, reabrir, reasignar ni
  eliminar.
- Columnas: Nombre, Documento, Tipo de proceso, Decisión (Aprobada/Rechazada,
  diferenciadas visualmente), Acciones (solo Ver detalle; la tabla no tiene
  una acción de descarga propia).
- Filtros: Tipo de proceso, Decisión (Aprobada/Rechazada/Todos), Documento.
  No hay filtro por estado aquí (todo está en `Respondida`).
- "Descargar resolución" vive dentro de "Ver detalle" (tarjeta "Resolución"
  del detalle, sección 5.1), no como acción de la fila de la tabla. Aparece
  siempre que `r.resolucion` exista **y** Tipo de proceso ∈ {Cancelación de
  Matrícula, Cancelación de Asignatura} — tanto si la Decisión fue Aprobada
  como Rechazada (ver sección 2). Nunca para Examen Supletorio.
  - **Punto abierto sin resolver**: la especificación original (HU-07
    Escenario 2) pedía descargar **dos copias en carpetas separadas** (DARCA
    y DECANATURA); el detalle solo contempla una descarga. Falta unificar
    esto — no lo decidas por tu cuenta, pregúntalo.

## 7. Vista Mi Usuario

Solo consulta: nombre, documento, rol, correo institucional, procesos
asignados. Sin edición salvo que se especifique lo contrario.

## 8. Estilo visual

| Elemento | Valor |
|---|---|
| Barra lateral | `#1B2660` |
| Barra superior | Blanca |
| Encabezado de tabla | `#2A3A86`, texto blanco |
| Filas | Cebra alternada |
| Acento | `#B23B2E` |

Tipografía sans-serif institucional, sin sombras pronunciadas ni degradados.

## 9. Reglas transversales

- Toda tabla: paginación, columnas ordenables, estado vacío con mensaje
  informativo (distinto de un mensaje de error).
- Formularios con adjunto obligatorio: bloquean el envío si falta el
  archivo.
- Formatos: PDF para resoluciones (cuando en el futuro exista un escaneo,
  pendiente de decisión); PDF/JPG/PNG para comprobantes y anexos.

## 10. Explícitamente fuera de alcance (no implementar)

- Semáforo de antigüedad de solicitudes.
- Devolución al Estudiante para subsanación (el rechazo por documentación
  incompleta obliga a radicar una solicitud nueva, no a corregir la misma).
- Reasignación de solicitudes entre funcionarios.

Sí está en alcance: la línea de tiempo del trámite dentro del detalle.

## 11. Puntos abiertos (no inventar solución, dejar visible o marcador)

1. Formato definitivo de la convención de radicado (`AAAA-XX-NNNN`) — hoy es
   una constante fija por formulario, no un consecutivo real generado por
   el sistema.
2. Etiqueta visual de los chips de estado: ¿nombres canónicos de la sección
   3 o etiquetas cortas ("Por Revisar", etc.)? Mientras no se decida, usar
   los nombres canónicos.
3. Descarga de resolución en una o dos copias (ver sección 6).
4. Si además de la observación de rechazo hay otro documento del cierre que
   deba listarse en el detalle de Respuestas (p. ej. constancia de
   verificación de pago).
5. Si el campo Programa va como columna/filtro de la tabla o solo en el
   detalle (hoy solo está en el detalle).

## 12. Qué NO hacer aquí

- No generar, adjuntar ni ofrecer la Resolución en "Remitir al Decano": en
  ese punto del trámite el Decano todavía no ha decidido, así que el
  documento no puede existir (esto no cambió con la sección 2).
- No agregar ningún flujo de Resolución (adjuntar, generar, descargar) para
  Examen Supletorio, en ningún estado — solo aplica a Cancelación de
  Matrícula y Cancelación de Asignatura.
- No resolver por tu cuenta los puntos abiertos de la sección 11 — pregunta
  antes de decidir.
- No tocar archivos de las carpetas de Estudiante o Decano desde aquí.
- No agregar backend real ni reemplazar el bridge de `localStorage` sin que
  se pida explícitamente.