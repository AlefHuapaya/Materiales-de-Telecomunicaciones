# Propuesta: cambios seguros en Oracle y desabastecimiento de materiales

> Versión 1, 2026-10-05. Complementa [01_Analisis_Flujo_Materiales.md](01_Analisis_Flujo_Materiales.md).
> Responde a dos preocupaciones nuevas:
> 1. Oracle es la misma plataforma para toda la empresa. Cambiar su configuración puede afectar a otras áreas y a los materiales que todavía están en proceso sin regularizar.
> 2. Falta material a nivel de la empresa: se instala todo lo liberado y quedan ejecuciones pendientes sin material.

---

## 1. Resumen

- **Sí se puede mejorar sin cambios drásticos.** La mayor parte de las mejoras se puede hacer **fuera de Oracle**, con reportes de solo lectura y reglas de proceso. Así no se toca nada de lo que ya está en curso. Es la **Vía A**.
- **Los cambios en Oracle (Vía B) se hacen solo si hacen falta**, de forma gradual y con tres reglas:
  1. Lo que está en proceso **no se migra**. Termina con las reglas de siempre.
  2. Antes de cambiar algo se **regularizan los pendientes**.
  3. Cada cambio se prueba primero en un **ambiente de prueba** y luego en un **piloto acotado**.
- **El desabastecimiento (P10) es un problema de visibilidad y anticipación.** Hoy se nota cuando ya no hay material. La propuesta es medir la **cobertura**, es decir, cuántas ejecuciones pendientes alcanza a cubrir el saldo. Cuando la cobertura baja de un umbral se pide la liberación adicional a ingeniería **antes** de quedarse sin material.

---

## 2. Nuevo punto crítico

### P10 🔴 Desabastecimiento: se instaló todo lo liberado y quedan ejecuciones sin material

**Situación:** ingeniería libera material cada cierto tiempo según los proyectos estimados. Cuando se instala todo lo liberado y todavía quedan ejecuciones, no hay material para continuar.

**Causas probables** (hay que confirmarlas con datos):

| Causa | Cómo se nota | Dato para comprobarla |
|---|---|---|
| La estimación de ingeniería se queda corta (cambios de ruta, reservas, catenaria) | El consumo real del RPT supera lo liberado por enlace | RPT vs. liberado, por proyecto |
| Material del proyecto usado en otro proyecto o en urgencias | El saldo baja sin que avancen los enlaces del proyecto | Traspasos y transferencias A↔B |
| Material inmovilizado en contratistas (sobrantes, retazos, sin devolver) | Hay stock "teórico" pero no en almacén | Conciliación P6 y antigüedad P9 |
| Se pide más liberación cuando ya no hay nada | El pedido llega tarde por el tiempo de aprobación y de compra | Fechas de pedido, de liberación y de llegada |
| Aumento del alcance (más ejecuciones de las estimadas) | El número de enlaces crece frente al plan | Lista de enlaces planificados vs. ejecutados |

**Propuesta (sin cambios en Oracle):**

1. **Saldo por ejecutar, por proyecto y material**

   `Saldo libre = Liberado por ingeniería − Entregado − Comprometido en solicitudes en curso`

   `Necesidad pendiente = Σ metraje o cantidad de los enlaces que faltan ejecutar`

   `Brecha = Necesidad pendiente − Saldo libre`. Si es positiva, ya se sabe que faltará material.

2. **Cobertura en semanas**

   `Cobertura = Saldo libre ÷ Consumo semanal promedio` (consumo según el RPT de las últimas 4 a 8 semanas)

   - Si la cobertura es menor que el **tiempo de reposición** (lo que tarda ingeniería en liberar más, más la compra si hace falta) + 1 semana de margen → **pedir la liberación adicional ya**.
   - Semáforo: 🟢 cobertura holgada · 🟡 cerca del umbral · 🔴 por debajo del umbral o brecha positiva.
   - *Ejemplo ilustrativo:* saldo de 18 km de fibra 48H, consumo de 6 km por semana → 3 semanas de cobertura. Si ingeniería tarda 4 semanas en liberar, el pedido ya va tarde.

3. **Solicitud formal de liberación adicional a ingeniería**, con un formato estándar:
   - proyecto, material, cantidad, brecha calculada
   - motivo: cambio de ruta, consumo mayor al estimado, alcance adicional o reposición de material prestado
   - enlaces que quedan detenidos si no llega el material

   Así la solicitud llega **justificada con datos**, no como un pedido urgente.

4. **Antes de pedir más, recuperar lo que ya existe** (en este orden):
   1. stock de la propia empresa en otros proyectos de la misma gerencia (P8)
   2. sobrantes en contratistas según la conciliación (P6)
   3. devoluciones adelantadas de stock con más de 90 días sin uso (P9)
   4. retazos útiles para trabajos cortos (P2-ter)

