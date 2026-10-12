# Contexto para Claude

Proyecto de logística de telecomunicaciones en Lima (Perú): flujo de materiales en Oracle, contratistas, fibra por lotes y salidas a sitio con Mira Group. Toda la documentación está en español.

- `README.md`: resumen priorizado del flujo de materiales.
- `docs/01_Analisis_Flujo_Materiales.md`: análisis completo del flujo.
- `docs/02_Control_Tickets_Salida_Sitio.md`: control de tickets Jira de salida a sitio por enlace. Es el trabajo activo. Revisa la sección 10 (próximos pasos).

## Reglas
- Repo **privado**. No subas archivos reales de la empresa: el `.gitignore` excluye xlsx, csv, kmz y la carpeta `datos/`. Los archivos reales van en `datos/` (local).
- Los ejemplos que se suban deben estar anonimizados.
- Solo se consideran las 19 contratas listadas en el doc 02.
- No uses Google Colab ni servicios externos con datos de Entel. Usa scripts locales o Power Query.
