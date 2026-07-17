---
name: especialista-inversiones-J-A-S
description: ESPECIALISTA EN INVERSIONES J-A-S — App web personal de Juan Carlos + Alba María para llevar el control del portafolio familiar (fondos, CDP, pólizas, bienes, cuentas) + Gastos Fijos por quincena. Single-file HTML/JS en Netlify, auth Gmail con whitelist, Google Sheets como base de datos, gráficos Chart.js, export Excel. Usar este skill cuando JC pida cualquier cambio o mejora al proyecto.
---

# Especialista Inversiones J-A-S

Contexto completo del proyecto **inversiones-J-A-S** para retomar trabajo sin perder contexto.

## Checkpoint 17 jul 2026 (v1.6 — saldo provisional entre estados de cuenta)

### Estado del proyecto
- **Producción:** Netlify (URL `*.netlify.app` — JC tiene la URL exacta) — auto-deploy desde `main`
- **Repo:** https://github.com/jhernandez-vibecode/inversiones-J-A-S
- **Local:** `C:\Users\segur\Documents\GitHub\inversiones-J-A-S\`
- **Rama activa:** `main`
- **Último commit (código):** `c0a6655` — feat: v1.6 — saldo provisional entre estados de cuenta
- **Hoja de datos real de JC:** `10PPnb3czhFs0DHn1KF6LDd6xmVFWjG2SL0TRIShk9bQ` (creada 24 abr 2026)
- **OAuth Client ID** (Google Cloud): `446215450096-i2s3glor63qodpf3t12ogdgunedqgp27.apps.googleusercontent.com`
- **Tamaño actual:** ~2980 líneas index.html

### Qué está en producción

**8 pestañas funcionales:**
1. **📊 Dashboard** — KPIs del portafolio (Fondos USD/CRC, CDP, Bienes, Patrimonio total) + donut distribución + gráfico evolución mensual de los 4 fondos + tabla de todos los activos
2. **📈 Fondos** — Saldos actuales (4 fondos: Superfondo Dólares Plus JC, BN Internacional SUMA Alba, Crecifondo Colones JC, Superfondo Colones Alba) + alta/edita/baja + **edición del nombre desde Editar Fondos** (v1.2) + tipo de cambio editable + **🏆 Ranking de Rendimiento** (v1.3) con toggle métrica (tasa % / ganancias USD) + período (mes / año) + selectores año/mes
3. **📅 Historial** — Mensual por fondo (saldo inicial, aportes, retiros, tasa, intereses, saldo final, **titular**, **fecha import**) · selector de año · gráfico evolución de las 4 series · **importador PDF de estados de cuenta BN** (CreciFondo, Superfondo Dólares Plus, BN Internacional Suma) · **botón eliminar registros** · **panel "Movimientos por fondo"** abajo con filtro fondo/año + KPIs Saldo/Depósitos/Retiros/Ganancias + tabla mensual
4. **🏦 CDP** — Certificados de depósito (Marchamos, Adelantos Renta) + estado vigente/aplicado + filtros + abogado: Federico Altamura (8836-4617)
5. **🛡️ Pólizas** — Vida INS (Universal Dólares JC + Alba, ANDAS, Medical) + Pensión BN Vital
6. **🏠 Bienes** — Inmuebles (Apt. Nunciatura 2507, Casa Condominio 3-10) + Vehículos (Honda CRV, Toyota Land Cruiser) con iconos seleccionables
7. **💳 Cuentas** — Cuentas bancarias BN de JC + Alba (CRC y USD)
8. **💸 Gastos Fijos** — Pagos por quincena con toggle interactivo · KPIs · histórico mensual independiente · pagos ocasionales · modal editar gastos fijos · selector de mes con badges EN CURSO/CERRADO/FUTURO

---

## Stack técnico
- HTML + Vanilla JS (single-file, ~2980 líneas)
- Outfit (Google Fonts) — toda la tipografía
- Chart.js v4.4 (donut + line)
- SheetJS v0.18 (export Excel `.xlsx`)
- pdfjs-dist v3.11 (parser PDF estados de cuenta BN)
- Google Identity Services (GIS) — auth Gmail
- Google Sheets API v4 — persistencia
- localStorage = **caché** del sheet ID por usuario: `inv_sheet_id_v2::<email>` (v1.4). Desde **v1.5 la fuente de la verdad es Drive**: si el caché no está, la hoja se descubre con `files.list` y NUNCA se inventa. + sessionStorage `inv_user_v2` (cache user info)
- **Sin build step** — todo CDN

## Estructura de archivos

```
inversiones-J-A-S/
├── index.html                       # App completa SPA (~2980 líneas tras v1.6)
├── README.md                        # Descripción breve del proyecto
├── SKILL.md                         # Este archivo (sync con ~/.claude/skills/.../SKILL.md)
└── mockup-gastos-fijos.html         # Mockup intermedio v1.1 (untracked, se puede borrar)
```

## Auth y whitelist

**OAuth scopes:**
- `email profile`
- `https://www.googleapis.com/auth/spreadsheets`
- `https://www.googleapis.com/auth/drive.file` (solo archivos creados por la app)

