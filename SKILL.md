---
name: especialista-inversiones-J-A-S
description: ESPECIALISTA EN INVERSIONES J-A-S — App web personal de Juan Carlos + Alba María para llevar el control del portafolio familiar (fondos, CDP, pólizas, bienes, cuentas) + Gastos Fijos por quincena. Single-file HTML/JS en Netlify, auth Gmail con whitelist, Google Sheets como base de datos, gráficos Chart.js, export Excel. Usar este skill cuando JC pida cualquier cambio o mejora al proyecto.
---

# Especialista Inversiones J-A-S

Contexto completo del proyecto **inversiones-J-A-S** para retomar trabajo sin perder contexto.

## Checkpoint 14 may 2026 (v1.3 — ranking de rendimiento en Fondos)

### Estado del proyecto
- **Producción:** Netlify (URL `*.netlify.app` — JC tiene la URL exacta) — auto-deploy desde `main`
- **Repo:** https://github.com/jhernandez-vibecode/inversiones-J-A-S
- **Local:** `C:\Users\segur\Documents\GitHub\inversiones-J-A-S\`
- **Rama activa:** `main`
- **Último commit (código):** v1.3 ranking de rendimiento (ver tabla de versionado abajo)
- **OAuth Client ID** (Google Cloud): `446215450096-i2s3glor63qodpf3t12ogdgunedqgp27.apps.googleusercontent.com`
- **Tamaño actual:** ~2566 líneas index.html

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
- HTML + Vanilla JS (single-file, ~2566 líneas)
- Outfit (Google Fonts) — toda la tipografía
- Chart.js v4.4 (donut + line)
- SheetJS v0.18 (export Excel `.xlsx`)
- pdfjs-dist v3.11 (parser PDF estados de cuenta BN)
- Google Identity Services (GIS) — auth Gmail
- Google Sheets API v4 — persistencia
- localStorage para `inv_sheet_id_v2` (cache del sheet ID por usuario) y `inv_user_v2` (cache user info)
- **Sin build step** — todo CDN

## Estructura de archivos

```
inversiones-J-A-S/
├── index.html                       # App completa SPA (~2566 líneas tras v1.3)
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

Se crea automáticamente en el primer login de cada usuario. ID guardado en `localStorage.inv_sheet_id_v2`.

**10 hojas (sheets) — orden y headers:**

| Hoja | Headers | Propósito |
|------|---------|-----------|
| `CONFIG` | tipo_cambio, updated | TC USD/CRC + último update |
| `FONDOS` | id, nombre, titular, saldo, moneda, tipo, vence | Fondos activos |
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
| **Google Drive API** | Crear el sheet la 1ra vez | sí | OAuth scope drive.file |

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
| **v1.3** | 14 may 2026 | **🏆 Ranking de Rendimiento en Fondos**. Card debajo de la tabla de fondos: <br>• Toggle **métrica**: 📈 Por tasa % / 💰 Por ganancias (USD equiv.)<br>• Toggle **período**: 📅 Por mes / 🗓️ Por año<br>• Selectores **año** + **mes** dinámicos (sólo opciones con datos en HISTORIAL)<br>• Ranking ordenado DESC con medallas 🥇🥈🥉 + posición #N<br>• Empty state cuando HISTORIAL está vacío o el período no tiene datos<br>• Tasa anual = compuesto ∏(1+t/100)−1, no suma simple<br>• Ganancias convierten cada fila con `toUSD()` según moneda del fondo<br>• Reutiliza `matchHistorialFondo()` para resolver el mapeo histórico→fondo (respeta multi-titular v1.2) |

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
- Verificar que el filtro "Movimientos por fondo" muestra el fondo de Alba correctamente DESPUÉS de que JC renombre f2 (de "Superfondo Dólares Plus- Alba" → "BN Internacional SUMA") desde Editar Fondos. El histórico de BN INTERNACIONAL SUMA en HISTORIAL ya existía y se asociará automáticamente al fondo renombrado.
- Confirmar que el gráfico del Dashboard ya muestra 4 líneas (Crecifondo, Superfondo $ JC, BN Internacional SUMA Alba, Superfondo Colones Alba). Después del rename, las que estaban en `(saldo actual)` discontinuas pasarán a línea sólida con datos reales.

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
- **% rendimiento anualizado** en filtro Movimientos
- **Comparativa entre fondos** en gráfico de barras (ganancias acumuladas)

### Nice-to-have del proyecto general
- Onboarding inicial primera visita (tour de las 8 pestañas)
- Página "FAQ" sobre el funcionamiento
- Backup automático del Sheet a Drive cada N días
- Importar de PDF también para tarjetas de crédito (no solo fondos BN)

## Recursos relacionados

- Repo: https://github.com/jhernandez-vibecode/inversiones-J-A-S
- Mockup intermedio v1.1: `inversiones-J-A-S/mockup-gastos-fijos.html` (untracked, se puede borrar tras validación)
- OAuth Client en Google Cloud Console: `446215450096-...` (Authorized JS origins debe incluir el dominio Netlify de JC)
