# Registro de cambios realizados por Manus

Este archivo complementa `CLAUDE.md`. Claude y cualquier mantenedor deben leer ambos antes de modificar mensajes comerciales, medición o integraciones.

## 2026-09-28 · Mejora 4: adquisición social por referidos medibles

### Hallazgo que origina el cambio

La auditoría pública encontró audiencias pequeñas (Instagram: 184 seguidores; Facebook: 129), interacción
visible baja en las publicaciones recientes y enlaces históricos repartidos entre `alexpueblag.github.io`,
`yodesarrollo.github.io` y `yodesarrollomx.github.io`. Además, el enlace de Facebook del cierre apuntaba al ID
obsoleto `103075448823440`, mientras la página pública activa usa `100075840402647`.

### Objetivo de negocio

Crear una fuente adicional de adquisición orgánica desde personas que ya completaron el formulario y conocen a
otro propietario de terreno, conservando una atribución uniforme hasta el nuevo lead.

### Cambios realizados

1. Se añadió después de la agenda principal el CTA “Compartir con alguien que tiene terreno”.
2. En móvil usa Web Share; en escritorio copia texto + URL al portapapeles.
3. La URL compartida siempre es la canónica y lleva UTMs de referido:
   `utm_source=referral&utm_medium=share&utm_campaign=plan-potencial&utm_content=resultado`.
4. Un éxito dispara `ReferralShare` en Meta y añade `share:true` al beacon de actividad.
5. El módulo permanece disponible también en el cierre posterior a la agenda.
6. Se corrigió el respaldo local de Facebook al perfil `100075840402647`.
7. Se corrigió `TEXTOS POTENCIAL!B132` al mismo perfil activo.

### Contrato conservado

No se modificaron Apps Script, endpoint, folios, campos del lead, consentimiento, agenda ni reglas de
calificación. `share` es un campo adicional del beacon de actividad, no del payload de lead.

### Validación requerida

1. El CTA aparece debajo de la agenda, no antes del CTA principal.
2. El fallback copia texto y URL canónica con las cuatro UTMs.
3. Web Share cancelado no cuenta como compartir ni muestra error.
4. Un compartir exitoso marca `window.__didShare=true` y dispara `ReferralShare`.
5. El referido entrante conserva `utm_source=referral` como primer toque.
6. Después de agendar, el CTA de referido permanece visible.
7. El enlace de Facebook público y el valor del Sheet apuntan al ID `100075840402647`.

### Métrica para decidir si funcionó

Medir semanalmente: clics/activaciones de compartir, leads con `utm_source=referral`, tasa referido→lead y
referido→sesión agendada. La métrica principal es sesiones agendadas originadas por referidos, no clics ni
seguidores.

### Reversión

Revertir el commit de frontend y restaurar `TEXTOS POTENCIAL!B132` únicamente si la página oficial cambia de
identificador. No eliminar las UTMs de publicaciones nuevas.

## 2026-09-28 · Velocidad inicial y SEO técnico

### Objetivo

Mejorar la primera carga de la plataforma y facilitar que los buscadores entiendan cuál es la URL canónica,
qué representa la página y cuáles URLs públicas deben descubrir. La optimización no cambia el embudo ni sus datos.

### Evidencia previa

Auditoría Lighthouse sobre `https://yodesarrollomx.github.io/plan-potencial/` antes del cambio:

- Performance: **83/100**.
- SEO: **100/100**, pero sin canonical ni structured data reportados como auditorías aplicables.
- Recursos bloqueantes: ahorro estimado de **860 ms**.
- Transferencia total: **306 KiB**.
- Leaflet se descargaba en la primera pantalla aunque el mapa está en el paso 2.
- La carga inicial incluía Leaflet JS (~147.6 KiB de recurso), Leaflet CSS (~14.8 KiB), fuentes y Meta Pixel.

### Cambios realizados

1. Se retiraron Leaflet JS/CSS del `<head>`.
2. Se creó `cargarLeaflet()`: inserta la dependencia solo al entrar al mapa, con promesa compartida y mensaje
   de carga/error. El mapa de cierre reutiliza el mismo cargador.
3. Google Fonts pasó de stylesheet bloqueante a `preload` con fallback `<noscript>`.
4. Meta Pixel se inicia en la primera interacción (`pointerdown`, teclado o touch) o como respaldo 2 segundos
   después de `load`, y mantiene una cola para no perder `PageView`, `PasoEmbudo`, `Lead`, `Schedule` o
   `Contact` durante la carga diferida.
5. Se añadió canonical a la casa pública de `yodesarrollomx.github.io`.
6. Se añadieron robots index/follow, Open Graph, Twitter Card y schema `WebPage` + `Organization`.
7. El logo obtuvo `width`, `height`, `decoding` y `fetchpriority` para reservar espacio y reducir CLS.
8. Se añadió `sitemap.xml` con la landing y el aviso de privacidad.

### Contrato no modificado

No cambiaron Apps Script, endpoint, payloads, folios, UTMs, CRM, eventos, nombres de campos ni reglas de
captación. La única diferencia de analítica es el momento de inicialización; los eventos se encolan y luego se
envían en el mismo orden.

### Validación requerida

1. `node --check` sobre los scripts inline.
2. La carga inicial no solicita `leaflet.js` ni `leaflet.css`.
3. Al ir al paso 2, Leaflet se solicita una sola vez y el mapa se inicializa.
4. El flujo de cierre conserva mapa, ubicación y navegación.
5. La cola de Meta conserva eventos emitidos antes de la inicialización.
6. Canonical, robots, Open Graph, schema y sitemap son accesibles desde la versión pública.
7. Repetir Lighthouse con la misma URL y comparar FCP, LCP, TBT, recursos bloqueantes y transferencia.

### Reversión

Revertir el commit de este cambio y eliminar `sitemap.xml`. No revertir el Sheet ni Apps Script: esta mejora es
de frontend y metadatos, no de datos ni backend.

### Regla permanente

Toda dependencia nueva debe cargarse bajo demanda si no es necesaria para la primera pantalla. Todo cambio de
canonical, robots, schema o URL pública debe actualizar también este registro y la alerta de `CLAUDE.md`.

### Resultado público verificado

Lighthouse ejecutado sobre `https://yodesarrollomx.github.io/plan-potencial/?audit=public-final` después del
despliegue:

- Performance: **87/100** (antes **83/100**).
- SEO: **100/100**.
- FCP: **1.7 s** (antes **2.1 s**).
- LCP: **2.6 s**.
- TBT: **380 ms** (antes **490 ms**).
- CLS: **0**.
- Transferencia: **267 KiB** (antes **306 KiB**).
- Recursos bloqueantes: **ninguno detectado** (antes ahorro estimado de 860 ms).
- Leaflet en carga inicial: **0 solicitudes**; al abrir el paso 2: **1 JS + 1 CSS**.
- `sitemap.xml`: público, XML válido y con **2 URLs**.

Las métricas de Lighthouse pueden variar entre corridas; las comparaciones deben repetirse con la misma URL,
configuración y condiciones. La prueba funcional pública confirmó que la analítica diferida no se inicia antes de
la interacción/timeout y que el mapa conserva su instrucción después de cargar.

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