**Cuentas autorizadas (`ALLOWED` en index.html):**
- `jhernandez@segurosdelins.com` (JC)
- `asistente@segurosdelins.com` (Alba María / asistente)

Otras cuentas reciben "Acceso denegado".

## Persistencia: Google Sheet "Control de Inversiones JC"

ID **cacheado** en `localStorage` bajo `inv_sheet_id_v2::<email>` (v1.4 — clave por usuario; se valida el acceso en cada login. La clave legacy compartida se adopta si es accesible con el token del usuario).

⚠️ **Desde v1.5 el caché NO es la fuente de la verdad: Drive lo es.** Si el caché no está (CCleaner,
"borrar datos de navegación"), la hoja **se descubre en Drive**; sólo se crea una nueva si no existe
ninguna Y JC lo confirma. Ver la sección *"La hoja de datos se descubre, no se inventa"* — es el
invariante más importante del proyecto y nació de un incidente real.

**10 hojas (sheets) — orden y headers:**

| Hoja | Headers | Propósito |
|------|---------|-----------|
| `CONFIG` | tipo_cambio, updated | TC USD/CRC + último update |
| `FONDOS` *(v1.6)* | id, nombre, titular, saldo, moneda, tipo, vence, **prov_fecha** | Fondos activos — range A:H (8 cols). `prov_fecha`≠'' ⇒ el saldo es provisional |
| `CDP` | id, nombre, titular, saldo, vence, estado, fecha_aplicado | Certificados depósito |
| `POLIZAS` | id, nombre, titular, monto, moneda, ahorro, tipo | Pólizas vida + pensión |
| `BIENES` | id, nombre, tipo, valor, detalle, icon | Inmuebles + vehículos |
| `CUENTAS` | id, titular, banco, moneda, desc | Cuentas bancarias |
| `HISTORIAL` *(v1.2)* | anio, mes, fondo, moneda, saldo_inicial, aportes, retiros, tasa, interes, saldo_final, **fecha_import**, **titular** | Movimientos mensuales — range A:L (12 columnas) |
| `GASTOS_FIJOS` | id, concepto, q1, q2, orden | Catálogo de gastos recurrentes (19 default) |
| `PAGOS_QUINCENA` | anio, mes, gasto_id, quincena | Log de pagos hechos (solo se guardan los pagados) |
| `OCASIONALES` | id, anio, mes, concepto, monto, quincena, pagado | Pagos no recurrentes por mes |

**Patrón save:** cada cambio reescribe la hoja completa con `saveSection(sheet, headers, rows)` — clear + set + actualiza CONFIG.updated. Para Gastos Fijos uso variantes silenciosas: `persistPagos()`, `persistOcasionales()`, `persistGastosFijos()` (sin toast "Guardado") porque cada toggle persiste y sería ruidoso.

**Seed:** la primera vez que una hoja está vacía, `autofillMissing()` la rellena con `DEFAULT` (objeto con datos iniciales hardcodeados). El botón **"♻️ Restaurar Excel"** del header borra todas las hojas y las repuebla con DEFAULT.

**Compatibilidad retro HISTORIAL:** los registros pre-v1.2 no tienen `titular`. La función `matchHistorialFondo(h, f)` los asigna al PRIMER fondo de D.fondos con mismo canónico+moneda (evita que aparezcan duplicados en gráfico/filtro cuando JC y Alba comparten nombre canónico).

