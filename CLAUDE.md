# Plan de Potencial — *el formulario "Despierta tu Terreno"* (imán de leads de Yodesarrollo)

Lee este archivo completo antes de tocar nada.

## Qué es

Formulario público por pasos para **dueños de terreno**. Es el PRIMER contacto del embudo de
co-desarrollo. Un solo HTML estático, sin build ni dependencias de servidor (`index.html`, 1,038 líneas).
Lo usa: cualquier persona que llega de redes/yodesarrollo.mx. Lo administra Alejandro desde el Sheet.

**Direcciones (comprobadas con curl el 2026-09-04):**

| Dirección | HTTP | Qué es |
|---|---|---|
| `https://yodesarrollomx.github.io/plan-potencial/` | **200** | **La casa canónica.** Sirve el mismo `index.html` que este repo (md5 `6e5a47b6…` idéntico al local) |
| `https://yodesarrollomx.github.io/plan-potencial/board.html` | **200** | Tablero interno del embudo (md5 idéntico al local `546db85a…`) |
| `https://alexpueblag.github.io/plan-potencial/` | 200 | Cascarón: 63 líneas que redirigen a la casa nueva conservando `?query` y `#hash` |
| `https://yodesarrollo.github.io/plan-potencial/` | 200 | **Copia vieja TODAVÍA VIVA con la app completa** (no es cascarón). Ver Hallazgos |
| `https://tableros.yodesarrollo.mx/plan-potencial/` | **000** | El dominio propio no existe todavía en el DNS (curl no resuelve) |

**El embudo (esto es negocio, no diseño — README.md:8-19 y memoria `codesarrollo-funnel`):**
1. Este formulario capta interés + datos del terreno.
2. **Videollamada GRATIS** — ahí el Masterdeveloper explica el Plan y su costo.
3. **Plan de Potencial = SERVICIO DE PAGO.**
4. **Co-desarrollo solo si los números califican.** No es automático.

## Reglas INVIOLABLES

- **Cero dinero en pantalla.** El formulario no muestra precios ni regala análisis: su único trabajo es
  agendar la videollamada. *(Si aparece una cifra, el embudo miente y quema la llamada de venta.)*
- **Nunca decir "Plan de Potencial gratis".** Lo gratis es la videollamada; el Plan se cobra.
  *(Decisión de Alejandro 2026-07-21, memoria `codesarrollo-funnel`: rompe la venta y la honestidad.)*
- **Nunca prometer co-desarrollo automático.** Siempre "si tu terreno califica". *(Filtro real.)*
- **El Sheet manda sobre el código.** Los ~133 textos del formulario se editan en `TEXTOS POTENCIAL`;
  `const TEXTOS` de `index.html:535` es solo respaldo. *(Si alguien "arregla" copy en el HTML, el Sheet
  se la pisa al siguiente load.)*
- **`docs/webhook-apps-script.gs` y `docs/tarea-analisis-potencial.md` NO se publican** (`.gitignore:4` y `:5`; `docs/brief-board-redes.md` va en `:7`). Comprobado: `git ls-files docs` solo lista `docs/paridad.md`, y los dos dan **404** en `yodesarrollomx.github.io/plan-potencial/docs/…` (curl 2026-09-04).
  *(Llevan nombres de token y el esquema del CRM.)*
- **Ningún secreto vuelve al repo.** `BOARD_SECRET` vive en Propiedades del script desde el commit
  `563ac5b`. *(Estuvo publicado en el README y en el .gs; se rotó el 1-ago, memoria `secretos-rotacion-1ago`.)*
- **Nunca POST a producción para probar.** El endpoint crea leads reales en el CRM. Los GET
  (`?recurso=textos`, `?recurso=board`) sí son seguros.
- **`revisarCitasAgendadas_` es el único que cierra el North Star.** Sin ese cruce con Calendar, el board
  solo ve "agendó (clic)" y nunca "cita confirmada". *(README.md sección Board.)*
- **La landing queda abierta; el tablero pide cuenta.** `?recurso=textos` y el POST de leads son públicos
  a propósito (captación); `?recurso=board` exige credencial del Portero (commit `0de7eb8`).
- **Correo único de YOD = `comercial@yodesarrollo.mx`** (memoria `plan-potencial-project`).

## Archivos

- `index.html` — la app completa: portada + 5 pasos + cierre, mapa con pin (Leaflet+satélite Esri),
  barra viva cualitativa, rama "otra forma" (capital/colaborar/licencia → `tipo:"lead_interes"`, línea 871),
  gate de datos, pantalla post-agenda. Constantes vivas: `WEBHOOK_URL` (:471), `AGENDA_URL` (:474),
  `META_PIXEL_ID` (:478, valor `1563269895817275` — el mismo píxel que arquitectura, se separa por
  `form:'potencial'`, commit `fe4914c`).