5. **Si de todas formas falta, asignar lo poco que hay con un criterio.** Primero los enlaces que **quedan completos** (P2-bis), luego por la matriz de prioridad (P4). Así no se reparte material que deja todo a medias.

6. **Retroalimentación a ingeniería:** cada mes se reporta la **desviación de consumo** por tipo de enlace y zona, `(consumo real − liberado) ÷ liberado`. Con eso ingeniería ajusta el factor de estimación de la siguiente liberación. Es el mismo dato que calibra el factor del KMZ (sección 2-B del documento 01).

**Indicadores nuevos:**
- Ejecuciones detenidas por falta de material (cantidad y días detenidos)
- Cobertura en semanas por material crítico
- Desviación entre consumo real y liberado, por proyecto

**Archivo de control nuevo:**

| Archivo | Contenido clave | Frecuencia | Fuente |
|---|---|---|---|
| **O. Liberaciones y cobertura** | Proyecto, material, liberado, entregado, comprometido, saldo libre, necesidad pendiente, brecha, consumo semanal, cobertura, fecha de la última solicitud de liberación | Semanal | Ingeniería + B + C + E + J |

### P11 🔴 Accesos por proyecto y por tarea: stock que existe pero el coordinador no ve

**Situación:** no todos los coordinadores tienen acceso a todos los proyectos ni a todas las **tareas** de un proyecto (1-1, 1-1-1, 1-1-2…). A veces un coordinador solo ve una. En el stock real revisado, todo el cable nuevo de un tipo estaba en una sola tarea: un coordinador con acceso a otra tarea del **mismo proyecto** veía cero.

**Propuesta:**
1. **Tres niveles de saldo en el tablero:**
   1. lo que **puedo pedir** con mis accesos
   2. lo que hay **en mis proyectos, en tareas sin acceso**: se coordina con quien tiene acceso
   3. lo que hay **en otros proyectos**: requiere traspaso (P8)
2. **Matriz de accesos** (archivo P): usuario × proyecto × tarea. Sirve para saber a quién pedir el material y para sustentar un pedido de acceso.
3. **En Oracle:** acceso de **consulta** a todas las tareas de los proyectos de la gerencia. Es un cambio 🟢 sin riesgo, porque ver no es lo mismo que poder pedir.
4. **Antes de pedir una liberación nueva (P10), revisar los niveles 2 y 3.**

### Reglas confirmadas del stock
- **"Disponible" no es "utilizable":** el exporte marca como disponible incluso el material de proyectos de baja. El tablero los excluye y muestra el material usado aparte.
- **Subinventarios de solicitud (D0001, D0003):** no se combinan en una misma solicitud, así que el saldo se muestra por separado.
- **SKUs equivalentes:** cables de distintos fabricantes con los mismos hilos y vano sí se pueden usar indistintamente. Se suman por grupo genérico (archivo L).
- **Lote ≠ bobina:** el lote corresponde al ingreso al almacén y no existe el dato de cada bobina. Propuesta sin tocar Oracle: un **registro de bobinas** en el almacén (lote, número de bobina, metraje de etiqueta, metraje actual). Mientras no exista, el simulador usa la línea del exporte como unidad.

| Archivo nuevo | Contenido clave | Frecuencia | Fuente |
|---|---|---|---|
| **P. Matriz de accesos** | Usuario, proyecto, tarea (o "todas"), tipo de acceso (consulta / solicitud) | Mensual o por cambio | TI / coordinadores |
| **Q. Registro de bobinas** | Lote, número de bobina, metraje de etiqueta, metraje actual, ubicación | Por ingreso y despacho | Almacén |

---

## 3. Vía A: mejorar sin modificar Oracle

Todo lo de esta vía **solo lee** información de Oracle (exportes y reportes) o cambia la forma de trabajar. No altera la configuración ni los datos, así que **no afecta a otras áreas ni a lo que está en proceso**.