## APIs externas

| API | Uso | Free tier | Auth |
|-----|-----|-----------|------|
| **Google Identity Services** | Login con whitelist | n/a | Client ID público |
| **Google Sheets API v4** | Persistencia (read/write) | sí | OAuth scope spreadsheets |
| **Google Drive API** | `files.list` — descubrir las hojas de la app (v1.5) | sí | OAuth scope drive.file |

⚠️ El sheet se **crea con la API de Sheets** (`POST sheets.googleapis.com/v4/spreadsheets`), no con la
de Drive. La API de **Drive** se empezó a usar en v1.5 y sólo para `files.list`: si está apagada en el
proyecto Cloud `446215450096`, el auto-descubrimiento muere con 403 (scope ≠ API habilitada).

## Diseño visual

**Paleta** (CSS vars en `:root`):
- `--navy:#0f2547` — header, títulos
- `--blue:#1a3a6e`
- `--cyan:#0ea5e9` — acento, tabs activos, KPI consol
- `--emerald:#10b981` — success, btn-add, KPI Pagado del mes
- `--amber:#f59e0b` — warnings, borde lateral KPI Q1/Q2 con pendiente
- `--red:#ef4444` — destructivo
- `--bg:#f1f5f9` — fondo
- `--card:#fff` — superficies
- `--border:#e2e8f0`
- `--text:#0f172a`
- `--muted:#64748b`

**Paleta para gráficos multi-serie:** `['#6366f1','#0ea5e9','#10b981','#f59e0b','#a855f7','#ec4899']` — rotativa por índice de fondo.

**Tipografía:** Outfit (300, 400, 500, 600, 700, 800).

**Patrones visuales:**
- Cards `border-radius:16px`, sombra suave `0 1px 4px rgba(0,0,0,.06)`, border `--border`
- KPIs en grid responsive con variantes `kc` (default), `ktotal` (gradiente navy), `kconsol` (gradiente cyan), `kpaid` (gradiente emerald)
- Badges pill `b-blue|purple|green|amber|red|gray`
- Tabs con `border-bottom-color: cyan` activo
- Tablas `.tc` con thead bg `#f8fafc`, hover `#fafbff`, footer `tfoot` para totales
- Modales con `.ov` overlay (backdrop blur) + `.m` card centrado, animación `slideUp`
- Toast bottom-right (`.toast-ok` navy, `.toast-warn` amber, `.toast-err` red)
- Líneas discontinuas (`borderDash:[6,4]`) para series que muestran "saldo actual" cuando no hay historial

## Funciones JS principales

### Auth + Sheets
- `initGSI()` — inicializa Google Identity Services
- `handleToken(resp)` — callback de OAuth, valida whitelist, crea sheet si no existe
- `ensureTabsExist()` — crea las 10 hojas faltantes
- `defaultPayload()` — devuelve headers + filas seed para todas las hojas
- `populateSheet()` — primer poblado del sheet
- `loadFromSheets()` — fetch en paralelo de 10 ranges, autofill de vacías, coerce de tipos
- `saveSection(sheet, headers, rows)` — clear + set + actualizar CONFIG.updated + toast "Guardado"

### Render por sección
- `renderAll()` — orquesta todos los renders
- `renderDash()` — KPIs + donut + line chart con 4 series (fallback "saldo actual" si no hay historial) + tabla todos los activos
- `renderFondos()`, `renderCDP()`, `renderPolizas()`, `renderBienes()`, `renderCuentas()`, `renderGastos()`
- `renderHistorial()` — gráfico 4 series + tabla con columnas titular/fecha_import + botón eliminar; al final llama `renderHistMovimientos()`
- `renderHistMovimientos()` *(v1.2)* — panel filtro fondo/año + 4 KPIs (saldo, dep, ret, gan) + tabla mensual
- `renderRankingFondos()` *(v1.3)* — card en pestaña Fondos: ranking ordenado DESC con medallas 🥇🥈🥉. Toggle métrica `tasa`/`interes` + período `mes`/`anio` + selectores año/mes dinámicos (solo se ofrecen los que tienen datos en HISTORIAL). Para ganancias convierte cada fila con `toUSD()`. Para tasa anual usa rendimiento compuesto `∏(1+t/100)-1`. Si ningún fondo tiene datos en el período, muestra empty state.
- `setRankMetric(m)` / `setRankPeriodo(p)` *(v1.3)* — cambian state global y re-renderizan solo el ranking

