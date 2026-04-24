# Control de Inversiones — J-A-S

App web personal para control del portafolio de inversiones familiar.

## Características

- 🔐 **Auth con Gmail** (whitelist: JC + Alba María)
- 📊 **Google Sheets como base de datos** (creado automáticamente en primer login)
- ⬇️ **Exportar a Excel** (.xlsx) con SheetJS
- 📈 **Charts interactivos** (distribución, histórico mensual)
- 🎨 **UI moderna** single-file HTML

## Secciones

1. **Dashboard** — KPIs + distribución + crecimiento
2. **Fondos de Inversión** — Superfondo $, BN Internacional, Crecifondo CRC, Superfondo CRC
3. **Historial Mensual** — 2025 y 2026 por fondo
4. **CDP** — Certificados de depósito a plazo
5. **Pólizas** — Vida INS + pensiones BN Vital
6. **Bienes** — Inmuebles + vehículos
7. **Cuentas** — Bancarias BN

## Deploy

### Netlify
1. Conectar el repo o subir `index.html` (drag & drop en app.netlify.com/drop)
2. En **Google Cloud Console** → OAuth Client `446215450096-...`:
   - Authorized JavaScript origins: agregar `https://TU-APP.netlify.app`

## Datos

Todos los datos viven en un Google Sheet llamado "Control de Inversiones JC" que se crea automáticamente en el primer login y se vincula al navegador vía `localStorage`.

El OAuth usa scope `drive.file` — solo archivos creados por la app son accesibles.

## Stack

- HTML + Vanilla JS (single file)
- Google Identity Services (GIS)
- Google Sheets API v4
- Chart.js 4.4
- SheetJS 0.18
- Fuente: Outfit

---

Privado · Juan Carlos Hernández Vargas · 2026
