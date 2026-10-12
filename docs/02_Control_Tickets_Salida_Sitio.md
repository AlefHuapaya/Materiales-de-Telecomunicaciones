# Control de tickets de salida a sitio (Jira) por enlace

> Estado: **diseño v1 (2026-10-11)**. Pendiente: ejemplo de ticket Jira completo y lista de líneas de tarea para construir la plantilla y el prototipo.

## 1. Objetivo
Para cada enlace ejecutado (fibra óptica o microondas, planta externa o interna) se quiere saber:
1. **Qué enlaces no tienen ticket de salida a sitio** y de quién es la responsabilidad.
2. **Qué enlaces tienen el ticket completo** y cerrado para nuestro lado.
3. Si el material **instalado coincide con el asignado**, o si sobró (devolución) o faltó (complementaria).

Así se cierra el ciclo logístico del material de cada enlace y no se pierde stock. Hay enlaces del año pasado que siguen abiertos y deben ponerse al día.

## 2. Contratas válidas
Solo se consideran las contratas de `CODIGOS DE CONTRATAS.xlsx` (hoja *Nros de Acuerdo*). El archivo tiene 26 filas, pero son **19 contratas únicas**, porque hay duplicados con otra escritura, por ejemplo "FATCOM SAC" y "FATCOM S.A.C.", o nombres con espacios invisibles (`\xa0`). Hay que normalizar los nombres antes de cruzar.

| # | Contrata (nombre normalizado) |
|---|---|
| 1 | BELTRAN CHORIZOLA Y BALAGUER S.A.C. |
| 2 | F1 SERVICES & SOLUTIONS S.A.C. |
| 3 | SEMI PERU MONTAJES INDUSTRIALES S.A.C. |
| 4 | GRUPO REDYSTEL PERU S.A.C. |
| 5 | TELRAD PERU S.A. |
| 6 | GRUPO TRICONM S.A.C. |
| 7 | TRANSMITEL S.A.C. |
| 8 | OC&T OBRAS CIVILES Y TELECOMUNICACIONES SRL |
| 9 | COMUNICACION FUTURA S.A.C. |
| 10 | JPR TELECOMUNICACIONES S.A.C. |
| 11 | FATCOM S.A.C. |
| 12 | DOMINIONPERU SOLUCIONES Y SERVICIOS S.A.C. |
| 13 | SERVICIO DE TELECOMUNICACIONES OLITEL S.A.C. |
| 14 | MERIT INGENIERIA PERU S.A.C. |
| 15 | RD SOLUTIONS S.A.C. |
| 16 | STP PERU SAC |
| 17 | ICETEL PERU S.A.C. |
| 18 | TORRES AJ PERU S.A.C |
| 19 | I & M TELECOMUNICACIONES SAC |

El coordinador de cada contrata está en el Excel. Solo 9 de las 19 tienen uno asignado.

## 3. Flujo del ticket en Jira y responsables
`Crear → Abierto → Prevalidación Construcción → (Observado Const. ↺) → Generado → Aprobado → En Gestión → Salida Total / Salida Parcial / Sin Stock / Observado / Cerrado / Cerrado con Obs. / Cancelado`

| Estado | Responsable | Regla |
|---|---|---|
| Abierto (creación) | **Contratista** | Debe crearse a más tardar **7 días** después de la **fecha de activación** del enlace |
| **Prevalidación Construcción** | **Nosotros** | Es nuestro pendiente. Filtro: unidad funcional **Construcción Infraestructura** e **Implementación** |
| Observado Const. | **Contratista** | Debe subsanar |
| Prevalidación / Observado / Obs. Levantada OYM | Área OYM | Fuera de nuestro filtro |
| Generado → Aprobado → En Gestión | **Mira Group** | Solo seguimiento |
| Pendiente | **Mira Group** | No aplica a nuestro filtro |
| **Sin Stock / Salida Parcial / Observado** | **Nosotros** | Vuelve a nosotros para revisarlo de nuevo |
| Salida Total / Cerrado / Cerrado con Obs. | — | Cerrado para nuestro lado |
| Cancelado | — | Duplicado ya resuelto en un **ticket masivo**, a veces por otro revisor de la misma contrata (ver la regla 5.3) |
| Reapertura | — | **No se considera** (≈1 de cada 200–300 tickets) |

## 4. Modelo de datos
| Tabla | Campos | Uso |
|---|---|---|
| **T1. Maestro de enlaces** | ID / nombre del enlace, contrata, coordinador, correo o licitación de asignación, tipo (FO/MW), **fecha de activación**, estado de obra | La lista completa. Sale de la matriz de Excel Online de Entel (ver la sección 7) |
| **T2. Asignación por enlace** | enlace, SKU, proyecto, cantidad asignada | Lo que se les dio. Los SKU son fijos y recurrentes |
| **T3. Tickets Jira** | clave del ticket, contrata, revisor, fecha de creación, estado, unidad funcional | Lo que crearon |
| **T4. Puente ticket ↔ enlace** | ticket, enlace | Una fila por par. Resuelve el caso de **varios enlaces en un solo ticket** |
| **T5. Detalle del ticket** | ticket, enlace, **línea de tarea** (111/112/113…), SKU, lote, **código de proyecto**, descripción, cantidad | Lo declarado como instalado |
| **T6. Catálogo de líneas de tarea / stock** | línea de tarea, SKU, descripción | Para validar los SKU (pendiente de recibir) |

Para comparar se usa la clave **enlace + SKU + proyecto**, porque no todos los materiales de un enlace son del mismo proyecto.