- `board.html` — tablero del embudo (162 líneas). Lee `?recurso=board`, manda la clave guardada en
  `localStorage.pyod_clave_v1` y carga el portero compartido
  `https://yodesarrollomx.github.io/potenciales-yod/portero.js` (:160, commit `2d9c3b7`).
- `aviso-privacidad.html` — aviso LFPDPPP 2025 (commit `4b83f73`).
- `docs/webhook-apps-script.gs` — ESPEJO del backend (908 líneas). No se publica.
- `docs/tarea-analisis-potencial.md` — prompt de la rutina diaria. No se publica.
- `docs/paridad.md` — mapeo campo por campo de los 2 Google Forms viejos → el flujo nuevo (0 huérfanos).
- `docs/brief-board-redes.md` — brief del agente del board. No se publica.
- `img/yod-logo.png` — logo oficial YOD.

## Arquitectura de datos

El Sheet es la raíz. Nunca invertir la dirección.

```
Sheet "CRM - YOD"  (id 1z1ZtvcUKnx4MUfxLICo8x5bTihlDY8tBC3j2sYwNvg8 · .gs:39)
  · TEXTOS POTENCIAL   (clave/valor, ~133 claves)   ── GET ?recurso=textos ──→ index.html
  · LEADS - POTENCIAL  (1 renglón por folio POT-…)  ←── UPSERT por folio ─── doPost
  · ACTIVIDAD POTENCIAL (1 fila/día, embudo denso)  ←── beacon tipo:"actividad"
  · GASTO POTENCIAL    (gasto de pauta semanal)     ←── POST tipo:"gasto" + BOARD_SECRET

index.html ──POST no-cors (+cola de reintento en localStorage `yod_pot_pendientes`, máx 3)──┐
index.html ──sendBeacon en pagehide/visibilitychange (:992-1002)───────────────────────────┤
                                                                                            ▼
                                      Apps Script Web App  /exec
                                      AKfycbw3EB-6Q9Mq-ouDU-JvKMrRUaw4auYVeGkKja783yJ7_dEpCOW8xoMhs8IQMDojmlDB3A
                                      doGet:  textos (abierto) · board (pide credencial)
                                      doPost: lead · actividad · gasto · estado · revisar_citas
                                                                                            │
board.html ──GET ?recurso=board&k=<clave del Portero>───────────────────────────────────────┘
tarea diaria 8:37 AM ──marcar_estado.sh──→ POST tipo:"estado" (marca seguimiento, nunca degrada)
trigger propio 7 AM ──revisarCitasAgendadas_()──→ cruza Calendar y marca "SESIÓN AGENDADA"
```

**⚠️ EL REPO ES ESPEJO.** Lo que corre es lo pegado en el editor de Apps Script, no
`docs/webhook-apps-script.gs`. **Prueba dura de que ya divergen (hoy 2026-09-04):** el espejo llama
`construirBoard_()` sin revisar credencial (`.gs:83`), pero el `/exec` en vivo contesta
`{"ok":false,"error":"liga"}` a `?recurso=board` sin clave. El candado existe solo en el desplegado.
Antes de tocar el .gs: **pide el Code.gs del editor** (memoria `backend-vivo-no-es-el-repo`).
Redespliegue = pegar + Implementar → Administrar → Editar → **Nueva versión** (la URL no cambia).

## Decisiones

