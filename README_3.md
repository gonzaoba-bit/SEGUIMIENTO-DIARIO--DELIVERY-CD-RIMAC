# Seguimiento Centralizado · Delivery CD Rímac

Tablero de avance de rutas y seguimiento de MR (mercadería rechazada). Trae datos de ejemplo incrustados (un corte de `SIC MR2026 - Seguimiento EDTs.xlsm`) y permite actualizar el avance subiendo tus propios archivos directamente en el navegador.

Es una sola página HTML autocontenida (sin build, sin backend). Todo el parseo y los cálculos ocurren en el navegador — nada se envía a ningún servidor.

## Panel de KPIs (arriba del todo)

La franja de tiles debajo del título trae: Status avance, Clientes totales, Visitados, Pendientes, Avance Hl y **Clientes rechazados** (total de clientes con rechazo registrado en la Hoja de Ruta, en el corte/selección actual — la misma base que usa "MR contactos" en la sección de MR, solo que aquí en número absoluto en vez de porcentaje).

## Actualizar el avance con tus archivos

Arriba del tablero hay un panel **"Actualizar con tus datos"**, plegable — haz clic en el encabezado (o en la flecha) para expandirlo o colapsarlo. Se colapsa automáticamente después de procesar un archivo, para dejar más espacio a los datos; puedes volver a abrirlo cuando quieras cargar otro corte.

Adentro hay dos casilleros. Cada uno acepta el archivo de **dos formas**: haciendo clic en cualquier parte del recuadro (abre el explorador de archivos de tu sistema para seleccionarlo) o arrastrando el archivo directamente sobre el recuadro y soltándolo — el borde se resalta mientras arrastras encima para confirmar que quedó "listo para soltar".

- **Hoja de Ruta (Base)** — el `.xls` que exporta el sistema (en realidad es una tabla HTML con extensión .xls). Define el universo del día: rutas, vehículos, clientes, Hl programado, empresa. Es obligatorio para procesar.
- **routes-data (Avance)** — el CSV de ejecución (separado por `;`, export de BEES: `tour_display_id`, `poc_external_id`, `status`, `total_delivered_vol`, `total_refused_vol`, etc). Es opcional: sin él, todo el universo de la Hoja de Ruta se muestra como pendiente.

El cruce entre ambos archivos se hace por el código de cliente (`Codigo` ↔ `poc_external_id`). Al presionar **Procesar y actualizar** el tablero recalcula rutas, avance por transportista, MR y pendientes con esos datos, colapsa el panel de carga, y muestra la fecha/hora de carga arriba del todo (junto al título). **Volver al ejemplo** restaura los datos de muestra.

**Quién manda para saber si un cliente fue rechazado o entregado**: la tasa de rechazo (MR contactos y MR volumen, a nivel clientes y a nivel Hl) se calcula usando **solo la Hoja de Ruta** — su columna `rechazo` ("Si"/"No") decide, sin cruzar con el status ni el volumen rechazado de routes-data. Esto aplica igual con o sin routes-data cargado: subir el CSV no cambia estos dos indicadores. routes-data se sigue usando para todo lo demás que sí depende de él (avance de rutas, clientes visitados, Hl entregado).

Para Total vs. Parcial (columna "Tipo" en las tablas de rechazos), al no cruzar con routes-data la única señal disponible en la Hoja de Ruta es `ped_rech_parcial` — una columna de confiabilidad limitada (ver nota en el código), pero es lo único que hay sin salir de la Hoja de Ruta.

Importante: ambos archivos deben ser del mismo turno/fecha. Si el cruce de clientes entre los dos archivos es bajo, el tablero muestra una advertencia en rojo — es señal de que no corresponden al mismo corte.

La Hoja de Ruta solo trae el código de "Empresa", no el nombre comercial. El tablero ya trae incorporada la tabla de equivalencias:

| Código | Empresa |
|---|---|
| 497469 | SEGSOL |
| 472274 | CORPORACIÓN BREXIMAR S.A.C. |
| 471586 | JRGRODVAL DISTRIBUCIONES S.A.C. |
| 494742 | F&B ANDINA S.A.C. |
| 496518 | LOGISTICA INTELIGENTE SOLUTION S.A.C. |
| 445314 | VARGAS S.R.L. |