### Multi-titular *(v1.2)*
- `fondoCanonico(nombre)` — devuelve 'CRECIFONDO' / 'SUPERFONDO DOLARES PLUS' / 'BN INTERNACIONAL SUMA' o null
- `matchHistorialFondo(h, f)` — match historial entry vs D.fondo (canónico + moneda + titular). Compat retro: registros sin titular se asignan al primer fondo con mismo canónico+moneda.
- `prepareAddMov()` — popula select del modal Registrar Mes con D.fondos (no canónicos hardcoded)
- `deleteHistorial(anio, mes, fondo, titular)` *(v1.2)* — confirma + filtra + saveSection

### Importar PDF
- `handlePdfFiles(files)` — entry point del drop/file input
- `extractPdfText(file)` — pdfjs.getDocument + getTextContent por página
- `parseFondoPDF(text)` *(v1.2)* — detecta fondo por keywords en TODO el texto (más robusto que parsear "BN <nombre>" del header), extrae titular del PDF para distinguir JC/Alba
- `renderPdfResults()` — UI de stage con dropdowns Saltar/Reemplazar/Agregar (re-renderiza en cada cambio)
- `importPdfRows()` *(v1.2)* — guarda en HISTORIAL con fecha_import + titular, propaga saldo a D.fondos cuando matchea por canónico+moneda+titular

### Gastos Fijos
- `gIsPaidFijo(id, q)` — busca en `D.pagos_quincena` si está pagado en el mes actual
- `gOcasionalesMes()` — filtra ocasionales del mes actual
- `gMonthStatus(y, m)` — devuelve 'curso' | 'cerrado' | 'futuro'
- `gCellHTML(monto, paid, onclick)` — genera el botón toggle
- `togglePagoFijo(id, q)` — toggle + render + persist async
- `toggleOcasional(id)` — toggle + render + persist async
- `persistPagos()`, `persistOcasionales()`, `persistGastosFijos()` — saves silenciosos
- `prepareAddOcasional()` + `saveOcasional()` + `deleteOcasional(id)`
- `buildEditGastosForm()` + `addGastoRow()` + `removeGastoRow(i)` + `saveGastosFijos()`
- `changeGastoMes(delta)` — navega entre meses

### Modales
Patrón con `MOD_MAP` (clave → ID overlay) + `openMod(k)` que dispatcha al builder/preparer correspondiente. Hooks actuales: `addOcasional`, `editGastos`, `mov` *(v1.2 — popula con D.fondos)*, `addFondo`, `addCdp`, `addPoliza`, `addBien`, `addCuenta`, `bienes`, `polizas`, `cuentas`, `cdp`.

## Versionado

