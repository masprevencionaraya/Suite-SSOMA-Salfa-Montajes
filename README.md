# Salfa · Suite de Prevención

Cuatro aplicaciones web independientes (sin backend, sin build step) para prevención de riesgos en terreno.

## Estructura

```
salfa-prevencion/
├── index.html              # Página de inicio, enlaza a las cuatro apps
├── fatiga/
│   └── index.html          # Turno·Seguro — control de fatiga y test de reacción
├── checklist/
│   ├── index.html          # HSE Montajes — checklist pre-uso con firma digital
│   └── vendor/              # jsPDF + jspdf-autotable (vendorizados, sin CDN)
├── calor/
│   ├── index.html          # Bitácora de exposición ocupacional a calor (D.S. 594)
│   ├── vendor/              # jsPDF + jspdf-autotable (vendorizados, sin CDN)
│   └── apps-script/
│       └── Code.gs          # Puente Apps Script: recibe el PDF y lo guarda en Drive (compartido por las 3 apps con subida)
└── energia-electrica/
    ├── index.html          # RCO N°8 — Contacto con Energía Eléctrica
    └── vendor/              # jsPDF + jspdf-autotable (vendorizados, sin CDN)
```

## Apps

### 🟡 Turno·Seguro — `/fatiga`
Autoevaluación de fatiga antes de operar maquinaria: horas de sueño, nivel de energía y un test de tiempo de reacción. Calcula un score de riesgo (apto / moderado / alto) y lleva un historial de turno en memoria (se reinicia al recargar la página).

### 🟠 HSE Montajes — `/checklist`
Checklist de inspección pre-uso de herramientas (RUT, área, 3 puntos de control) con firma digital en canvas. Guarda los registros en `localStorage` del navegador como cola offline, y sube automáticamente el informe en PDF a Google Drive (mismo puente que `calor`, ver más abajo).

### 🔴 Exposición a calor — `/calor`
Bitácora técnica paso a paso para la evaluación de exposición ocupacional a calor según los Arts. 96 a 98 bis del D.S. N°594:

1. **Tramos de trabajo y descanso** — se registra cada área/actividad (situación ambiental, TBH, TG, TBS según corresponda, y tiempo de permanencia).
2. **Resultados ponderados** — calcula automáticamente el TGBH ponderado en el tiempo (Art. 97), el costo energético ponderado M (Art. 98), determina la categoría de carga de trabajo (liviana/moderada/pesada) y el régimen trabajo-descanso real, y compara el TGBH contra la tabla de valores límites permisibles para emitir un veredicto CUMPLE / NO CUMPLE.
3. **Gestión de altas temperaturas (Art. 98 bis)** — checklist de las acciones exigidas frente a alertas de la Dirección Meteorológica de Chile y SENAPRED (seguimiento diario, identificación del peligro, plan de gestión y otras disposiciones del Minsal).
4. **Firma digital** y guardado en `localStorage` como cola offline, igual que el checklist HSE.