Un código que no esté en esta tabla se muestra como "Empresa &lt;código&gt;". Para agregar o corregir una equivalencia, edita el objeto `EMPRESA_NAMES` al inicio del `<script>` del archivo (o pídemelo y te lo actualizo).

## Filtro global por empresa

Debajo del panel de carga hay una barra **"Filtrar por empresa"** con un chip por transportista/empresa. Permite seleccionar **varias empresas a la vez** (se activan/desactivan al hacer clic) y afecta **todo el tablero**, no solo la tabla de rutas: hero/KPIs, gauges de avance y de MR, "Universo del corte", el gráfico combinado de rechazos, el detalle de rutas, "Rechazos registrados" y "Pedidos pendientes" se recalculan según la selección. "Todos" limpia la selección y vuelve a mostrar todas las empresas. Cada empresa conserva siempre el mismo color, esté o no seleccionada.

El buscador de texto sobre "Detalle de rutas" filtra por ruta, placa o transportista **dentro de** la selección de empresas activa.

## Filtro por viaje

Cuando la Hoja de Ruta que subes trae la columna **Viaje** (número de viaje/carga por vehículo dentro del día) y hay más de un valor distinto, aparece una segunda barra **"Filtrar por viaje"** debajo del filtro de empresa, con el mismo comportamiento: selección múltiple, "Todos" para limpiar, y afecta todo el tablero igual que el filtro de empresa (ambos se combinan). Con los datos de ejemplo esta barra no aparece, porque ese corte no conserva el detalle de viaje por cliente — se muestra automáticamente en cuanto subes un archivo real que lo incluya.

## Avance de rutas por empresa (arriba)

Justo debajo de los KPIs principales hay una tarjeta con el **% de rutas ya en estado Completada** sobre el total de rutas programadas, por transportista (barra + porcentaje + "X/Y rutas completadas"), para verlo de un vistazo sin bajar a la sección de detalle. Una ruta cuenta como completada según el mismo criterio de la tabla "Detalle de rutas" (ver más abajo).

## Modo oscuro / claro

Junto al nombre de la empresa, en la barra superior, hay un botón (🌙/☀️) para alternar entre modo oscuro y claro manualmente. Por defecto el tablero sigue la preferencia del sistema operativo/navegador; al usar el botón, esa elección queda fija mientras tengas la página abierta (no se guarda entre recargas — vuelve a seguir el sistema si recargas la página). En modo oscuro el logo se invierte automáticamente (y se ve un poco más grande) para que no se pierda sobre el fondo oscuro.

## Ordenar tablas

Todas las tablas del tablero (Detalle de rutas, Riesgo/Rechazos, Pedidos pendientes, MR por transportista, Modulación de pedidos y Checklist de vehículos) se pueden ordenar haciendo clic en el encabezado de cualquier columna: la primera vez ordena de menor a mayor (o alfabéticamente para texto), un segundo clic invierte el orden. El orden activo se mantiene aunque cambies el filtro de empresa/viaje o subas un archivo nuevo.

## Pestaña "Modulación de pedidos"

Junto a "Radar operativo" hay una segunda pestaña, **Modulación de pedidos**, que mide si los rechazos (totales o parciales) quedaron con un motivo de alerta registrado.

**A diferencia de MR contactos/volumen (que desde hace unos cambios usan solo la Hoja de Ruta), esta pestaña sí cruza ambos archivos**: cuando routes-data marca un cliente como `PARTIAL_DELIVERY`, quiere decir que ese código fue atendido de manera parcial, así que también entra al cálculo — igual que cualquier cliente con volumen rechazado confirmado por routes-data (`total_refused_vol > 0`) o marcado `rechazo = Si` en la Hoja de Ruta. El cruce se hace para todos los clientes, para poder sincerar de verdad cuáles no están alertados.

