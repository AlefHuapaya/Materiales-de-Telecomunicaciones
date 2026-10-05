# Análisis del flujo de materiales: logística de telecomunicaciones

> Versión 1, 2026-10-04. Documento de trabajo, se ajusta cuando llegue el reporte RPT.

---

## 1. Flujo actual

| # | Etapa | Quién | Herramienta |
|---|-------|-------|-------------|
| 1 | Solicitud de materiales (cable, antenas, RRU, BBU, SFP) por metrado, unidad o serie | Coordinadores (12), solo 5 con cuenta | Oracle |
| 2 | Validación de stock en la matriz | Sistema | Oracle |
| 3 | Lista de espera: se evalúa viabilidad y urgencia | Reunión de logística | Manual |
| 4 | Programación del día de entrega | Logística | Manual |
| 5 | Recojo o entrega a la contratista | Contratista | Presencial |
| 6 | Ejecución en sitio, cierre y "salida a sitio" (descargo del stock de la contratista) | Contratista y Mira Group | RPT (diario) |
| 7 | Transferencias entre contratistas (casos urgentes) | Contratista A → B | Coordinación |
| 8 | Logística inversa: devolución del material sin uso a los 6 meses | Contratista → empresa | Manual |
| ⟂ | Traspaso de material entre proyectos (ID de proyecto) | Dueño del proyecto origen | Solicitud formal (~2 semanas) |

---

## 2. Puntos de dolor y propuesta de mejora

Prioridad: 🔴 alta · 🟡 media · 🟢 baja

### P1 🔴 Cuentas compartidas (5 cuentas para 12 usuarios)
- **Problema:** usar la cuenta de otra persona rompe la **trazabilidad**: en una auditoría, todo lo que se hizo queda a nombre del titular de la cuenta. Es el principal riesgo legal y de control interno del flujo, aunque se haga con cuidado.
- **Propuesta:** Oracle permite **preparar una solicitud a nombre de otro**. En EBS iProcurement funciona con el campo *Requester* y el perfil `ICX: Override Requestor`. En Fusion existe el rol *Procurement Preparer*.
  - Los 7 coordinadores sin cuenta reciben un rol **Preparador**, que prepara pero no aprueba.
  - Los 5 usuarios actuales se quedan como **Solicitante/Aprobador**.
  - Así se respeta la decisión de Administración (solo 5 aprueban) y cada persona entra con su propia cuenta.
- **Acción:** pedido formal a TI/Administración con este argumento de control interno. Hasta que se resuelva, llevar una **bitácora de uso de cuentas** (quién, cuándo y qué solicitud).

### P2 🔴 Si falta una línea, se cancela la solicitud completa
- **Problema:** si una de 10 líneas no tiene stock, se cae toda la solicitud.
- **Causa probable en Oracle:** las líneas están agrupadas en un **Ship Set con enforce** (en Order Management, para una requisición interna), o el flujo está configurado como "todo o nada". Oracle sí maneja **backorder automático**: atiende lo que hay y abre una línea pendiente por el faltante.
- **Propuesta:**
  1. Que TI revise la configuración (*ship sets*, tolerancias) y habilite **entrega parcial + backorder**.
  2. Como regla interna, separar en otra solicitud las líneas críticas o de stock dudoso.
- **Indicador:** % de solicitudes canceladas por falta de stock (objetivo: menos del 5 %).

### P2-bis 🔴 Entrega parcial de fibra: por enlace completo, no por metros sueltos
- **Problema:** en planta externa, entregar 2 km de un enlace que necesita 10 km no sirve de nada. En cambio, 2 km sí sirven si alcanzan para cerrar **otro enlace completo** de 2 km. Una entrega parcial "por cantidad" no le dice a nadie qué enlace se puede ejecutar.
- **Cómo funciona el stock:** un mismo **SKU** (por ejemplo, fibra de 48 hilos) tiene varios **lotes**, y cada lote equivale a uno o más rollos con su propio metraje. Se pueden solicitar varios lotes del mismo SKU y **sumarlos** hasta llegar a lo que pide el enlace.
- **Regla propuesta:**
  1. La solicitud se arma **por enlace y por tramo**. Un tramo es el recorrido entre dos puntos de empalme, por ejemplo entre mangas o cajas.
  2. A cada tramo se le suma la **reserva técnica** (gasas, subidas a poste, cocas), que se define como un % o en metros fijos.
  3. **Se entrega solo si el enlace queda completo.** Si un enlace no se puede cubrir entero, no se le despacha nada y su metraje queda libre para otro enlace.
  4. La solicitud debe mostrar **cuánto se extrae de cada lote**. Por ejemplo: *SKU 48H → Lote A 4,000 m + Lote B 3,500 m + Lote C 2,600 m = 10,100 m, para 10,000 m requeridos (reserva incluida)*.