Incluye además tablas de referencia colapsables (Costo Energético según Tipo de Trabajo y Valores Límites Permisibles del Índice TGBH) para consultar mientras se completa la bitácora, un botón para descargar cada bitácora como informe en PDF (vía el diálogo de impresión del navegador), y subida automática del PDF a una carpeta de Google Drive — ver [Subida automática a Google Drive](#☁️-subida-automática-a-google-drive) más abajo.

### 🟣 RCO N°8 · Energía Eléctrica — `/energia-electrica`
Cartilla de verificación de controles críticos operacionales extraída de `SSOMA-ME-RCO-08-CV-08` (versión ENE.2020), con sus tres secciones originales:

1. **Requisitos de las personas** — capacitación y certificación según nivel de tensión.
2. **Requisitos de la organización** — procedimientos de aislación/bloqueo, identificación de fuentes de energía, verificación de ausencia de energía, etc.
3. **Requisitos a los equipos e instalaciones** — EPP dieléctrico, herramientas aisladas, puesta a tierra, kit de rescate en sala eléctrica.

Cada ítem se responde SI/NO/N-A y admite **Comentario**, **Evidencia** y un **adjunto opcional** (foto o documento — ver más abajo) por separado, tal como en la cartilla original. Misma firma digital (con opción de indicar otro nombre/cargo de quien ejecutó la actividad), cola offline en `localStorage`, descarga de informe en PDF y subida automática a Google Drive que el resto de la suite.

**Adjuntos de evidencia:** cada ítem admite adjuntar una foto o documento (PDF, Word, Excel). Las fotos se comprimen en el navegador (máx. 1200px, JPEG calidad 0.72) antes de guardarse; el resto de formatos se guarda tal cual, con un tope de 4 MB por archivo para no saturar el almacenamiento local. El adjunto aparece embebido (miniatura o nombre de archivo) tanto en la vista imprimible como en el informe PDF que se sube a Drive.

**Nota:** `fatiga` es prototipo front-end puro — no envía datos a ningún servidor. `checklist`, `calor` y `energia-electrica` sí suben de verdad su informe PDF a Drive una vez configurado el puente de Apps Script (ver más abajo, es el mismo para las tres); mientras no esté configurado, el botón "Sincronizar" se comporta como una cola local simulada.

## ☁️ Subida automática a Google Drive

Como estas apps no tienen backend propio, `checklist`, `calor` y `energia-electrica` comparten un mismo script de Google Apps Script como puente: cada app genera su informe en PDF en el navegador (con [jsPDF](https://github.com/parallax/jsPDF) + `jspdf-autotable`, incluidos localmente en el `vendor/` de cada una, sin depender de ningún CDN) y lo envía por `fetch` a ese script, que lo guarda en la carpeta de Drive indicada — con el nombre de la app en el prefijo del archivo para distinguirlos.

**Configuración (una sola vez, sirve para las tres apps):**

1. Abre `calor/apps-script/Code.gs` en este repo — trae instrucciones paso a paso en los comentarios.
2. Despliega ese código como Aplicación Web en [script.google.com](https://script.google.com/) bajo tu propia cuenta de Google (la que tiene acceso a la carpeta de Drive destino).
3. Copia la URL resultante (termina en `/exec`) y pégala en la constante `DRIVE_UPLOAD_URL` dentro de `calor/index.html`, `checklist/index.html` **y** `energia-electrica/index.html`.
4. Verifica que `DRIVE_SHARED_SECRET` coincida exactamente entre las tres apps y `SHARED_SECRET` en `Code.gs`.

**Cómo funciona en cada app:** al guardar un registro, intenta subirlo a Drive de inmediato si hay conexión; si falla o está offline, queda en la cola local ("Registros en este dispositivo") y el botón "Sincronizar" reintenta la subida de todo lo pendiente.

**Nota de seguridad:** al ser apps 100% estáticas, la URL del script y el "secreto compartido" quedan visibles en el código fuente que llega al navegador de cualquiera que las use. El secreto solo filtra accesos casuales/automatizados a la carpeta, no es una autenticación real — razonable para un formulario interno de baja sensibilidad, pero no lo uses para nada que requiera control de acceso real.

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
- Persistencia: `localStorage` (checklist, calor y energía eléctrica)
- Generación de PDF: [jsPDF](https://github.com/parallax/jsPDF) + `jspdf-autotable`, vendorizados en el `vendor/` de `calor`, `checklist` y `energia-electrica`
- Puente a Google Drive: Google Apps Script (`calor/apps-script/Code.gs`, compartido por `calor`, `checklist` y `energia-electrica`)

## Pendientes para producción

- [ ] Reemplazar la simulación de sync de Turno·Seguro por una API real (REST o similar)
- [ ] Autenticación de operadores/supervisores
- [ ] Notificación real a supervisor en caso de "Alto riesgo" en Turno·Seguro
- [ ] Persistir historial de Turno·Seguro (hoy se pierde al recargar)
- [ ] Compresión de la firma (el `dataURL` del canvas puede pesar bastante en `localStorage`)
- [ ] Reemplazar el "secreto compartido" del puente de Drive por autenticación real si la sensibilidad de los datos lo justifica