- **% modulado total de la operación** — tarjeta destacada arriba de todo: suma de clientes rechazados y **correctamente alertados** de TODAS las empresas juntas, sobre el total de rechazos del corte completo. No es el promedio de los % por empresa (eso sesgaría a empresas chicas); es un conteo agregado real.
- **% modulado por empresa** — lo mismo pero desglosado: cantidad de clientes rechazados y **correctamente alertados** sobre la cantidad de rechazos totales de esa empresa. "Correctamente alertado" exige las dos cosas: que la Hoja de Ruta tenga `alerta = Si` **y** que además traiga un motivo/comentario registrado (`mr`/`MRsub`/`comentarios_sub`) — un `alerta = Si` sin motivo documentado no cuenta como alertado.
- **Rutas top offender** — las rutas con más clientes rechazados sin alerta.
- **Detalle de clientes rechazados** — código, cliente, ruta, empresa, si fue rechazo Total o Parcial, y si quedó Alertado (Sí/No).

Esta pestaña necesita los datos por cliente de tu Hoja de Ruta (columnas `alerta`, `mr`/`MRsub`/`comentarios_sub`), así que con los datos de ejemplo muestra un aviso en vez de datos — se activa automáticamente al subir tus archivos.

Un cliente cuenta como "rechazado" para esta vista si **cualquiera** de los dos archivos lo indica: el routes-data por un volumen rechazado o por un estado `PARTIAL_DELIVERY` (entrega parcial), o la Hoja de Ruta por su propia columna `rechazo`. Ese cruce es intencional — ninguno de los dos archivos se toma como única fuente de verdad. Un cliente con `PARTIAL_DELIVERY` en routes-data siempre se clasifica como "Parcial" y se valida contra el motivo de alerta de la Hoja de Ruta, aunque esa columna no lo marque explícitamente. (Las columnas `ped_rech_total`/`ped_rech_parcial` de la Hoja de Ruta no se usan para decidir si un cliente está rechazado: en la práctica vienen repetidas por ruta completa, no por cliente, así que no sirven para identificar qué pedido puntual fue rechazado — solo `rechazo` sí es confiable a nivel de cada fila.)

## Pestaña "Checklist de vehículos"

Tercera pestaña, independiente del resto del tablero. Permite subir el export de checklist de VecFleet (`.xlsx`, columnas Estado, Unidad, TIPO DE CHECKLIST, Hora Inicio, Hora Fin, y las categorías de revisión) y mide:

- **Filtro Salida / Retorno** — chips arriba del resumen ("Todos", "Salida", "Retorno") que filtran el resumen, el cumplimiento por empresa, el tiempo y la tabla de detalle según el `TIPO DE CHECKLIST` del archivo. El farol de placas pendientes es la única excepción: siempre considera ambos tipos a la vez, porque su función es justamente comparar si a una placa le falta la salida, el retorno, o ambos.
- **Cumplimiento de salida y de retorno** — % de placas esperadas hoy (universo completo de tu Hoja de Ruta) que ya tienen un checklist registrado de ese tipo, sea cual sea su Estado (VALIDO o INVALIDO cuentan igual como "hecho"; lo que se mide aquí es que el checklist se haya hecho, no si pasó). El subtítulo de cada tarjeta muestra "hechas/esperadas placas" y, si corresponde, cuántas de esas fueron inválidas. Este cálculo siempre usa el checklist completo (ambos tipos) cruzado con la Hoja de Ruta, sin importar el filtro Salida/Retorno de arriba.
- **Cumplimiento por empresa** — cruzando la placa del checklist (columna `Unidad`) con el `Vehiculo` de tu Hoja de Ruta para saber a qué transportista pertenece cada checklist; aquí sí se mide calidad (% de checklists sin fallas, Estado VALIDO) sobre lo registrado por esa empresa — es un indicador distinto y complementario al cumplimiento de cobertura de arriba.
- **Farol de placas pendientes** — compara TODAS las placas esperadas hoy (de tu Hoja de Ruta) contra las que ya tienen algún checklist de salida y de retorno (válido o inválido, no importa): rojo = sin ningún checklist, ámbar = falta uno de los dos (salida o retorno), verde = tiene ambos. El objetivo es asegurar que ninguna placa se quede sin checklist, sin importar si salió aprobada o no.
- **Rutas con menor tiempo de checklist** — duración (fin − inicio) más corta, para detectar revisiones posiblemente superficiales; también muestra el tiempo promedio de todos los checklists.
- **Detalle de checklists** — tabla ordenable con placa, ruta, empresa, tipo, estado, fallas y duración de cada registro.