## 5. Reglas
### 5.1 Conciliación del material
`Diferencia = Asignado − Declarado`
- Igual a 0: cuadra.
- Mayor que 0: **sobrante**, tiene que devolverse (logística inversa).
- Menor que 0: **faltante**, hay que verificar que exista una complementaria.
- Fibra (metrados): margen de tolerancia por definir, por ejemplo ±3 %.
- Si un SKU no está en el catálogo (se comprará después o hoy no hay stock), se marca **"SKU fuera de catálogo"** y se revisa aparte. No se cuenta como error.

### 5.2 Plazo de 7 días desde la activación
- 🟢 Ticket creado dentro del plazo.
- 🟡 Sin ticket, pero dentro del plazo.
- 🔴 Sin ticket y vencido. Se muestran los días de atraso. La responsabilidad es de la **contratista**.

### 5.3 Ticket cancelado
Un ticket cancelado no cubre al enlace por sí solo. Se busca otro ticket no cancelado de la misma contrata que incluya ese enlace (el masivo):
- Si existe, el enlace queda cubierto y el cancelado se marca como **duplicado**.
- Si no existe, el enlace pasa a 🔴 **sin ticket**.

### 5.4 Estado del enlace (semáforo)
El estado del ticket no es el estado del enlace. Un enlace se cierra solo cuando **todos** sus tickets están cerrados para nuestro lado **y** el material cuadra.

| Estado del enlace | Condición | Responsable |
|---|---|---|
| 🔴 Sin ticket (vencido) | Activado hace más de 7 días y sin ticket válido | Contratista |
| 🟡 En plazo | Activado hace 7 días o menos y sin ticket | Contratista |
| 🟠 Pendiente nuestro | Ticket en Prevalidación Construcción, Sin Stock, Salida Parcial u Observado | Nosotros |
| ⚪ Observado a la contrata | Ticket en Observado Const. | Contratista |
| 🔵 En Mira Group | Ticket en Generado, Aprobado, En Gestión o Pendiente | Mira Group |
| 🟣 Con diferencias | Ticket cerrado, pero con sobrante o faltante sin regularizar | Ambos |
| 🟢 Cerrado | Ticket cerrado y material conciliado | — |

## 6. Identificar los enlaces dentro de los tickets
Los nombres de enlace siguen el patrón `CODSITIO_LM_Nombre_CODSITIO_LM_Nombre`, con códigos de sitio de 7 dígitos (ejemplo: `0102867_LM_HB_Dante_AMT_0100068_LM_Caceres`). Para encontrar los enlaces en el título, la descripción o el correo del ticket:
1. Buscar con una expresión regular los códigos de 7 dígitos (`\b0\d{6}\b`, por ajustar con datos reales).
2. Formar el par de códigos A–B y cruzarlo con T1.
3. Lo que no se encuentre va a una lista de revisión manual.

## 7. Traer la lista de enlaces y coordinadores desde la matriz de Excel Online
**Opción A: copia fija.** En una hoja nueva de la matriz:
```
=UNICOS(FILTRAR(APILARH(TablaEnlaces[Nombre Enlace]; TablaEnlaces[Coordinador]; TablaEnlaces[Fecha Activación]); TablaEnlaces[Nombre Enlace]<>""))
```
Copia el resultado, pégalo **solo como valores** en un libro nuevo y borra la hoja. Si te da error de sintaxis, cambia `;` por `,`. Ajusta los nombres de la tabla y de las columnas.

**Opción B: libro que se actualiza solo (Power Query en Excel de escritorio).**
1. En la matriz online: *Archivo → Información → Copiar ruta de acceso*.
2. En el libro nuevo: *Datos → Obtener datos → Desde la Web*. Pega la ruta y borra `?web=1`.
3. Inicia sesión con tu cuenta profesional. Elige la tabla y entra a *Transformar datos*.
4. Deja solo las columnas de enlace, coordinador y fecha de activación (*Quitar otras columnas*, *Quitar filas en blanco y duplicados*).
5. *Cerrar y cargar*. Para traer los cambios: *Datos → Actualizar todo*.

**Google Colab: descartado.** Para leer el SharePoint de Entel, TI tendría que registrar una aplicación, y además los datos de la empresa saldrían a una cuenta de Google. El cruce se hace con un script local (prototipo) o con Power Query, uniendo las tablas (para la operación diaria).

## 8. Cómo poner al día los enlaces del año anterior
1. Armar T1 solo con los enlaces abiertos.
2. Exportar a CSV todos los tickets de salida a sitio con el filtro de unidad funcional.
3. Detectar los enlaces de cada ticket (sección 6) y llenar T4.
4. Calcular el semáforo y sacar la lista de 🔴 por contrata y coordinador, con un plazo para que creen el ticket.

## 9. Para que no vuelva a pasar
- Exigir que **cada línea del ticket indique su enlace**. Sin eso, en los tickets con varios enlaces solo se puede conciliar el grupo completo.
- Proponer un campo obligatorio "Enlace(s)" en Jira, o un formato fijo para el título.

## 10. Próximos pasos
- [ ] Recibir un **ejemplo de ticket Jira completo** (el correo que genera), anonimizado.
- [ ] Recibir la **lista de líneas de tarea** (111/112/113…) con el stock actual.
- [ ] Recibir el extracto de enlaces, coordinadores y fecha de activación (opción A).
- [ ] Construir la plantilla de la matriz de control (T1–T6, semáforo, conciliación).
- [ ] Script de prototipo: lee el export de Jira y T1, detecta los enlaces y genera el reporte.
- [ ] Definir la tolerancia de los metrados de fibra.