| Versión | Fecha | Highlights |
|---------|-------|-----------|
| **v1.0** | (previo) | 7 pestañas: Dashboard, Fondos, Historial, CDP, Pólizas, Bienes, Cuentas · Auth Gmail · Sheets API · gráficos · export Excel · importer PDF de BN |
| **v1.1** | 28 abr 2026 | **Pestaña Gastos Fijos** — 19 gastos seed · KPIs total/Q1/Q2/% pagado · toggle pagado interactivo · histórico mensual · pagos ocasionales · modal editar · 3 hojas nuevas. Commit `dcce5ab` |
| **v1.2** | 2 may 2026 | **Multi-titular en HISTORIAL + Filtros + UX fixes**. Hilo completo: <br>• `fix: parser PDF reconoce fondo correctamente y propaga saldo a pestaña Fondos` (`5f2eded`) — antes capturaba "BN Sociedad Administradora..." como nombre del fondo<br>• `feat: editar nombre de fondos + botón eliminar registros del historial` (`bc1c2f6`)<br>• `fix: dropdown Reemplazar/Saltar en importar PDF re-renderiza para mostrar el botón Importar` (`7383206`)<br>• `feat: fecha de import en historial + filtro de movimientos por fondo` (`bbe3e00`)<br>• `feat: HISTORIAL distingue JC vs Alba por titular en fondos compartidos` (`9f2fa3e`)<br>• `fix: filtro Movimientos muestra saldo actual del fondo + arregla duplicación de líneas` (`450fe76`)<br>• `fix: gráficos Dashboard e Historial muestran las 4 series siempre` (`344a2ac`) |
| **v1.3** | 14 may 2026 | **🏆 Ranking de Rendimiento en Fondos**. Card nueva debajo de la tabla de fondos. Hilo: <br>• `feat: ranking de rendimiento en pestaña Fondos (v1.3)` (`ea90468`) — toggle métrica tasa%/ganancias + toggle período mes/año + selectores dinámicos año/mes (solo con datos) + ranking DESC con medallas 🥇🥈🥉 + empty state. Tasa anual = compuesto ∏(1+t/100)−1. Ganancias en USD equiv. usando `toUSD()`. Reutiliza `matchHistorialFondo()` para respetar multi-titular v1.2.<br>• `feat: ranking de Fondos suma KPIs de total acumulado + promedio mensual` (`edb47cb`) — cuando la métrica es ganancias, aparecen 2 KPIs arriba de la tabla: 💰 Total acumulado (Σ ganancias del período) + 📊 Promedio mensual (total ÷ meses con datos en HISTORIAL). En vista mensual N=1 y se aclara en sub-texto. En vista anual usa solo meses con datos, no ÷12, así no se diluye con meses futuros. |
| **v1.6** | 17 jul 2026 | **Saldo provisional entre estados de cuenta** (`c0a6655`) — JC anota el saldo total que ve en el banco a mitad de mes, se refleja al instante marcado como provisional, y el PDF/registro de mes lo confirma y pisa. Ver sección "Saldo provisional entre estados de cuenta". FONDOS +col `prov_fecha` (A:H), centralizado en `FONDOS_HDR`/`fondoRow()`/`saveFondosSheet()`. Botón `🕑 Saldo al día` + modal `ovProvisional` + `.prov-tag` + `quitarProvisional()`. |
| **v1.5** | 17 jul 2026 | **La app nunca más inventa una hoja** (`0e0e8f6`) — a raíz del incidente CCleaner del 17 jul (ver sección "La hoja de datos se descubre, no se inventa"). `buscarHojasApp()` + `esHojaInversiones()` + `resumenHoja()` + `pedirHoja()` + `conectarHoja()` + `openHojaPanel()`. Login: sin caché se le pregunta a Drive; 1 hoja se reconecta sola, varias se eligen, ninguna se ofrece crear. Nuevo botón **🔗 Hoja** en el header. Modal `ovHoja` (fuera de `#app`, visible en login). `♻️ Restaurar Excel` pide escribir `RESTAURAR`. Global `USER_KEY`. |
| **v1.4** | 11 jun 2026 | **Hardening total** (`6aab6a4`) — auditoría línea por línea + revisión adversaria multi-agente (16 hallazgos confirmados, 0 refutados). <br>**Datos:** Restaurar Excel limpia las 10 hojas (antes 7) · `loadFromSheets` aborta si CUALQUIER lectura falla (`shGetSafe` devuelve `null`, nunca `[]`) para que el autofill jamás pise datos reales · `shClear` lanza en error · try/catch+toast en los 6 saves de modales. <br>**Auth:** sheet ID por usuario `inv_sheet_id_v2::<email>` + validación de acceso en cada login + adopción de clave legacy · renovación automática de token en 401 (`authFetch` + `refreshAccessToken` singleton con timeout 15s por identidad + `error_callback` + `hint` de cuenta en TODOS los `requestAccessToken`) · `showApp` corre DESPUÉS de resolver el sheet · errores siempre visibles (toast si app abierta, `#loginError` si no). <br>**Fechas:** `parseFechaLocal`/`fmtFecha` (CR es UTC-6: `2026-12-01` ya no se muestra como 30 nov) · `hoyLocal()` en fecha_aplicado/fecha_import/nombre Excel (toISOString daba el día siguiente después de 6pm) · CDP sin fecha → badge "Sin fecha". <br>**UI:** Dashboard excluye CDPs aplicados de KPI/donut/Patrimonio/tabla (`cdpVig`) y `markCDP` llama `renderAll()` · años dinámicos en Historial (`histYearRow`/`setHistYear`), modal Registrar mes y gráfico Dashboard (cero años hardcodeados) · `escAttr` en todos los value de formularios · Excel exporta Estado + Fecha Aplicado de CDPs. <br>**Nota:** quedó anotado como deuda conocida (no bloqueante): `saveSection` clear→write no atómico, y escape de texto HTML (self-XSS) fuera de atributos. |

