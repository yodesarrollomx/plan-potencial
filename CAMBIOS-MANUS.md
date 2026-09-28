# Registro de cambios realizados por Manus

Este archivo complementa `CLAUDE.md`. Claude y cualquier mantenedor deben leer ambos antes de modificar mensajes comerciales, medición o integraciones.

## 2026-09-28 · Capital: de promesa porcentual a calificación por proyecto

### Objetivo de negocio

Mejorar la calidad del interés de inversionistas y reducir riesgo reputacional o legal. Una cifra anual aislada atrae conversaciones basadas en una promesa que no explica proyecto, plazo, costos ni riesgos. El nuevo mensaje conserva la invitación comercial, pero deja claro que la oportunidad depende de la calificación y de la información específica de cada proyecto.

### Cambio publicado

**Antes:**

> Asociación en participación, retornos de hasta 20% anual.

**Ahora:**

> Participar en proyectos que califiquen, con información y riesgos definidos por proyecto.

### Capas actualizadas

1. `index.html`, catálogo `OTRAS`, respaldo local de `otra_capital_d`.
2. Google Sheet `CRM - YOD`, pestaña `TEXTOS POTENCIAL`, celda `B90`, que es la fuente efectiva del copy público.
3. `CLAUDE.md`, alerta activa y regla para futuras ediciones.

### Capas no modificadas

- Apps Script y su despliegue.
- Endpoint público.
- Estructura del Sheet.
- Folios, leads, CRM, Meta Pixel y eventos.
- Lógica del formulario y agenda.

### Validación requerida

1. El GET público `?recurso=textos` debe devolver el nuevo valor en `textos.otra_capital_d`.
2. La página pública debe mostrar el nuevo texto al entrar por “No tengo terreno, me interesa de otra forma” y elegir “Aportar capital”.
3. El repositorio no debe contener la frase anterior fuera de este registro histórico.

### Reversión

La reversión requiere cambiar las dos fuentes: `index.html` y `TEXTOS POTENCIAL!B90`. No debe revertirse solo una, porque el Sheet sobrescribe el respaldo HTML cuando responde.

### Regla permanente

No publicar porcentajes de rendimiento, seguridad, plusvalía o retorno como promesa general. Si Dirección y Legal aprueban una cifra para un proyecto concreto, debe incluir fecha, plazo, supuestos, costos, riesgos, elegibilidad y lenguaje de no garantía.