- **⚠ Ojo con sumar lotes:** dos rollos solo se pueden sumar en un mismo tramo si **en el punto donde se unen hay un empalme previsto en el diseño**. Si el tramo no admite un empalme intermedio, necesita **un solo rollo** que lo cubra completo. Por eso la asignación se hace **por tramo** y no solo por el total del enlace.
- **En Oracle:**
  - El inventario con **control de lotes** ya registra el stock de cada lote.
  - Al asignar la orden de movimiento, la ventana *Select Available Inventory* permite **elegir qué lotes** se despachan.
  - El atributo **lote divisible**: si está en "No", hay que despachar el lote completo y no se puede cortar el rollo. Hay que revisar cómo está configurado para la fibra.
  - A TI se le puede pedir un reporte o pantalla de "asignación sugerida por tramo" que haga este cálculo automáticamente.
- **Lógica de asignación** (la que usa el simulador): primero los tramos más largos. A cada tramo se le asigna el **rollo más corto que lo cubra**, para no gastar rollos grandes en tramos cortos y dejar menos sobrantes inútiles.
- **Archivo nuevo, J. Diseño de enlaces:** enlace, tramo, punto inicial y final, metros de diseño, % de reserva, SKU y prioridad.

### P2-ter 🟡 Lote de respaldo y retazos de fibra
- **Situación actual:** la reserva técnica se maneja pidiendo **un lote adicional de respaldo**. Al terminar quedan retazos de 10, 20 o 50 m que no sirven para un enlace grande, porque significarían demasiados empalmes por fusión.
- **Propuesta:**
  1. **Longitud mínima útil:** definir con ingeniería un umbral. Todo pedazo por debajo de él es **retazo** y deja de contar como stock disponible para enlaces troncales. Así el tablero no muestra metros que en la práctica no sirven.
  2. **Los retazos se registran aparte** (subinventario o etiqueta "retazo"). Se usan en trabajos cortos, si los hay, o se devuelven o dan de baja. No se quedan en el almacén de la contratista inflando su saldo.
  3. **El respaldo pasa a ser un fondo por contratista:** un solo lote de contingencia por contratista o zona, en vez de uno por enlace, con **devolución obligatoria** de lo que no se use. El tamaño del fondo se calcula con el exceso real del historial.
  4. **Al asignar, se elige el rollo más corto que cubra el tramo** (como hace el simulador), para dejar menos sobrantes de longitud inútil.

---

## 2-B. Solicitud masiva automatizada por enlace

**Conclusión: es viable para el cable. Para el resto de materiales conviene un esquema semiautomático.**

### Las 3 fuentes que ya existen
| Fuente | Qué aporta | Qué le falta |
|---|---|---|
| **Asignación del proyecto** (*data field*) | Nombre del enlace, trabajo asignado, materiales genéricos | SKU, cantidades exactas |
| **KMZ** | Ruta de diseño propuesta (geometría) | Es solo una propuesta, puede cambiar |
| **ID de ruta** | Ruta que realmente toma el tramo, validada por el coordinador | Control de versiones (v1, v2…) |

### Lo que falta construir
1. **Catálogo genérico → SKU:** una tabla que traduzca "Fibra 48H aérea" al SKU de Oracle, y lo mismo para mufas, ODF y demás. Sin esta tabla no hay automatización posible, porque la asignación no trae SKU.
2. **Cálculo del metraje desde el KMZ:** el KMZ es un KML comprimido. La longitud de cada tramo se calcula desde sus coordenadas. Esa longitud es la distancia **en planta**. Para tendido **aéreo** hay que aplicar un **factor** que cubre catenaria, subidas a poste y reservas en mangas.
3. **Calibrar el factor con datos reales:** comparar lo que mide el KMZ con lo que se consumió según el **RPT**, en los enlaces ya cerrados. Así se obtiene el factor real por zona o por contratista, en lugar de un porcentaje fijo "a ojo".
4. **Lista de enlaces con prioridad:** el equipo ordena los enlaces y el sistema genera la solicitud completa de cable, enlace por enlace y en ese orden, asignando lotes por tramo como en P2-bis.