| Problema | Solución sin tocar Oracle | Qué se usa de Oracle |
|---|---|---|
| P1 Cuentas compartidas | Bitácora obligatoria de uso de cuentas. Cada solicitud indica quién la preparó en el campo de comentarios o justificación | Nada nuevo |
| P2 Una línea cancela todo | **Solicitudes más pequeñas:** una por enlace o por grupo de materiales. Las líneas de stock dudoso van en una solicitud aparte. Si falta algo, solo se cae esa parte | Nada nuevo |
| P2-bis Entrega por enlace | Se arma por fuera la asignación por tramo (simulador). Al despachar, logística elige los lotes en la pantalla estándar de asignación de la orden de movimiento | Selección de lotes estándar, si el ítem ya tiene control de lotes |
| P3 Visibilidad de stock | Tablero diario: exporte de stock + solicitudes en curso. Semáforo de saldo libre | Exportes / reportes estándar |
| P4 Prioridad | Matriz de puntaje en el tablero | Nada |
| P5 Citas | Cita confirmada y registro de cumplimiento | Nada |
| P6 Stock por contratista | Conciliación en Excel/Power BI: entregado − RPT − devuelto ± transferencias | Exporte de entregas |
| P7 Transferencias | Formato con ID y aprobación | Nada |
| P8 Traspasos | Matriz de dueños de proyecto y SLA interno | Nada nuevo |
| P9 Antigüedad | Reporte de aging en el tablero | Exporte de entregas |
| **P10 Desabastecimiento** | **Cobertura, brecha y solicitud de liberación anticipada** (sección 2) | Exportes + liberaciones de ingeniería |
| 2-B Solicitud masiva | Fase 1: el motor genera la plantilla y un usuario la copia a Oracle | Carga manual, igual que hoy |

**Límite de esta vía:** el control vive en archivos externos y depende de la disciplina del equipo. Sirve para ordenar el proceso y **obtener los datos que justifiquen** después un cambio en Oracle, si hace falta.

---

## 4. Vía B: cambios en Oracle sin romper lo que está en proceso

### 4.1 Por qué existe el riesgo
- Oracle es **una sola instancia** para toda la empresa. Un cambio a nivel global (de sitio, del maestro de ítems o de un tipo de orden compartido) afecta a todas las áreas que lo usan.
- Las transacciones **en proceso** (requisiciones abiertas, órdenes de movimiento pendientes, entregas sin RPT, transacciones con error en la interfaz) se crearon con las reglas anteriores. Si las reglas cambian a mitad de camino, pueden quedar **atascadas, duplicadas o descuadradas** en el costo o en el stock.
- Algunos cambios **no se pueden hacer** con stock existente. Por ejemplo, Oracle normalmente no permite cambiar el control de lotes o de series de un ítem que ya tiene stock o transacciones abiertas. *Confirmar con TI.*

### 4.2 Las 6 reglas para cambiar sin romper

1. **Regularizar antes de cambiar.** Se saca un inventario de pendientes y se cierra o corrige cada uno:
   - requisiciones y órdenes de movimiento abiertas o detenidas
   - transacciones con error en las interfaces
   - entregas a contratistas que no tienen su salida a sitio en el RPT
   - diferencias del último conteo

   En EBS, la ventana de **periodos contables de inventario** muestra las transacciones pendientes al intentar cerrar un periodo. Es una buena lista de control. *Confirmar con TI cuál usan.*
2. **Fecha de corte y convivencia.** Lo creado **antes** de la fecha de corte termina con el flujo actual hasta cerrarse. Lo creado **después** usa la regla nueva. Nada se migra a medio camino.
3. **Cambios acotados, no globales.** Siempre que se pueda, se configura en el nivel más bajo: un **usuario o responsabilidad** en lugar de todo el sitio, un **tipo de orden o subinventario nuevo** en lugar de modificar el existente, o **un proyecto piloto** en lugar de toda la gerencia.
4. **Probar primero en un clon.** TI prueba el cambio en una copia del ambiente productivo que **incluye las transacciones en curso**. Así se ve qué les pasa antes de tocar producción.
5. **Piloto y luego expansión.** Un proyecto y una contratista durante 2 a 4 semanas. Se mide, se corrige y recién después se amplía.
6. **Plan de vuelta atrás.** Cada cambio debe poder desactivarse: quitar la responsabilidad, volver a usar el tipo de orden anterior, etc. Y se hace con la gestión de cambios formal de TI, con aprobación y responsable.

### 4.3 Clasificación de los cambios propuestos por riesgo

