---
name: especialista-inversiones-J-A-S
description: ESPECIALISTA EN INVERSIONES J-A-S — App web personal de Juan Carlos + Alba María para llevar el control del portafolio familiar (fondos, CDP, pólizas, bienes, cuentas) + Gastos Fijos por quincena. Single-file HTML/JS en Netlify, auth Gmail con whitelist, Google Sheets como base de datos, gráficos Chart.js, export Excel. Usar este skill cuando JC pida cualquier cambio o mejora al proyecto.
---

# Especialista Inversiones J-A-S

Contexto completo del proyecto **inversiones-J-A-S** para retomar trabajo sin perder contexto.

## Checkpoint 28 abr 2026 — noche (v1.1 con Gastos Fijos pusheado a main)

### Estado del proyecto
- **Producción:** Netlify (URL `*.netlify.app` — JC tiene la URL exacta) — auto-deploy desde `main`
- **Repo:** https://github.com/jhernandez-vibecode/inversiones-J-A-S
- **Local:** `C:\Users\segur\Documents\GitHub\inversiones-J-A-S\`
- **Rama activa:** `main`
- **Último commit:** `dcce5ab` — feat: pestaña Gastos Fijos por quincena con histórico mensual
- **OAuth Client ID** (Google Cloud): `446215450096-i2s3glor63qodpf3t12ogdgunedqgp27.apps.googleusercontent.com`

### Qué está en producción

**8 pestañas funcionales:**
1. **📊 Dashboard** — KPIs del portafolio (Fondos USD/CRC, CDP, Bienes, Patrimonio total) + donut distribución + gráfico evolución mensual + tabla de todos los activos
2. **📈 Fondos** — Saldos actuales de fondos de inversión (Superfondo $, BN Internacional, Crecifondo CRC, Superfondo CRC) + alta/edita/baja + tipo de cambio editable
3. **📅 Historial** — Mensual por fondo (saldo inicial, aportes, retiros, tasa, intereses, saldo final) · selector de año (2025/2026) · gráfico evolución · **importador PDF de estados de cuenta BN** (CreciFondo, Superfondo Dólares Plus, BN Internacional Suma)
4. **🏦 CDP** — Certificados de depósito (Marchamos, Adelantos Renta) + estado vigente/aplicado + filtros + abogado: Federico Altamura (8836-4617)
5. **🛡️ Pólizas** — Vida INS (Universal Dólares JC + Alba, ANDAS, Medical) + Pensión BN Vital
6. **🏠 Bienes** — Inmuebles (Apt. Nunciatura 2507, Casa Condominio 3-10) + Vehículos (Honda CRV, Toyota Land Cruiser) con iconos seleccionables
7. **💳 Cuentas** — Cuentas bancarias BN de JC + Alba (CRC y USD)
8. **💸 Gastos Fijos** *(v1.1 — 28 abr 2026)* — Pagos por quincena con toggle interactivo (click = marcar pagado, click de nuevo = deshacer) · KPIs total/pendiente Q1/Q2/% pagado · histórico mensual independiente · pagos ocasionales · modal editar gastos fijos · selector de mes con badges EN CURSO/CERRADO/FUTURO

**Sin pendientes bloqueantes.** El feature de Gastos Fijos quedó deployado y JC va a probarlo en producción para validar.

---

## Stack técnico
- HTML + Vanilla JS (single-file, ~2200 líneas tras v1.1)
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
├── index.html                       # App completa SPA (~2200 líneas tras v1.1)
├── README.md                        # Descripción breve del proyecto
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
| `HISTORIAL` | anio, mes, fondo, moneda, saldo_inicial, aportes, retiros, tasa, interes, saldo_final | Movimientos mensuales |
| `GASTOS_FIJOS` *(v1.1)* | id, concepto, q1, q2, orden | Catálogo de gastos recurrentes (19 default) |
| `PAGOS_QUINCENA` *(v1.1)* | anio, mes, gasto_id, quincena | Log de pagos hechos (solo se guardan los pagados) |
| `OCASIONALES` *(v1.1)* | id, anio, mes, concepto, monto, quincena, pagado | Pagos no recurrentes por mes |

**Patrón save:** cada cambio reescribe la hoja completa con `saveSection(sheet, headers, rows)` — clear + set + actualiza CONFIG.updated. Para Gastos Fijos uso variantes silenciosas: `persistPagos()`, `persistOcasionales()`, `persistGastosFijos()` (sin toast "Guardado") porque cada toggle persiste y sería ruidoso.

**Seed:** la primera vez que una hoja está vacía, `autofillMissing()` la rellena con `DEFAULT` (objeto con datos iniciales hardcodeados). El botón **"♻️ Restaurar Excel"** del header borra todas las hojas y las repuebla con DEFAULT.

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

**Tipografía:** Outfit (300, 400, 500, 600, 700, 800).

**Patrones visuales:**
- Cards `border-radius:16px`, sombra suave `0 1px 4px rgba(0,0,0,.06)`, border `--border`
- KPIs en grid responsive con variantes `kc` (default), `ktotal` (gradiente navy), `kconsol` (gradiente cyan), `kpaid` (gradiente emerald, v1.1)
- Badges pill `b-blue|purple|green|amber|red|gray`
- Tabs con `border-bottom-color: cyan` activo
- Tablas `.tc` con thead bg `#f8fafc`, hover `#fafbff`, footer `tfoot` para totales
- Modales con `.ov` overlay (backdrop blur en v1.1) + `.m` card centrado, animación `slideUp`
- Toast bottom-right (`.toast-ok` navy, `.toast-warn` amber, `.toast-err` red)

