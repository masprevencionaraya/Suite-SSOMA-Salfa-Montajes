# Salfa · Suite de Prevención

Aplicaciones web independientes (sin backend, sin build step) para prevención de riesgos en terreno.

## Estructura

```
salfa-prevencion/
├── index.html              # Página de inicio, enlaza a ambas apps
├── fatiga/
│   └── index.html          # Turno·Seguro — control de fatiga y test de reacción
├── checklist/
│   └── index.html          # HSE Montajes — checklist pre-uso con firma digital
└── gestion-sitio/
    └── index.html          # Check List Gestión del Sitio (SSOMA) — orden y aseo
```

## Apps

### 🟡 Turno·Seguro — `/fatiga`
Autoevaluación de fatiga antes de operar maquinaria: horas de sueño, nivel de energía y un test de tiempo de reacción. Calcula un score de riesgo (apto / moderado / alto) y lleva un historial de turno en memoria (se reinicia al recargar la página).

### 🟠 HSE Montajes — `/checklist`
Checklist de inspección pre-uso de herramientas (RUT, área, 3 puntos de control) con firma digital en canvas. Guarda los registros en `localStorage` del navegador como cola offline, y simula sincronización a un backend cuando hay conexión.

### 🔴 Check List Gestión del Sitio — `/gestion-sitio`
Versión digital del formulario SSOMA "Check List Gestión del Sitio" (Rev. 0, 20/01/2021), con el mismo diseño de las listas RCO. Incluye obra/contrato, área, los 17 elementos de orden y aseo (SI / NO / N.A), responsable y fecha de ejecución obligatorios para cada "NO", porcentaje de cumplimiento, observaciones y firmas de **Realizó** y **Revisó**. Guarda en `localStorage` (clave `salfa_gestion_sitio_pending`) como cola offline.

**Nota:** las apps son prototipos front-end. El checklist no envía datos a ningún servidor real — el botón "Sincronizar" simula el envío y vacía la cola local. Para producción real, hay que reemplazar esa simulación por una llamada a una API.

## Cómo correrlas localmente

No requieren instalación ni dependencias — son HTML/CSS/JS puro (usan Tailwind y Google Fonts vía CDN, así que necesitas conexión a internet la primera vez que cargan).

```bash
# Opción 1: abrir directamente
open index.html

# Opción 2: servidor local simple
python3 -m http.server 8000
# luego abrir http://localhost:8000
```

## Deploy en GitHub Pages

1. Sube esta carpeta a un repositorio de GitHub.
2. Ve a **Settings → Pages**.
3. En "Source", selecciona la rama `main` y la carpeta `/ (root)`.
4. Guarda — GitHub te dará una URL tipo `https://tu-usuario.github.io/tu-repo/`.

## Stack

- HTML + Tailwind CSS (CDN) + CSS custom (variables, tipografía Barlow Condensed / Inter / IBM Plex Mono vía Google Fonts)
- JavaScript vanilla, sin frameworks
- Persistencia: `localStorage` (solo en el checklist)

## Pendientes para producción

- [ ] Reemplazar la simulación de sync por una API real (REST o similar)
- [ ] Autenticación de operadores/supervisores
- [ ] Notificación real a supervisor en caso de "Alto riesgo" en Turno·Seguro
- [ ] Persistir historial de Turno·Seguro (hoy se pierde al recargar)
- [ ] Compresión de la firma (el `dataURL` del canvas puede pesar bastante en `localStorage`)