### Cable y demás materiales
| Tipo | Cómo se solicita | Por qué |
|---|---|---|
| **Cable de fibra** | **Automático**: metraje del KMZ × factor, asignado a lotes por tramo | Sale directo de la ruta |
| **Mufas, ODF, cajas** | **Semiautomático**: cantidad base según el diseño (mufas por punto de empalme, ODF por extremo) | Se pueden contar en el KMZ y la asignación, pero varían en campo |
| **SFP, tarjetas, controladoras, antenas, RRU/BBU** | **Kit base por tipo de enlace + solicitud complementaria** | Hay tarjetas que se agregan después del trabajo y no se pueden predecir |

- **Solicitud complementaria:** un formato rápido con justificación ("tarjeta adicional post-trabajo, enlace X") ligado al mismo enlace. Medir el **% de adicionales por tipo de enlace** sirve para ir corrigiendo el kit base.

### Control de cambios de ruta
- La ruta cambia por **zonas arqueológicas**, **propietarios** que no aceptan el tendido aéreo frente a su predio y **observaciones municipales**.
- Cada cambio genera una **nueva versión del ID de ruta** con su motivo: arqueológico, propietario, municipal u otro.
- El sistema **recalcula solo la diferencia** de metraje:
  - Si la nueva ruta es **más larga** y la diferencia no entra en el respaldo, se genera una solicitud complementaria.
  - Si es **más corta**, se registra la devolución.
- **Prevención:** en Perú, el **CIRA** (Certificado de Inexistencia de Restos Arqueológicos, del Ministerio de Cultura) se exige a proyectos de telecomunicaciones. Marcar en el KMZ los tramos que pasan cerca de zonas sensibles y gestionar el CIRA **antes** de pedir el cable evita pedir metraje para una ruta que luego se cae.
- **Indicador:** % de enlaces con cambio de ruta y metros de diferencia por motivo. Sirve para ajustar el respaldo y anticipar zonas problemáticas.

### Cómo llega a Oracle
- **Fase 1 (sin TI):** el motor genera una **plantilla de solicitud** por enlace (SKU, cantidad, lote sugerido, proyecto). Un usuario con cuenta la copia a Oracle. Reduce errores y tiempo aunque la carga siga siendo manual.
- **Fase 2 (con TI):** carga masiva. En Oracle EBS existe el programa estándar **Requisition Import**, que toma las líneas de la tabla de interfaz `PO_REQUISITIONS_INTERFACE_ALL` y crea las solicitudes. En Fusion se pide a TI el mecanismo equivalente de importación. **Falta confirmar** si su flujo usa requisiciones u órdenes de movimiento; eso define la interfaz.

### P3 🔴 Sin visibilidad del stock real ni de lo que ya se está pidiendo
- **Problema:** el coordinador no sabe cuánto hay disponible ni si otro coordinador ya pidió el mismo material, y eso genera tickets con error.
- **Propuesta en Oracle:** usar **Disponible para transar / reservar**, que se calcula como *On-hand − reservas − transacciones pendientes*. Si la solicitud **reserva** el stock al aprobarse, el siguiente coordinador ya ve la cantidad descontada. Para la alerta "este material ya está siendo solicitado" hace falta una personalización: *Forms Personalization* en EBS o una regla de validación en Fusion. Se pide a TI.
- **Propuesta inmediata, sin tocar Oracle:** un **tablero compartido** (Excel/Power BI con actualización diaria) que muestre:
  - stock Oracle por material y proyecto
  - cantidad comprometida en solicitudes en curso
  - **saldo libre real** = stock − comprometido
  - semáforo: verde si hay saldo, amarillo si ya hay un pedido en curso, rojo si no hay saldo.

### P4 🟡 Lista de espera y reuniones manuales
- **Problema:** las prioridades se deciden en reunión, sin un criterio escrito.
- **Propuesta:** una **matriz de priorización** con puntaje objetivo:
  - Urgencia (caída de servicio / regulatorio / planificado): 3 / 2 / 1
  - Fecha comprometida con el cliente o la licitación: menos de 7 días = 3
  - Stock disponible completo: sí = 2
  - Así la reunión solo revisa las excepciones, no la lista completa.