**Componente clave de v1.1: `.exp-cell`** (toggle pagado en Gastos Fijos):
- Pendiente: fondo blanco + checkbox vacío gris + monto navy. Hover → border cyan + sombra elevada + checkbox cyan claro
- Pagado: fondo gris + ✓ verde redondo + monto tachado gris muted. Hover → fondo rojo claro + ↺ rojo (afford "deshacer")
- Animación `expPop` 260ms al togglear (scale 0.92 → 1)

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
- `renderDash()` — KPIs + donut + line chart + tabla todos los activos
- `renderFondos()`, `renderHistorial()`, `renderCDP()`, `renderPolizas()`, `renderBienes()`, `renderCuentas()`, `renderGastos()` *(v1.1)*

### Gastos Fijos *(v1.1)*
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
Patrón con `MOD_MAP` (clave → ID overlay) + `openMod(k)` que dispatcha al builder/preparer correspondiente.

## Versionado

| Versión | Fecha | Highlights |
|---------|-------|-----------|
| **v1.0** | (previo) | 7 pestañas: Dashboard, Fondos, Historial, CDP, Pólizas, Bienes, Cuentas · Auth Gmail · Sheets API · gráficos · export Excel · importer PDF de BN |
| **v1.1** | 28 abr 2026 noche | **Pestaña Gastos Fijos** — 19 gastos seed (Ahorro, Comida, Tarjeta, Mesada, Salario Alba, etc.) · KPIs total/Q1/Q2/% pagado · toggle pagado interactivo (✓ tachado / ↺ deshacer) · histórico mensual independiente por mes · pagos ocasionales con badge · modal editar (alta/baja/cambio del catálogo, aplica a todos los meses) · selector mes con badge EN CURSO/CERRADO/FUTURO · 3 hojas nuevas (GASTOS_FIJOS, PAGOS_QUINCENA, OCASIONALES) · solo CRC · commit `dcce5ab` |

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
- Cada toggle de Gastos Fijos hace 1 request a Sheets — si JC marca 10 cosas seguidas son 10 requests. Si se vuelve molesto, debounce.
- `data-tab="<seccion>"` en nav-tab → `id="tab-<seccion>"` en `.section`
- `MOD_MAP[clave] = 'ovOverlayId'` — si agregás modal, agregalo al MOD_MAP

⚠️ **Sync SKILL.md** si hay cambios estructurales — solo se mantiene esta copia (`~/.claude/skills/especialista-inversiones-J-A-S/SKILL.md`).

⚠️ **Deploy**: push a `main` → Netlify auto-deploy en ~30s. JC autoriza pushes directos a main para cambios validados (igual que en sus otros proyectos `trading` y `reclamos-sdi`).

## Pendientes / próximos pasos

### Validación pendiente con JC
- Probar pestaña **Gastos Fijos** en producción:
  - Primera carga debe crear las 3 hojas nuevas con seed (toast "Llenando secciones vacías…")
  - Ajustar montos reales del Sistema SAIS (default ¢160k/¢160k) en "Editar gastos fijos"
  - Marcar/desmarcar y validar que persiste tras recargar página
  - Navegar a meses pasados/futuros y validar que cada mes guarda su propio estado
  - Probar pago ocasional (alta + toggle + delete)

### Features futuras propuestas
- **Vista anual de Gastos Fijos** — tabla 12 meses × N gastos con totales por mes (similar al Excel original)
- **Recurrencia configurable** — gastos cada 2 meses, anuales, etc. (actualmente solo quincenal Q1/Q2)
- **Alertas de vencimiento** — días para pagar antes del 17 / 02 (push notification?)
- **Categorías de gastos** — agrupar por Ahorro, Servicios, Personal, Familia, etc. con totales por categoría
- **Comparativa mes vs promedio** — KPI "este mes vs promedio últimos 6"
- **Export Gastos Fijos a Excel** — actualmente el export ya cubre el resto, agregar las 3 hojas nuevas
- **Debounce en toggles** — si JC marca varios pagos seguidos, agrupar en un solo request a Sheets
- **Edit inline de monto en celda** — sin abrir modal completo
- **Modo móvil mejorado** — la pestaña actual es responsive pero la tabla puede mejorar en pantallas <380px
- **Dark mode** — toggle en header, persistir en localStorage

### Nice-to-have del proyecto general
- Onboarding inicial primera visita (tour de las 8 pestañas)
- Página "FAQ" sobre el funcionamiento
- Backup automático del Sheet a Drive cada N días
- Importar de PDF también para tarjetas de crédito (no solo fondos BN)

## Recursos relacionados

- Repo: https://github.com/jhernandez-vibecode/inversiones-J-A-S
- Mockup intermedio v1.1: `inversiones-J-A-S/mockup-gastos-fijos.html` (untracked, se puede borrar tras validación)
- OAuth Client en Google Cloud Console: `446215450096-...` (Authorized JS origins debe incluir el dominio Netlify de JC)
