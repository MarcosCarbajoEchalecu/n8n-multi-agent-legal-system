# Arquitectura técnica — ALUP

## Decisiones de diseño

### ¿Por qué multi-agente en lugar de un agente único?

El derecho administrativo municipal cubre materias muy distintas con legislación específica para cada una. Un agente único con todo el corpus legal genera respuestas menos precisas y más lentas. La arquitectura multi-agente permite que cada agente tenga su propio contexto legal reducido y especializado, mejorando tanto la precisión como el rendimiento.

### ¿Por qué Redis para la memoria contextual?

Los expedientes sancionadores requieren varios turnos de consulta — el inspector puede hacer preguntas de seguimiento sobre el mismo caso. Redis permite mantener el contexto entre turnos de forma eficiente y con TTL configurable, evitando que la memoria se acumule indefinidamente.

### ¿Por qué Airtable para la gestión documental?

Airtable actúa como catálogo de documentos legales activos, permitiendo añadir, actualizar o retirar normativa sin tocar el workflow. El equipo jurídico puede gestionar el corpus legal desde una interfaz no técnica.

---

## Sistema de puntuación

Cada agente puntúa la relevancia de cada documento del 1 al 100. Solo se procesan documentos con puntuación ≥ 50, y se seleccionan los TOP 3 por consulta. Esto optimiza el uso de tokens y mejora la coherencia de la respuesta.

## Flujo de memoria temporal

```
Consulta recibida
      │
      ▼
¿Existe sesión activa en Redis?
      ├── Sí → Recuperar contexto + borrar memoria temporal
      └── No → Iniciar nueva sesión
      │
      ▼
Procesar consulta con contexto
      │
      ▼
Guardar documento en memoria temporal
      │
      ▼
Respuesta al usuario
```

## Gestión de PDFs

Los expedientes en PDF se procesan vía Google Drive:
1. El inspector sube el PDF a una carpeta de Drive
2. n8n detecta el archivo nuevo
3. Se extrae el texto con Extract from PDF
4. El texto se procesa igual que una consulta de texto