## Saldo provisional entre estados de cuenta (v1.6 — 17 jul 2026)

**Problema:** el saldo de un fondo solo se movía al cargar el PDF de fin de mes. JC invierte el
15 y no lo veía hasta el 30 (en el Excel viejo actualizaba el dato al día cada quincena).

**Modelo (decidido con JC):** anota el **saldo total que ve en el banco** a mitad de mes; la app
lo pone como `f.saldo` (SET, no suma) y lo marca provisional con `f.prov_fecha=hoyLocal()`. Se
refleja al instante en Dashboard y Fondos (todo lee `f.saldo`, así que "dato al día" sale gratis).
Cuando llega el estado de cuenta, **el saldo oficial MANDA**: pisa `f.saldo` y limpia `prov_fecha`.

**Conciliación (2 caminos, mismo efecto):**
- `importPdfRows()`: al propagar el último mes a un fondo, `f.saldo=oficial; f.prov_fecha=''`. El
  guard `>0.5` se amplió para disparar también cuando había `prov_fecha` (por si el provisional
  coincidía con el oficial).
- `saveMov()` (registrar mes manual): si el mes registrado es el **más reciente** de ese fondo,
  fija `f.saldo=saldo_final` y limpia la marca. Un mes atrasado NO toca el saldo actual.
- `quitarProvisional(id)`: escape manual, saca la marca sin cambiar el saldo.

**UI:** botón `🕑 Saldo al día` + modal `ovProvisional` (select fondo, saldo mostrado disabled,
input del saldo del banco) en la pestaña Fondos. Etiqueta `.prov-tag` "provisional · fecha ·
quitar" en la fila + banner ámbar arriba. Export Excel: columna Estado (Confirmado/Provisional).

**⚠️ Esquema FONDOS centralizado — usar SIEMPRE el helper.** `FONDOS_HDR` + `fondoRow(f)` +
`saveFondosSheet()` son el ÚNICO lugar que define columnas y guarda FONDOS. Los ~4 call sites
(saveFondos, addFondo, delFondo, importPdfRows) llaman `saveFondosSheet()`, no `saveSection('FONDOS',…)`
inline. Agregar un campo a FONDOS = tocar solo esas 3 líneas + el range de lectura (`FONDOS!A:H`,
2 sitios en loadFromSheets) + la coerce. Esto mata la trampa "contagiosa" que sufrió HISTORIAL.
**Retrocompat:** hojas viejas sin la columna leen `prov_fecha=''` (no provisional) y se actualizan
solas al primer guardado de FONDOS.

## 🔴 La hoja de datos se descubre, no se inventa (v1.5 — 17 jul 2026)

