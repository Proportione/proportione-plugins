# Skill: Check WP Visual

Regresion visual de sitios WordPress de Proportione usando Playwright MCP.
Captura screenshots de paginas clave y compara contra baselines.

**Target:** $ARGUMENTS (URL del sitio, nombre del cliente, "todos" para batch, o "baseline" para capturar nuevas baselines)

## Configuration

This skill references external paths. Set these environment variables or replace inline:

- `$QA_PROPORTIONE_DIR` — Root of the QA_Proportione repo (e.g. `/path/to/QA_Proportione`)

## Directorios

- **Baselines:** `$QA_PROPORTIONE_DIR/baselines/wordpress/{dominio}/`
- **Reports:** `$QA_PROPORTIONE_DIR/reports/{fecha}/visual/`

## Sitios y paginas a capturar

Para cada sitio, capturar estas paginas (adaptar segun menu del sitio):

| Pagina | Path | Prioridad |
|--------|------|-----------|
| Homepage | `/` | ALTA |
| About/Quienes somos | `/about/` o equivalente | ALTA |
| Servicios | `/servicios/` o equivalente | ALTA |
| Contacto | `/contacto/` o equivalente | MEDIA |
| Blog (listado) | `/blog/` | MEDIA |
| Blog (post) | primer post | BAJA |
| 404 | `/url-inexistente-qa-test` | BAJA |

## Flujo

### PASO 1 — Determinar modo

Segun `$ARGUMENTS`:
- **`baseline`** o **`capturar baseline`** -> PASO 2 (capturar y guardar como referencia)
- **URL o nombre de sitio** -> PASO 3 (capturar y comparar contra baseline existente)
- **`todos`** -> PASO 3 en batch para todos los sitios

### PASO 2 — Capturar baselines (modo baseline)

Para cada pagina del sitio:

1. Usar Playwright MCP para navegar a la URL
2. Esperar a que la pagina cargue completamente:
   - Esperar `networkidle` o al menos 3 segundos
   - Scroll hasta el final para trigger lazy loading
   - Scroll de vuelta arriba
3. Capturar screenshot a **2 viewports**:
   - Desktop: 1280x800
   - Mobile: 375x812 (iPhone 14)
4. Guardar screenshots con naming consistente:
   ```
   baselines/wordpress/{dominio}/
   ├── homepage-desktop.png
   ├── homepage-mobile.png
   ├── about-desktop.png
   ├── about-mobile.png
   ├── servicios-desktop.png
   ├── servicios-mobile.png
   ├── contacto-desktop.png
   ├── contacto-mobile.png
   └── metadata.json          # Fecha captura, URLs, viewports
   ```
5. Crear `metadata.json`:
   ```json
   {
     "site": "proportione.com",
     "captured": "2026-03-30T19:00:00Z",
     "viewports": {
       "desktop": {"width": 1280, "height": 800},
       "mobile": {"width": 375, "height": 812}
     },
     "pages": [
       {"name": "homepage", "url": "https://proportione.com/", "status": 200},
       {"name": "about", "url": "https://proportione.com/about/", "status": 200}
     ]
   }
   ```

Informar al usuario:

> Baselines capturadas para {sitio}: {n} screenshots ({n} desktop + {n} mobile).
> Proximas ejecuciones de /check-wp-visual compararan contra estas baselines.

### PASO 3 — Comparar contra baselines

1. Verificar que existen baselines en `baselines/wordpress/{dominio}/`
   - Si no existen: informar y ofrecer capturarlas primero (PASO 2)

2. Capturar screenshots actuales (mismo proceso que PASO 2) en directorio temporal:
   ```
   /tmp/qa-visual-{dominio}-{timestamp}/
   ```

3. Para cada par (baseline vs actual), comparar:

   **Metodo 1 — Comparacion visual con Claude (recomendado):**
   - Leer ambas imagenes (baseline y actual) con la herramienta Read
   - Claude compara visualmente y describe diferencias
   - Clasificar cambios:
     - **REGRESSION**: algo se rompio (layout, imagenes faltantes, texto cortado)
     - **CHANGE**: cambio intencional (nuevo contenido, redesign)
     - **MINOR**: diferencia insignificante (fecha actualizada, contador de visitas)

   **Metodo 2 — pixelmatch (cuando este instalado):**
   ```bash
   # Requiere: npm install -g pixelmatch-cli
   pixelmatch baseline.png actual.png diff.png --threshold 0.1
   ```

4. Para cada diferencia detectada, documentar:
   - Que pagina y viewport
   - Que cambio (descripcion visual)
   - Screenshot de la diferencia
   - Clasificacion: REGRESSION / CHANGE / MINOR

### PASO 4 — Generar informe

```markdown
## Visual Regression — {sitio} — {fecha}

### Resumen
[VERDE: sin regresiones / AMARILLO: cambios detectados / ROJO: regresiones]

### Comparacion
| Pagina | Viewport | Estado | Detalle |
|--------|----------|--------|---------|
| Homepage | Desktop | OK | Sin cambios |
| Homepage | Mobile | CHANGE | Nuevo banner promocional |
| Servicios | Desktop | REGRESSION | Imagen hero no carga |
| Servicios | Mobile | REGRESSION | Layout roto en ancho < 400px |

### Regresiones detectadas
1. **servicios-desktop.png** — Imagen hero no carga
   - Baseline: [imagen visible]
   - Actual: [placeholder o roto]
   - Posible causa: URL de imagen cambiada o CDN down

### Cambios intencionados (verificar con el equipo)
1. **homepage-mobile.png** — Nuevo banner superior
   - Actualizar baseline? (ejecutar `/check-wp-visual baseline {sitio}`)

### Baselines info
- Capturadas: {fecha baseline}
- Antiguedad: {n} dias
```

### PASO 5 — Actualizar baselines (si se solicita)

Si el usuario confirma que los cambios son intencionados:

1. Copiar screenshots actuales como nuevas baselines
2. Actualizar `metadata.json` con nueva fecha
3. Informar: "Baselines actualizadas para {sitio}. {n} screenshots reemplazadas."

---

## Modo batch

Si `$ARGUMENTS` es "todos":

1. Ejecutar PASO 3 para cada sitio con baselines
2. Generar informe consolidado:

```markdown
## Visual Fleet Check — {fecha}

| Sitio | Pages | OK | Changes | Regressions |
|-------|-------|----|---------|-------------|
| proportione.com | 8 | 7 | 1 | 0 |
| porqueviven.org | 6 | 4 | 1 | 1 |
| institutoteologia.org | 8 | 8 | 0 | 0 |
```

3. Guardar en `$QA_PROPORTIONE_DIR/reports/{fecha}/visual/fleet-report.md`

---

## Notas

- Los screenshots pueden variar ligeramente por: contenido dinamico (fechas, contadores), ads, cookies banners, A/B tests. Usar threshold del 5% antes de reportar como regresion.
- Siempre cerrar cookies/GDPR banners antes de capturar (click en "Aceptar" si aparece).
- Para sitios con login, capturar solo las paginas publicas a menos que se indique lo contrario.
- Las baselines deben actualizarse despues de cada cambio de diseno intencionado.
- Si las baselines tienen mas de 30 dias, sugerir recapturarlas.
- Playwright MCP ya esta configurado en el entorno — no requiere instalacion adicional.