Como es independiente, funciona incluso si no subiste tu Hoja de Ruta (usa los datos de ejemplo para el cruce por placa); para que el cruce por empresa y el farol de pendientes reflejen tu operación real, sube primero tu Hoja de Ruta. El archivo se procesa enteramente en tu navegador (incluye un lector mínimo de `.xlsx` hecho a mano, sin depender de ninguna librería externa ni de internet).

## Seguimiento de MR (arriba del todo)

La sección de MR es ahora la primera del resumen operativo, y considera **únicamente el rechazo por cantidad de clientes** (contactos rechazados / contactos totales) — ya no muestra ningún indicador de volumen/Hl en esta sección. La tarjeta "Tasa de rechazo vs. meta" muestra un solo gauge:

- **MR contactos**: clientes rechazados / clientes programados — meta ≤8%.
- **Farol sobre meta**: cuando el indicador supera su meta, el gauge se pone rojo y aparece una etiqueta "● Sobre meta" encima (antes el gauge se quedaba siempre verde sin importar qué tan lejos estuviera de la meta). Mientras está dentro de la meta, se ve verde con "● Dentro de meta".

La tarjeta "Universo del corte" muestra solo **Contactos programados** y **Tasa de rechazo (contactos)**. El gráfico "Rechazos por transportista" muestra solo la barra de contactos rechazados (y su tasa) por transportista — ya no incluye la barra de Hl rechazados.

El Hl rechazado / Hl programado (MR volumen) se sigue calculando internamente para el resto del tablero (por ejemplo, para el cruce que usa "Modulación de pedidos"), pero ya no se pinta en el gauge ni en "Universo del corte"/"Rechazos por transportista".

### Cuadro "MR por transportista" (por empresa)

Debajo de "Rechazos por transportista" hay una tabla aparte, ordenable, con dos indicadores **a nivel empresa** (uno por fila, uno por transportista):

- **MR contactos**: clientes rechazados / clientes programados, de esa empresa.
- **MR volumen**: Hl rechazado / **Hl programado**, de esa empresa — misma fórmula que el MR volumen del resto del tablero, aplicada por transportista.

Cada porcentaje se muestra con una mini-barra y un color (verde si está dentro de la meta, rojo si la supera — mismo criterio y mismas metas que el gauge grande de arriba: ≤8% contactos, ≤2.5% volumen), para verlo de un vistazo sin tener que leer cada número. La tabla también muestra las cifras absolutas (clientes rechazados/programados, Hl rechazado/programado) junto a cada porcentaje.

**Nota sobre los datos de ejemplo**: se corrigió una inconsistencia en el dataset de ejemplo (el que se ve antes de subir tus archivos, o si subes archivos pero no llegas a apretar "Procesar y actualizar"). El gauge "MR contactos" mostraba 0.46% mientras que "Tasa de rechazo (contactos)", en la misma pantalla, mostraba 6.5% — ambos números venían del mismo archivo fuente pero de dos pivotes distintos que no coincidían entre sí (uno de ellos con un periodo más largo que el corte del día). Ahora ambos indicadores usan la misma base y muestran el mismo número en todos lados. Esto **no afecta tus archivos reales** — al subir tu Hoja de Ruta y routes-data y apretar "Procesar y actualizar", el tablero siempre calculó (y sigue calculando) MR contactos como una razón literal: total de clientes rechazados / total de clientes programados.

## Cuándo una ruta se marca "Completada" (pestaña "Detalle de rutas")

Una ruta pasa a **Completada** apenas todos sus clientes quedan cerrados según el `status` de routes-data, usando este criterio:

| Status (routes-data) | Cuenta como |
|---|---|
| CONCLUDED | Finalizado |
| DELIVERY_STARTED | Finalizado |
| PARTIAL_DELIVERY | Finalizado |
| DEFINITELY_RETURNED | Finalizado |
| WAITING_MODULATION | Finalizado |
| IN_TREATMENT | Finalizado |
| NOT_STARTED | Pendiente |
| RESCHEDULED | Pendiente |
| ON_THE_WAY | Pendiente |

