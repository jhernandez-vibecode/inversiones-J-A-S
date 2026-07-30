# Control de Inversiones — J-A-S

App web personal para el control del portafolio de inversiones familiar.

**En producción:** https://inversiones-j-a-s.netlify.app/ (auto-deploy desde `main`)

## Características

- 🔐 **Auth con Google** — whitelist de 2 cuentas (JC + Alba María)
- 📊 **Google Sheets como base de datos** — 10 hojas, la app las descubre en tu Drive
- 📄 **Importador de PDF** — lee los estados de cuenta mensuales del BN y carga el historial
- 🕑 **Saldo provisional** — anotás el saldo del banco a mitad de mes y el estado de cuenta lo confirma después
- 🏆 **Ranking de rendimiento** — qué fondo rinde mejor, por tasa o por ganancias, del mes o del año
- ⬇️ **Exportar a Excel** (.xlsx) con SheetJS
- 📈 **Gráficos** — distribución del portafolio y evolución mensual por fondo
- 🎨 Single-file HTML, sin build step

## Secciones

1. **Dashboard** — KPIs + distribución + evolución + tabla de todos los activos
2. **Fondos de Inversión** — Superfondo $, BN Internacional SUMA, Crecifondo CRC, Superfondo CRC
3. **Historial Mensual** — por fondo y por titular, con importador de PDF
4. **CDP** — Certificados de depósito a plazo
5. **Pólizas** — Vida INS + pensión BN Vital
6. **Bienes** — Inmuebles + vehículos
7. **Cuentas** — Bancarias BN
8. **Gastos Fijos** — Pagos por quincena, con histórico mensual y pagos ocasionales

## Deploy

### Netlify

1. Conectar el repo (`main` despliega solo, ~30s) o subir `index.html` a app.netlify.com/drop
2. En **Google Cloud Console** → OAuth Client `446215450096-...`:
   - Authorized JavaScript origins: `https://inversiones-j-a-s.netlify.app`
   - APIs habilitadas: **Google Sheets API** y **Google Drive API** (la de Drive se usa desde
     v1.5 para encontrar la hoja; si está apagada, el login no puede resolverla)

## Datos

Todos los datos viven en un Google Sheet llamado "Control de Inversiones JC", dentro del Drive
del usuario. El OAuth usa scope `drive.file`: la app solo ve los archivos que ella misma creó.

⚠️ **La hoja se descubre, no se inventa.** El `localStorage` guarda la dirección de la hoja,
pero es solo un **caché**: un limpiador de disco lo borra sin aviso. La fuente de la verdad es
Drive. Si el caché no está, la app le pregunta a Drive; si encuentra una sola hoja se reconecta
sola, si hay varias las muestra para elegir, y **nunca crea una hoja nueva sin confirmación
explícita**. Crear una a ciegas fue lo que en julio de 2026 dejó el portafolio real huérfano y
sembró una hoja de reemplazo con los datos de ejemplo del código.

El botón **🔗 Hoja** del header muestra a cuál hoja estás conectado y permite cambiarla.

## Stack

- HTML + Vanilla JS (single file, sin dependencias de build)
- Google Identity Services (GIS) — un solo flujo de auth: access token
- Google Sheets API v4 + Google Drive API (`files.list`)
- pdf.js 3.11 — parser de los estados de cuenta del BN
- Chart.js 4.4
- SheetJS 0.18
- Fuente: Outfit

---

Privado · Juan Carlos Hernández Vargas · 2026