### P5 🔴 La contratista no llega el día de entrega (+2 o 3 días)
- **Propuesta:**
  - **Cita confirmada** 24 h antes, con ventana horaria y nombre de la persona que recoge.
  - Si no llega, el cupo se **libera** y la solicitud vuelve a la cola. El material no queda bloqueado.
  - **Registro de cumplimiento por contratista** (% de citas cumplidas) que se presenta en el comité.
  - Revisar si el contrato tiene un **SLA o penalidad** por incumplimiento de citas. Si no lo tiene, proponerlo para el próximo contrato o adenda.

### P6 🔴 No se sabe cuánto material tiene hoy cada contratista
- **Problema:** el RPT depende de que la contratista cierre el trabajo, y lo hace tarde porque lo junta con los tickets de pago.
- **Propuesta:**
  - **Fórmula de conciliación por contratista y material:**
    `Stock teórico = Entregado − Consumido (RPT) − Devuelto ± Transferencias A↔B`
  - **Separar el cierre técnico del cierre de pago:** el consumo se reporta como máximo X días hábiles después de ejecutar (por ejemplo 5), aunque el ticket de pago llegue después.
  - **Corte mensual de inventario** firmado por la contratista. En equipos seriados (RRU, BBU, SFP) se verifica **por número de serie**.
  - Oracle: evaluar un **subinventario por contratista** (stock en consignación), de modo que cada entrega sea una transferencia y cada salida a sitio sea un consumo. El saldo queda en el sistema y no en un Excel.

### P7 🟡 Transferencias entre contratistas
- **Propuesta:** toda transferencia A → B debe tener un **formato con ID**, aprobación de logística y registro (en Oracle como transferencia entre subinventarios, si se aplica lo de P6). Sin ese registro, la conciliación de P6 no cuadra.

### P8 🔴 Traspaso de material entre proyectos (~2 semanas)
- **Propuesta:**
  1. **Acceso de consulta** a todos los proyectos de la **misma gerencia**. Ver el stock no es lo mismo que poder moverlo.
  2. Si existe Oracle Project Manufacturing, usar **Préstamo/Devolución (Borrow/Payback)**: se presta material de otro proyecto y luego se devuelve al mismo costo. Es más rápido que un traspaso definitivo.
  3. **Matriz de dueños de proyecto:** ID de proyecto, gerencia, responsable, aprobador y suplente. Así se sabe con quién hablar.
  4. **SLA interno** de 48 a 72 h para los traspasos dentro de la gerencia, con un formato estándar.
  5. Los traspasos con otras gerencias se mantienen por el flujo formal.

### P9 🟢 Logística inversa cada 6 meses
- **Propuesta:** un **reporte de antigüedad (aging)** del stock en contratistas: 0–90, 90–150 y más de 150 días. Se avisa al día 150 para decidir antes de llegar a 180: asignar a otro plan, transferir o devolver.

---

## 3. Archivos de control necesarios

| Archivo | Contenido clave | Frecuencia | Fuente |
|---|---|---|---|
| **A. Maestro de materiales** | Código, descripción, unidad (m/und), ¿seriado?, familia | Fijo | Oracle |
| **B. Stock Oracle** | Material × proyecto × almacén: on-hand y disponible | Diario | Export de Oracle |
| **C. Solicitudes en curso** | N.º de solicitud, material, cantidad, proyecto, solicitante, estado, prioridad | Diario | Oracle y registro propio |
| **D. Programación de entregas** | Solicitud, contratista, fecha de la cita, ¿asistió?, reprogramación | Diario | Logística |
| **E. RPT (salidas a sitio)** | Sitio, contratista, material, cantidad o serie, fecha de ejecución y fecha de cierre | Diario | RPT / Mira Group |
| **F. Stock por contratista** | Conciliación (fórmula P6) + aging | Semanal | Calculado con B, D, E, G |
| **G. Transferencias A↔B** | ID, origen, destino, material, cantidad o serie, aprobador | Por evento | Logística |
| **H. Matriz de proyectos** | ID de proyecto, gerencia, dueño, aprobador, ¿tenemos acceso? | Mensual | Gerencia |
| **I. Bitácora de cuentas** (temporal) | Usuario real, cuenta usada, fecha, solicitud | Por evento | Equipo |
| **J. Diseño de enlaces** | Enlace, tramo, punto inicial y final, metros de diseño, % de reserva, SKU, prioridad | Por proyecto | Ingeniería / diseño |
| **K. Stock por lote** | SKU, lote, metraje disponible, almacén, ¿divisible?, ¿retazo? | Diario | Export de Oracle (lotes) |
| **L. Catálogo genérico → SKU** | Nombre genérico (como aparece en la asignación), SKU, unidad, tipo de solicitud (auto/semi/kit) | Fijo, se actualiza | Logística + Oracle |
| **M. Versiones de ruta** | ID de ruta, versión, motivo del cambio, metros antes/después, quién validó | Por evento | Coordinadores |
| **N. Kits base por tipo de enlace** | Tipo de enlace, material, cantidad base, % histórico de adicionales | Trimestral | Logística + ingeniería |