Una ruta muestra "Completada" cuando el 100% de sus clientes están en un status "Finalizado"; "Sin iniciar" cuando ninguno lo está; "En ruta" en cualquier punto intermedio. Antes, `DEFINITELY_RETURNED`, `WAITING_MODULATION`, `DELIVERY_STARTED` e `IN_TREATMENT` no contaban como cierre del cliente, así que una ruta con esos status se quedaba mostrando "En ruta" aunque el transportista ya hubiera terminado de atenderla.

## Publicarlo en GitHub Pages

1. Sube este `index.html` a la raíz de un repositorio (o a una carpeta `/docs`).
2. En el repo: **Settings → Pages → Source**, elige la rama y carpeta donde quedó el archivo.
3. GitHub te da una URL tipo `https://<usuario>.github.io/<repo>/` en 1-2 minutos.

También funciona abriéndolo directo desde el explorador de archivos (doble clic) — la carga de archivos funciona igual, sin necesidad de subirlo a ningún lado.

## Orden de las secciones (pestaña "Radar operativo")

1. Seguimiento de MR — arriba, la tasa de rechazo vs. meta (gauge "MR contactos") y el "Universo del corte" (contactos programados y tasa de rechazo por contactos); debajo, un gráfico de barras con los contactos rechazados por transportista; y al final, la tabla ordenable "MR por transportista" con MR contactos y MR volumen (Hl rechazado/Hl programado) por empresa, con mini-barra y color según meta en cada celda.
2. Detalle de rutas (tabla filtrable).
3. Riesgo y estado de ejecución / Rechazos registrados — según la fuente de datos, incluye la columna **Tipo** (Total / Parcial) cuando hay rechazos: "Parcial" cuando el pedido tuvo entrega y rechazo a la vez (o viene marcado así en la Hoja de Ruta / `status = PARTIAL_DELIVERY` en routes-data), "Total" cuando fue enteramente rechazado. Con archivos cargados, también trae **Hl rechazado** y **Cajas rechazadas** por pedido (esta última tomada directamente de la columna `CjasRechazado` de la Hoja de Ruta), y un filtro **Todos / Total / Parcial** arriba de la tabla que mide cuántos rechazos hay de cada tipo sobre el universo completo del corte (no solo lo que se ve en pantalla — ver más abajo).
4. Pedidos pendientes de mayor volumen.
5. Progreso por transportista (avance de rutas, hectolitros y gauges generales) — al final de la página.

La pestaña **Modulación de pedidos** es independiente de este orden (ver arriba).

## Límites actuales

- Los datos que subes no quedan guardados: si recargas la página sin volver a subir los archivos, vuelve a mostrar el ejemplo.
- La sección de "Riesgo y estado de ejecución" cambia de contenido según la fuente: con los datos de ejemplo muestra rutas marcadas manualmente en el Excel (sin Hl/Cajas rechazadas ni filtro Total/Parcial, porque esos datos no existen en ese formato); con archivos subidos muestra los pedidos con rechazo registrado en la Hoja de Ruta, con Hl y cajas rechazadas por pedido, si fue total o parcial, y el filtro correspondiente. El filtro Todos/Total/Parcial mide sobre **todos** los rechazos del corte (así el filtro no esté activo), pero la tabla en sí solo pinta hasta 30 filas a la vez (las de mayor volumen rechazado, o las 30 de mayor volumen dentro del tipo filtrado) — el texto arriba de la tabla siempre indica el total real detrás del filtro, aunque la tabla muestre menos filas.
- Con los datos de ejemplo, los números de "Rechazos por transportista" (contactos) vienen de la propia ventana de MR del Excel original, que acumula un período más largo que el corte de rutas del día — por eso no van a cuadrar exactamente con el resto del tablero. Las tasas (%) sí están corregidas y son comparables. Con tus archivos cargados no aplica: todo se calcula del mismo corte.
- El logo y los colores están adaptados a la identidad de Delivery CD Rímac (negro/rojo). Para cambiar el logo, reemplaza `logo_final.png`/`logo_b64.txt` y regenera con `gen2.py`.
- La pestaña de Checklist de vehículos necesita que el navegador soporte `DecompressionStream` (lo tienen las versiones recientes de Chrome, Edge y Firefox) para leer el `.xlsx`; si tu navegador es muy antiguo, mostrará un mensaje de error en vez de procesar el archivo.