- **2026-06-25 · Alejandro** — Reemplaza los 2 Google Forms viejos (Plan de Potencial + "Desarrollemos
  juntos"); capital/colaborar/licencia van a la rama corta "otra forma". *Porqué: un solo embudo con datos
  estructurados.* (memoria `plan-potencial-project`)
- **2026-06-25 · Alejandro** — Paleta REAL = blanco/crema + negro `#0c0c0c` + petróleo `#013e42`.
  ~~Vino `#703438`~~ **OBSOLETO desde 2026-06-25** (fue error de medición: mucho conteo, área diminuta).
  El token `--vino` quedó como ALIAS de `#013e42` (`index.html:20`) para no romper usos. *No renombrarlo.*
- **2026-06-25 · Alejandro** — Respuesta prometida = "menos de 12 horas" (no 48). *El "48 h" implicaba
  análisis gratis rápido.* Y no se pregunta el fondo del terreno: se deriva m²÷frente. (commit `e2a709a`)
- **2026-06-26 · panel de conversión** — Mapa con pin en el paso 2 y miniatura satelital en el cierre.
  *Gratificación tangible sin regalar el Plan de pago.* El gauge de m² construibles se dejó FUERA a
  propósito: rozaría el producto cobrado. (commit `5c3bcea`)
- **2026-06-30** — El board distingue explícitamente "agendó (clic)" de "cita confirmada (CRM)".
  *Con 12 visitas reales se vio que `kpis.agendadas` era 0 siempre; mentir sobre el North Star es peor
  que no medirlo.* (commit `130ff66`)
- **2026-06-30** — Cola de reintento de leads en `localStorage`. *`no-cors` nunca confirma éxito y
  `upsertLead_` es idempotente por folio, así que reintentar es seguro.* (commit `3eaf447`)
- **2026-07-21 · Alejandro** — CTA correcto = "Agenda tu videollamada gratis". El Plan de Potencial NO
  es gratis y NO va con sello de 48 h. (memoria `codesarrollo-funnel`)
- **2026-07-25 · Alejandro** — **El Plan de Potencial ya NO es un estudio de una entrega: es una GUÍA
  con ACOMPAÑAMIENTO MENSUAL de pago, de hasta un año.** *Es una relación en el tiempo, no un documento.*
  ⚠️ Esto **todavía no está escrito en el copy de este repo** (memoria `codesarrollo-funnel`).
- **2026-07-30** — `TOKEN_TAREA` rotado (el viejo `YOD-POT-TAREA-…` quedó filtrado en el historial público
  y se comprobó explotable). Vive en Propiedades del script + en `marcar_estado.sh` de la tarea.
  ~~Token literal en el .gs y en la doc~~ **OBSOLETO desde 2026-07-30.** (memoria `yod-auditoria-total-29jul`)
- **2026-08-01 · Alejandro ("si debe de pedir cuenta el embudo")** — `?recurso=board` pide credencial del
  Portero; `textos` y el POST de leads siguen abiertos *porque alimentan la captación*. (commit `0de7eb8`)
- **2026-08-01** — `BOARD_SECRET` sale del repo a Propiedades del script y **falla cerrado**: sin valor no
  pasa nadie (`.gs:58-63`). *Estaba publicado en el README.* (commit `563ac5b`)
- **2026-09-01 · mudanza** — Casa canónica = `yodesarrollomx.github.io`; el portero se carga de ahí.
  ~~`alexpueblag.github.io/potenciales-yod/portero.js`~~ **OBSOLETO desde 2026-09-01.** (commit `2d9c3b7`)
- **2026-09-04** — Este `CLAUDE.md` se escribe por primera vez. *No existía; el README solo cubría
  el "cómo se opera", no el "por qué" ni lo obsoleto.*

## Pendientes

| Tema | Dueño | Evidencia para darlo por cerrado |
|---|---|---|
| Copy del producto: el Plan es guía + acompañamiento mensual (~1 año), no un estudio | Alejandro (decide) → luego el repo | Las claves de `TEXTOS POTENCIAL` editadas y visibles en la página en vivo |
| DNS de `tableros.yodesarrollo.mx` (bloqueado en Miguel Reina / cPanel) | Alejandro | `curl -o /dev/null -w "%{http_code}" https://tableros.yodesarrollo.mx/plan-potencial/` → 200 |
| La copia `yodesarrollo.github.io/plan-potencial/` sigue sirviendo la app completa con `board.html` viejo | Alejandro (decide si cascarón o borrar) | md5 de su `index.html` y `board.html` = cascarón, o 404 |
| Los botones de terreno de yodesarrollo.mx (Sheet "YO DESARROLLO - SERVIDOR", hoja WEBPAGE) apuntan a la casa vieja | Alejandro | Ver el DOM del sitio en vivo con la URL nueva |
| CAPI server-side de Meta apagada (`META_PIXEL_ID` y `META_CAPI_TOKEN` vacíos en el .gs, :72-73) | Alejandro (token de Events Manager) | Eventos `Lead` deduplicados por `event_id` en Events Manager |
| Gasto de pauta sin cargar → costo por lead y por cita en $0 | Alejandro | Filas reales en `GASTO POTENCIAL` y KPIs de costo no-cero en el board |
| `docs/tarea-analisis-potencial.md` conserva el placeholder del token en "Constantes" | quien toque la doc | El .md sin la línea `TOKEN_TAREA = …` |

## Por confirmar (NO afirmar sin preguntar)

- ¿La copia `yodesarrollo.github.io/plan-potencial/` debe volverse cascarón como la de `alexpueblag`, o borrarse? *(hoy sirve la app entera y su board carga el portero de la casa vieja)*
- ¿La rutina diaria `plan-potencial-analisis-diario` sigue corriendo y dejando borradores? *(el SKILL.md existe en `~/.claude/scheduled-tasks/`, última edición 2026-08-29; no verifiqué corridas)*
- ¿Los botones del Sheet WEBPAGE de yodesarrollo.mx ya apuntan a `yodesarrollomx.github.io`? *(no abrí el sitio)*
- ¿Cuántos leads reales hay hoy en `LEADS - POTENCIAL`? *(el board pide credencial y no la tengo)*