**El tablero central** junta B, C y F en una sola vista: *¿hay material?, ¿quién lo pidió?, ¿dónde está?*

---

## 4. Indicadores

1. % de solicitudes canceladas por falta de stock
2. Días promedio entre la solicitud y la entrega
3. % de citas cumplidas por contratista
4. Días promedio entre la ejecución y el cierre en el RPT, por contratista
5. Diferencia entre la conciliación teórica y el conteo físico
6. Días promedio de un traspaso entre proyectos
7. Valor del stock con más de 150 días en contratistas

---

## 5. Hoja de ruta

| Fase | Plazo | Qué | ¿Requiere TI? |
|---|---|---|---|
| **Victorias rápidas** | 0–4 semanas | Tablero B+C+F, matriz de priorización, citas confirmadas, matriz de dueños de proyecto, bitácora de cuentas, formato de transferencias | No |
| **Configuración de Oracle** | 1–3 meses | Rol Preparador, entrega parcial/backorder, reservas al aprobar, acceso de consulta a proyectos de la gerencia | Sí |
| **Estructural** | 3–6 meses | Subinventario por contratista, alerta de material ya solicitado, Borrow/Payback, SLA en contratos | Sí, más Legal y Compras |

---

## 6. Datos pendientes de confirmar

- ¿Es Oracle **E-Business Suite** (EBS) o **Fusion Cloud**? Cambia el nombre de las pantallas y los roles.
- ¿Quién administra Oracle (TI interno o un proveedor)?
- Estructura del **RPT**: columnas y si incluye números de serie.
- ¿Cuántas contratistas hay y cuántos almacenes tiene cada una?
- ¿Existe hoy una cláusula de SLA o penalidad en el contrato con las contratistas?

---

## Fuentes (documentación oficial de Oracle)
- Préstamo/Devolución entre proyectos: [Oracle Project Manufacturing – Inventory Transfers](https://docs.oracle.com/cd/E18727_01/doc.121/e13680/T634873T635599.htm), [Implementation Guide](https://docs.oracle.com/cd/E18727-01/doc.121/e13685/T635081T635120.htm)
- Solicitud a nombre de otro: [iProcurement Implementation Guide](https://docs.oracle.com/cd/E18727_01/doc.121/e13409/T207713T209128.htm), [Procurement Preparer (Fusion)](https://docs.oracle.com/en/cloud/saas/procurement/26c/oapcm/Procurement_Preparer_job_roles.html)
- Backorder y ship sets: [Internal Requisitions – Order Management](https://docs.oracle.com/cd/E26401_01/doc.122/e48931/T446883T443952.htm), [Shipping Execution](https://docs.oracle.com/cd/E18727_01/doc.121/e13431/T414830T414838.htm)
- Lotes y asignación de órdenes de movimiento: [Oracle Inventory User's Guide – Move Orders](https://docs.oracle.com/cd/E18727_01/doc.121/e13450/T291651T428143.htm), [Lot Management (Fusion)](https://docs.oracle.com/en/cloud/saas/supply-chain-and-manufacturing/26b/famml/lot-management.html)
- Carga masiva de solicitudes (EBS): [Requisition Open Interface](https://oracleappsdetails.blogspot.com/2009/10/requisition-open-interface.html), [PO_REQUISITIONS_INTERFACE_ALL](http://oracleapps88.blogspot.com/2011/11/porequisitionsinterfaceall.html) (fuentes secundarias; confirmar con TI)
- CIRA (Perú): [¿Qué es el CIRA?](https://arqueoconsultora.com/articulo/que-es-el-cira), [Trámite virtual CIRA/PMA](https://www.turiweb.pe/certificado-de-inexistencia-de-restos-arqueologicos-se-podra-obtener-de-forma-virtual/) (fuentes secundarias; validar en gob.pe/cultura)
- Disponible para transar/reservar: [Oracle Fusion – Available to Reserve/Transact](https://docs.oracle.com/en/cloud/saas/supply-chain-and-manufacturing/25d/famml/available-to-reserve-and-available-to-transact-values.html)
