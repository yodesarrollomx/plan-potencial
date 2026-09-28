# Registro de cambios realizados por Manus

Este archivo complementa `CLAUDE.md`. Claude y cualquier mantenedor deben leer ambos antes de modificar mensajes comerciales, medición o integraciones.

## 2026-09-28 · Paso de contacto: menos fricción y datos más válidos

### Objetivo de negocio

Reducir abandono en el punto de mayor intención. El gate mostraba Nombre, WhatsApp y Correo al mismo tiempo,
aunque el correo secundario no era obligatorio. La nueva versión pide solo Nombre + WhatsApp por defecto,
mantiene la alternativa de correo y permite añadir un correo secundario de forma opcional.

### Cambios publicados

1. Labels visibles para Nombre, WhatsApp/Correo y Correo opcional.
2. `autocomplete` adecuado para nombre, teléfono y correo.
3. Correo secundario colapsado detrás de “Agregar correo opcional”.
4. Cambio dinámico del label, ayuda, teclado y autocomplete al elegir correo como canal principal.
5. Validación de WhatsApp entre 10 y 15 dígitos.
6. Validación del correo opcional únicamente cuando la persona lo escribe.
7. Errores con `role="alert"`, `aria-live`, `aria-invalid` y foco en el campo que debe corregirse.
8. Placeholder activo `ph_email` simplificado a `tu@correo.com` en `TEXTOS POTENCIAL`.

### Contrato preservado

El backend sigue recibiendo `nombre`, `tel`, `email` y `contacto_canal`. No se modificaron Apps Script,
endpoint, folio, cola de reintentos, Meta Pixel ni eventos. WhatsApp sigue siendo el canal por defecto.

### Validación requerida

1. La vista inicial del paso 5 muestra dos campos: Nombre y WhatsApp.
2. “Agregar correo opcional” abre y cierra el tercer campo sin cambiar el canal principal.
3. “Prefiero por correo” convierte el segundo campo en correo y oculta el correo secundario.
4. Un teléfono con menos de 10 o más de 15 dígitos no avanza.
5. Un correo opcional inválido no avanza; vacío sí.
6. Un teléfono válido conserva `contacto_canal="whatsapp"`; el modo correo conserva `contacto_canal="correo"`.
7. El JavaScript inline pasa validación sintáctica y la página funciona en móvil y escritorio.

### Reversión

Revertir el commit correspondiente y restaurar `TEXTOS POTENCIAL!B101`. No modificar el backend: el contrato de
payload no cambió.

### Métrica para decidir si funcionó

Comparar por periodos equivalentes la tasa `paso 5 → lead`, el porcentaje de contactos válidos y el tiempo a
primer contacto. No concluir con muestras mínimas ni atribuir causalidad si cambió la pauta al mismo tiempo.

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
