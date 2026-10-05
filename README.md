# Flujo de materiales: logística de telecomunicaciones

Propuesta de mejora del flujo de solicitud, entrega, consumo y devolución de materiales (cable de fibra, antenas, RRU, BBU, SFP, ODF, mufas) gestionados en Oracle y ejecutados por contratistas.

> Resumen comprimido. El detalle completo está en [docs/01_Analisis_Flujo_Materiales.md](docs/01_Analisis_Flujo_Materiales.md).
> Presentación de avance (12 diapositivas): [docs/02_Presentacion_Avance_v1.pptx](docs/02_Presentacion_Avance_v1.pptx).
> Estado: **análisis y diseño (v1, 2026-10-05)**. Pendiente: recibir ejemplos de la asignación, el KMZ y el RPT para construir el prototipo.

---

## Flujo actual (resumen)
`Solicitud en Oracle → validación de stock → lista de espera / reunión → programación → entrega a la contratista → ejecución y salida a sitio (RPT, Mira Group) → transferencias A↔B → logística inversa (6 meses)`
Proceso aparte: **traspaso de material entre proyectos** (~2 semanas).

---

## Puntos de mejora por prioridad

### 🔴 Alta
| # | Problema | Propuesta clave |
|---|---|---|
| P1 | 5 cuentas Oracle para 12 usuarios: se comparten cuentas y se pierde la trazabilidad (riesgo de auditoría) | Rol **Preparador**: cada coordinador prepara con su usuario y los 5 actuales aprueban. Bitácora de uso de cuentas mientras tanto |
| P2 | Una línea sin stock cancela la solicitud completa | Revisar los *ship sets* y la regla de todo o nada. Activar **entrega parcial + backorder** |
| P2-bis | Entregar fibra parcial no sirve si no completa un enlace | **Entrega por enlace completo, tramo por tramo**. Se suman lotes del mismo SKU, pero solo se unen rollos donde hay un empalme previsto. Se asigna el rollo más corto que cubra el tramo |
| P3 | No se ve el stock real ni lo que ya está pedido | Reservar el stock al aprobar. **Tablero diario**: stock − comprometido = saldo libre, con semáforo. Alerta de material ya solicitado (personalización de TI) |
| P5 | La contratista llega 2 o 3 días tarde | Cita confirmada 24 h antes. Si no llega, se libera el cupo. Ranking de cumplimiento y SLA en el contrato |
| P6 | No se sabe cuánto material tiene cada contratista | Conciliación: `entregado − consumido (RPT) − devuelto ± transferencias`. Cierre técnico en 5 días, separado del pago. Subinventario por contratista |
| P8 | Traspaso entre proyectos: ~2 semanas | Acceso de consulta a los proyectos de la gerencia. **Borrow/Payback** de Oracle. Matriz de dueños de proyecto. SLA de 48 a 72 h |

### 🟡 Media
| # | Problema | Propuesta clave |
|---|---|---|
| P2-ter | El lote de respaldo deja retazos de 10 a 50 m que no sirven | **Longitud mínima útil**: lo que está por debajo es retazo y no cuenta como stock. Un solo fondo de respaldo por contratista, con devolución obligatoria |
| P4 | La prioridad se decide en reunión, sin criterio escrito | Matriz de puntaje (urgencia, fecha comprometida, stock completo). La reunión solo ve excepciones |
| P7 | Transferencias A→B sin registro | Formato con ID y aprobación de logística |
| 2-B | Solicitudes manuales, línea por línea | **Solicitud masiva por enlace**: automática para el cable (KMZ × factor calibrado con el RPT); kit base + complementaria para el resto de materiales. Control de versiones del ID de ruta y CIRA anticipado |

### 🟢 Baja
| # | Problema | Propuesta clave |
|---|---|---|
| P9 | Logística inversa a los 6 meses, manual | Reporte de antigüedad (aging) con alerta al día 150 |

---

## Motor de solicitud masiva (diseño)
```
Asignación (enlace + genéricos) ─┐
KMZ (ruta de diseño)  ───────────┼─► Motor de cálculo ◄── Catálogo genérico→SKU
ID de ruta (versión validada) ───┘         ▲          ◄── Stock por lote (Oracle)
                                           │
                ┌──────────────────────────┴───────────┐
          Cable: automático                  Otros: kit base + adicionales
          (tramos + lotes completos)
                └──────────────► Archivo de carga Oracle ──► Coordinador valida
↻ Si cambia el ID de ruta, se recalcula solo la diferencia
```
- **Fase 1 (sin TI):** el motor genera la plantilla y un usuario la copia a Oracle.
- **Fase 2 (con TI):** carga masiva con *Requisition Import* en EBS, o el equivalente en Fusion.

---

## Archivos de control
| | Archivo | Frecuencia |
|---|---|---|
| A | Maestro de materiales | Fijo |
| B | Stock Oracle (disponible) | Diario |
| C | Solicitudes en curso | Diario |
| D | Programación de entregas / citas | Diario |
| E | RPT: salidas a sitio | Diario |
| F | Stock por contratista (conciliación + aging) | Semanal |
| G | Transferencias A↔B | Por evento |
| H | Matriz de proyectos y dueños | Mensual |
| I | Bitácora de cuentas (temporal) | Por evento |
| J | Diseño de enlaces por tramo | Por proyecto |
| K | Stock por lote (incluye si es retazo) | Diario |
| L | Catálogo genérico → SKU | Fijo |
| M | Versiones de ruta | Por evento |
| N | Kits base por tipo de enlace | Trimestral |

---

## Hoja de ruta
| Fase | Plazo | Qué | ¿Requiere TI? |
|---|---|---|---|
| Victorias rápidas | 0–4 sem | Tablero B+C+F, matriz de prioridad, citas, matriz de proyectos, bitácora, catálogo genérico→SKU | No |
| Configuración de Oracle | 1–3 meses | Rol Preparador, backorder, reservas, acceso de consulta, control de lotes y lote divisible | Sí |
| Estructural | 3–6 meses | Subinventario por contratista, motor de solicitud masiva, carga por interfaz, Borrow/Payback, SLA en contratos | Sí, más Legal y Compras |

## Herramientas
- [herramientas/simulador_lotes_fibra.html](herramientas/simulador_lotes_fibra.html): asigna lotes de un SKU a los tramos de cada enlace y despacha solo los enlaces que quedan completos. Se abre en el navegador.

## Pendiente de confirmar
- ¿Oracle EBS o Fusion? ¿Se pide como requisición o como orden de movimiento?
- Ejemplos de la asignación, un KMZ y el RPT (anonimizados).
- Longitud mínima útil de fibra y si el lote es divisible.
- Número de contratistas y si el contrato tiene SLA o penalidades.