| Cambio (doc 01) | A quién afecta | Riesgo para lo que está en proceso | Cómo hacerlo de forma segura |
|---|---|---|---|
| **Acceso de consulta** a proyectos de la gerencia (P8) | Solo a quien recibe el acceso | 🟢 Ninguno: solo lectura | Responsabilidad o rol de solo consulta |
| **Rol Preparador** (P1) | Solo a los 7 coordinadores | 🟢 Ninguno: es acceso nuevo, no cambia solicitudes existentes | Responsabilidad nueva. Los 5 aprobadores siguen igual |
| **Reportes nuevos** (stock libre, cobertura, aging) | Nadie: solo leen | 🟢 Ninguno | Reporte o consulta a medida, sin escribir datos |
| **Carga masiva** con la interfaz estándar (2-B, fase 2) | Solo las solicitudes nuevas cargadas | 🟡 Bajo: solo crea registros nuevos; el riesgo es cargar datos malos | Probar en el clon, cargar por lotes pequeños, validar antes de importar |
| **Alerta de material ya solicitado** (P3) | Usuarios de la responsabilidad personalizada | 🟡 Bajo: no cambia datos, solo avisa | Personalización limitada a una responsabilidad o sandbox, no global |
| **Reservas al aprobar** (P3) | Ítems y proyectos donde se active | 🟡 Medio: una reserva bloquea stock que otros veían libre | Empezar con reservas **manuales** (función estándar) en el piloto, sin automatizar |
| **Entrega parcial + backorder** (P2) | Todos los que usan ese tipo de orden | 🟠 Medio-alto si se modifica el tipo de orden compartido | Crear un **tipo de orden o flujo nuevo** solo para la gerencia. Las órdenes viejas siguen con el anterior |
| **Subinventario por contratista** (P6) | Logística y contabilidad | 🟠 Medio-alto: cambia cómo se registra la entrega y el consumo, con impacto contable | Solo para entregas nuevas desde la fecha de corte. El material ya entregado se concilia y se carga como saldo inicial con un conteo firmado. Requiere a Finanzas |
| **Control de lotes / lote divisible** (P2-bis) | Todos los que usan el ítem en la empresa | 🔴 Alto: es un cambio del maestro de ítems y puede no estar permitido con stock existente | **No cambiar el ítem.** Usar la selección de lotes estándar y llevar la asignación por tramo fuera de Oracle. Si hiciera falta, crear un ítem nuevo, no modificar el actual |
| **Borrow/Payback** (P8) | Toda la configuración de proyectos | 🔴 Alto si el módulo no está activo | No activarlo por ahora. Usar la matriz de dueños + SLA (Vía A) |

### 4.4 Qué les pasa a los materiales que quedan en proceso

| Situación en la fecha de corte | Qué se hace |
|---|---|
| Solicitud aprobada pero no despachada | Termina con el flujo actual. No se modifica |
| Solicitud cancelada por falta de una línea | Se vuelve a crear con la regla nueva (solicitudes pequeñas o backorder) |
| Material entregado a la contratista sin RPT | Se **regulariza primero**: cierre técnico o devolución. Si se crea el subinventario por contratista, entra como saldo inicial conciliado, no como transacción retroactiva |
| Transferencia A↔B sin registro | Se documenta con el formato nuevo y se ajusta en la conciliación antes del corte |
| Transacción con error en la interfaz | La corrige TI antes del corte. No se activa ningún cambio mientras existan errores abiertos del mismo ítem o proyecto |

---

## 5. Secuencia recomendada

| Paso | Plazo | Qué | ¿Toca Oracle? |
|---|---|---|---|
| 1 | Semanas 0–4 | Vía A completa: tablero, conciliación, **cobertura y brecha (P10)**, solicitudes pequeñas, bitácora | No |
| 2 | Semanas 2–6 | Inventario de pendientes y regularización. Medir la línea base de los indicadores | Solo lectura |
| 3 | Mes 2 | Cambios 🟢: acceso de consulta, rol Preparador, reportes | Sí, sin riesgo para lo que está en proceso |
| 4 | Mes 2–3 | Cambios 🟡 en el clon de pruebas y luego en el piloto: carga masiva, alerta, reservas manuales | Sí, acotado |
| 5 | Mes 4–6 | Cambios 🟠 solo si los datos lo justifican: tipo de orden nuevo con backorder, subinventario por contratista. Con fecha de corte y Finanzas | Sí, con piloto |
| — | — | Cambios 🔴: no se recomiendan. Se mantienen las alternativas | No |

**Mensaje para la gerencia y TI:** no se pide cambiar Oracle de inmediato. Primero se ordena el proceso con datos, se regulariza lo pendiente, y solo después se piden cambios acotados, probados y reversibles, empezando por los que no tienen riesgo.

---

## 6. Datos pendientes de confirmar

- Cada cuánto libera ingeniería material y cuánto tarda una liberación adicional. ¿Y una compra?
- Si la liberación de ingeniería queda registrada en Oracle (por proyecto) o en un archivo aparte.
- Lista de enlaces pendientes por proyecto, con su metraje estimado.
- Si TI cuenta con un ambiente de prueba clonado de producción y cada cuánto se refresca.
- El procedimiento de gestión de cambios de TI (quién aprueba un cambio de configuración).
- Si hay transacciones con error en las interfaces o periodos de inventario sin cerrar.
