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

## 2. DECISIÓN CLAVE — la Resolución es 100% física, no se toca en código

Por reglamento de la Universidad del Cauca **no se admiten firmas digitales**.
Esto ya está implementado correctamente en el código actual (no lo
retrocedas) y debe seguir así en cualquier cambio futuro:

- **Paso "Remitir al Decano"**: solo hay un botón "Remitir al Decano", **sin
  ningún documento adjunto ni generado**. La Resolución no existe todavía en
  este punto del trámite (ver `funcionario.html` ~línea 1218: el propio
  diálogo aclara que "la Resolución no se genera en este paso").
- **Notificación al Estudiante tras la decisión del Decano**: el modal que
  informa la decisión al Estudiante muestra únicamente un mensaje de texto
  — "debe acercarse a la Decanatura a firmar la Resolución", dentro de los
  cinco (5) días siguientes — **sin ningún archivo adjunto** (ver
  ~línea 1246-1252). No agregues botones de "Adjuntar PDF", "Generar
  Resolución" ni "Subir escaneo" en este flujo.
- **Historial de Respuestas**: cuando se muestra el detalle de una solicitud
  ya aprobada, el texto indica que la Resolución fue "firmada físicamente
  por el Decano", que el original se entregó al Estudiante y las copias
  quedaron archivadas en Decanatura y en DARCA — de nuevo, sin archivo
  digital (ver ~línea 1080-1087).
- **Pendiente de decisión (próxima reunión):** aún no se sabe si en algún
  momento se subirá un **escaneo** de la Resolución ya firmada físicamente,
  como archivo digital de consulta/archivo. Mientras no se confirme, no
  implementes nada relacionado con esto; si un requerimiento nuevo lo
  pidiera, trátalo como punto abierto y consúltalo antes de construir algo.
- Examen Supletorio **nunca genera Resolución**, apruebe o rechace el
  Decano — eso ya está correctamente señalado en el código
  (`dialog-note`: "Este proceso no genera Resolución; la decisión se
  comunica directamente al Estudiante").

## 3. Modelo de estados (relativos al Funcionario)

El sistema guarda dos campos internos — `Responsable Actual` y `Decisión` —
de los que se deriva la etiqueta visible.

| Estado | Significado | Acción esperada del Funcionario |
|---|---|---|
| Pendiente | La solicitud acaba de llegar del Estudiante | Revisar y remitir al Decano, o rechazar |
| En Gestión | No es su turno | Ninguna; solo seguimiento |
| Pendiente de Respuesta | El Decano ya decidió y devolvió el trámite | Enviar la respuesta al Estudiante |
| Pendiente de Verificación | El Estudiante subió el comprobante de pago | Verificar el comprobante y cerrar |
| Respondida | Trámite cerrado | Ninguna; solo consulta |

- Cancelación de Matrícula y Cancelación de Asignatura: `Pendiente`,
  `En Gestión`, `Pendiente de Respuesta`, `Respondida`.
- Examen Supletorio: los cuatro anteriores + `Pendiente de Verificación`.
- `Pendiente de Verificación` **no existe** para los procesos de
  cancelación — el filtro por estado debe reflejar esta dependencia.
- `En Gestión` agrupa deliberadamente dos situaciones (espera del Decano y
  espera del Estudiante); no la desambigües en la tabla, solo en el detalle
  (campo `esperaDe`).

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
  Estado (nunca ofrece `Respondida`; no ofrece `Pendiente de Verificación`
  si el Funcionario no tiene asignado Examen Supletorio), Número de
  documento (búsqueda). Botón de limpiar filtros. Estado vacío con mensaje
  informativo, nunca como error.
- Acciones por estado: `Pendiente` → Ver detalle · Remitir al Decano ·
  Rechazar. `En Gestión` → solo Ver detalle (ninguna acción de gestión
  habilitada). `Pendiente de Respuesta` → Ver detalle · Enviar respuesta.
  `Pendiente de Verificación` → Ver detalle · Verificar comprobante.

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
  diferenciadas visualmente), Acciones (Ver detalle · Descargar resolución
  condicional).
- Filtros: Tipo de proceso, Decisión (Aprobada/Rechazada/Todos), Documento.
  No hay filtro por estado aquí (todo está en `Respondida`).
- "Descargar resolución" solo aparece si Decisión = Aprobada **y** Tipo de
  proceso ∈ {Cancelación de Matrícula, Cancelación de Asignatura}. Nunca
  para Examen Supletorio ni para rechazadas.
  - **Punto abierto sin resolver**: la especificación original (HU-07
    Escenario 2) pedía descargar **dos copias en carpetas separadas** (DARCA
    y DECANATURA); la sección de Respuestas solo contempla una descarga.
    Falta unificar esto — no lo decidas por tu cuenta, pregúntalo.

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
5. **Si se subirá o no, más adelante, un escaneo de la Resolución firmada**
   — pendiente para la próxima reunión (ver sección 2).
6. Si el campo Programa va como columna/filtro de la tabla o solo en el
   detalle (hoy solo está en el detalle).

## 12. Qué NO hacer aquí

- No generar, adjuntar, descargar ni simular firma digital de la Resolución
  en "Remitir al Decano" ni en la notificación al Estudiante.
- No resolver por tu cuenta los puntos abiertos de la sección 11 — pregunta
  antes de decidir.
- No tocar archivos de las carpetas de Estudiante o Decano desde aquí.
- No agregar backend real ni reemplazar el bridge de `localStorage` sin que
  se pida explícitamente.