**Invariante:** el `localStorage` es un **caché** de la dirección de la hoja, NUNCA la fuente
de la verdad. La fuente de la verdad es **Drive**. Un limpiador de disco (CCleaner, "borrar
datos de navegación") borra el caché sin aviso y sin vuelta atrás; Drive sobrevive.

**El incidente (17 jul 2026):** CCleaner borró `inv_sheet_id_v2::<email>`. La app concluyó
"no hay ID → usuario nuevo → creo hoja", creó `1CLjNtO…` y la sembró con `DEFAULT`. Como
`DEFAULT` es el portafolio real hardcodeado de cuando se armó la app, JC no vio una hoja
vacía: vio **su portafolio atrasado a abril**, y creyó que se había borrado la data. La hoja
buena (`10PPnb3…`, historial hasta JUNIO 2026) siguió intacta en Drive todo el tiempo.
Recuperación: reapuntar el `localStorage` al ID viejo.

**Flujo de resolución del sheet en `handleToken` (no romper):**
1. Caché por usuario → `sheetAccesible()` → usar.
2. Clave legacy → validar → usar.
3. **`buscarHojasApp()`** (Drive `files.list`, scope `drive.file` ⇒ solo hojas de esta app),
   filtrado por **`esHojaInversiones()`** (debe tener tabs CONFIG+FONDOS+HISTORIAL).
   - **1 hoja** → se adopta sola + toast "Reconecté tu hoja". *Este es el caso del CCleaner:
     se auto-cura y JC ni se entera.*
   - **>1** → `pedirHoja()`, JC elige.
   - **0** → recién ahí se ofrece crear.
4. **Nunca crear sin confirmación explícita.** Este punto solo habría evitado el incidente.

**Trampas ya pisadas — no repetirlas:**
- ❌ **No desempatar por fecha de modificación.** El 17 jul la hoja inventada era la más
  reciente y la buena parecía la vieja. El criterio útil es **cuánto historial trae y hasta
  qué mes llega** (`resumenHoja()`), que es lo que muestra el selector.
- ❌ **No filtrar por nombre** en `buscarHojasApp()`: renombrar la hoja la volvería invisible.
  El filtro correcto es estructural (`esHojaInversiones`).
- ❌ **`#ovHoja` NO puede cerrarse solo quitando la clase `open`.** Es una promesa que el
  login está esperando: hay que resolverla (`_hojaCerrar`) o el `await` queda colgado y JC
  ve un login muerto. Por eso está excluido del listener genérico de `.ov`.
- ⚠️ **La API de Drive es nueva en esta app** (la hoja se crea con la de *Sheets*). El scope
  `drive.file` ≠ API habilitada. Si está apagada en el proyecto Cloud `446215450096`,
  `files.list` da 403 y el auto-descubrimiento muere. `buscarHojasApp()` traduce ese 403 a
  un mensaje accionable.
- ⚠️ **`CLIENT_ID` debe seguir siendo exclusivo de esta app.** `drive.file` es por Client ID:
  si se reusa en otra app que cree hojas, aparecerían aquí. `esHojaInversiones()` es el
  candado, pero la regla sigue viva. (Verificado 17 jul: Control-Comisiones usa un Client ID
  distinto y ni siquiera pide `drive.file`.)

**`♻️ Restaurar Excel` es la otra puerta al desastre:** borra las 10 hojas y siembra `DEFAULT`.
Desde v1.5 pide escribir `RESTAURAR`. El `emptyBanner` del Dashboard también lo llama, pero
es **UI muerta** (`loadFromSheets` siempre lo esconde y nada lo muestra).

## Reglas de desarrollo

🔴 **CRÍTICO** — App es **PERSONAL/familiar** (JC + Alba):
- NO aplican guardrails financieros corporativos
- Tono directo, en español de CR
- Datos personales sensibles — siempre auth + whitelist

🔴 **NUNCA** commitear secrets:
- CLIENT_ID es público (OAuth web client) — OK en repo
- No hay API keys server-side — todo el auth es browser-side con OAuth user
- Datos viven en el Google Sheet del usuario (no en backend nuestro)

⚠️ **Reglas técnicas**:
- Single-file HTML — no build step, no servidor
- Tailwind NO se usa — todo CSS custom con variables (`var(--cyan)` etc.)
- Iconos = emojis (consistencia con resto del app — JC usa emojis intencionalmente)
- Persistencia patrón: clear + set hoja completa (no append por fila)
- `data-tab="<seccion>"` en nav-tab → `id="tab-<seccion>"` en `.section`
- `MOD_MAP[clave] = 'ovOverlayId'` — si agregás modal, agregalo al MOD_MAP
- Si agregás columna a HISTORIAL: actualizar (1) defaultPayload header, (2) shGetSafe range A:L → A:M, (3) los 3 saves de saveSection('HISTORIAL',...) — son `deleteHistorial`, `saveMov`, `importPdfRows`. Olvidar uno corrompe el sheet.
- Si agregás columna a FONDOS: desde v1.6 está centralizado. Tocar solo `FONDOS_HDR` + `fondoRow()` (un lugar) + los 2 rangos de lectura `FONDOS!A:H` en loadFromSheets + la coerce. Todos los guardados usan `saveFondosSheet()`; NO agregar `saveSection('FONDOS',…)` inline. Replicá este patrón (HDR const + rowMapper + saveXSheet) si otra hoja gana columnas.

⚠️ **Multi-titular HISTORIAL** *(v1.2)*:
- El historial guarda fondo **canónico** (`CRECIFONDO`/`SUPERFONDO DOLARES PLUS`/`BN INTERNACIONAL SUMA`) + titular del PDF.
- D.fondos puede tener varios fondos con mismo canónico+moneda (ej: "Superfondo Dólares Plus" para JC y Alba).
- `matchHistorialFondo(h, f)` desempata por palabras del titular. Sin titular: asigna al primer fondo de D.fondos con mismo canónico+moneda.
- Gráficos del Dashboard e Historial: si un fondo no tiene movimientos en el año, dibujan línea constante con el `saldo actual` (estilo discontinuo + sufijo "(saldo actual)" en leyenda).

⚠️ **Sync SKILL.md** si hay cambios estructurales — sincronizar `~/.claude/skills/especialista-inversiones-J-A-S/SKILL.md` con la copia del repo.

⚠️ **Deploy**: push a `main` → Netlify auto-deploy en ~30s. JC autoriza pushes directos a main para cambios validados (igual que en sus otros proyectos `trading` y `reclamos-sdi`).

⚠️ **Lección aprendida v1.2**: Cambios al schema de HISTORIAL son contagiosos (5 sitios mínimo). En la sesión del 2 may hubo que iterar varias veces porque introduje el filtro/multi-titular sin mapear todos los puntos primero. Para futuros cambios de schema, **mapear todos los call sites antes de empezar a editar**.

## Pendientes / próximos pasos

### Validación pendiente con JC
- **v1.3 ranking** — Validar en producción que con datos reales el ranking ordena correctamente, que los selectores año/mes filtran lo esperado, y que los KPIs de Total + Promedio mensual cuadran con lo que JC sabe de su portafolio.
- (Pendiente desde v1.2) Verificar que el filtro "Movimientos por fondo" muestra el fondo de Alba correctamente DESPUÉS de que JC renombre f2 (de "Superfondo Dólares Plus- Alba" → "BN Internacional SUMA") desde Editar Fondos. El histórico de BN INTERNACIONAL SUMA en HISTORIAL ya existía y se asociará automáticamente al fondo renombrado.
- (Pendiente desde v1.2) Confirmar que el gráfico del Dashboard ya muestra 4 líneas (Crecifondo, Superfondo $ JC, BN Internacional SUMA Alba, Superfondo Colones Alba). Después del rename, las que estaban en `(saldo actual)` discontinuas pasarán a línea sólida con datos reales.

### Features futuras propuestas
- **Vista anual de Gastos Fijos** — tabla 12 meses × N gastos con totales por mes
- **Recurrencia configurable** — gastos cada 2 meses, anuales, etc.
- **Alertas de vencimiento** — días para pagar antes del 17 / 02
- **Categorías de gastos** — agrupar con totales por categoría
- **Comparativa mes vs promedio** — KPI "este mes vs promedio últimos 6"
- **Export Gastos Fijos a Excel** — agregar las 3 hojas nuevas al export
- **Debounce en toggles** — agrupar requests si JC marca varios pagos seguidos
- **Edit inline de monto en celda** — sin abrir modal completo
- **Dark mode** — toggle en header, persistir en localStorage
- **Comparativa entre fondos** en gráfico de barras (ganancias acumuladas) — el ranking v1.3 lo muestra en tabla, falta visualización gráfica
- **Exportar ranking a Excel** — agregar hoja con el ranking del año actual
- **Drill-down del ranking** — click en una fila del ranking → ver el detalle mensual del fondo (gráfico + tabla)

### Nice-to-have del proyecto general
- Onboarding inicial primera visita (tour de las 8 pestañas)
- Página "FAQ" sobre el funcionamiento
- Backup automático del Sheet a Drive cada N días
- Importar de PDF también para tarjetas de crédito (no solo fondos BN)

## Recursos relacionados

- Repo: https://github.com/jhernandez-vibecode/inversiones-J-A-S
- Mockup intermedio v1.1: `inversiones-J-A-S/mockup-gastos-fijos.html` (untracked, se puede borrar tras validación)
- OAuth Client en Google Cloud Console: `446215450096-...` (Authorized JS origins debe incluir el dominio Netlify de JC)
