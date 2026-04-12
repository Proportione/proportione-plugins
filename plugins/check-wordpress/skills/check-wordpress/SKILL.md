# Skill: Check WordPress

QA integral de sitios WordPress de Proportione. Orquesta el framework
`@wp-qa/shared` existente con Playwright MCP para navegacion y verificacion visual.

**Target:** $ARGUMENTS (URL del sitio, nombre del cliente, o "todos" para batch)

## Configuration

This skill references external paths. Set these environment variables or replace inline:

- `$WP_QA_DIR` — WordPress shared QA framework (e.g. `/path/to/Wordpress/_comun/qa`)
- `$QA_PROPORTIONE_DIR` — Root of the QA_Proportione repo (e.g. `/path/to/QA_Proportione`)

## Sitios WordPress de Proportione

| Sitio | URL | Cliente |
|-------|-----|---------|
| proportione.com | https://proportione.com | Proportione |
| porqueviven.org | https://porqueviven.org | Porque Viven |
| institutoteologia.org | https://institutoteologia.org | IITD |
| dev.institutoteologia.org | https://dev.institutoteologia.org | IITD (dev) |
| win2winevents.com | https://win2winevents.com | Win2Win |
| dev.win2winevents.com | https://dev.win2winevents.com | Win2Win (dev) |
| ta-cle.com | https://ta-cle.com | Tacle |

---

## Flujo

### PASO 1 — Determinar alcance

1. Que sitio(s) revisar? Si no se especifica, preguntar.
2. Revision completa o un aspecto concreto? (a11y, SEO, links, security, performance)
3. Si el usuario dice "todos", ejecutar en batch.

### PASO 2 — Navegacion con Playwright MCP

Usa Playwright MCP para abrir el sitio y navegar las paginas principales:

1. Homepage
2. Paginas principales del menu (About, Servicios, Contacto, etc.)
3. Al menos 1 entrada de blog si existe
4. Pagina 404 (navegar a `/url-que-no-existe`)

En cada pagina, recopilar:
- Errores de consola JavaScript
- Recursos que fallan (404, 500)
- Tiempo de carga percibido

### PASO 3 — Accesibilidad (axe-core)

El framework `@wp-qa/shared` ya tiene `accessibility.js` con axe-core.
Si es posible, ejecutar programaticamente:

```bash
cd $WP_QA_DIR
node -e "const qa = require('.'); /* usar funciones de accessibility */"
```

Complementar con verificacion manual via Playwright MCP:

- [ ] Todas las imagenes tienen alt text?
- [ ] Los enlaces tienen texto descriptivo (no "click aqui")?
- [ ] Los formularios tienen labels asociados?
- [ ] El contraste de colores es suficiente?
- [ ] Se puede navegar con teclado? (Tab order)
- [ ] Hay heading hierarchy correcta? (h1 -> h2 -> h3, sin saltos)

### PASO 4 — SEO

El framework tiene `seo-validator.js`. Verificar:

- [ ] Cada pagina tiene title unico y descriptivo (<60 chars)?
- [ ] Meta description existe y es unica (<160 chars)?
- [ ] Hay un solo h1 por pagina?
- [ ] Las URLs son limpias (no `?p=123`)?
- [ ] Existe sitemap.xml? Navegar a `/sitemap.xml`
- [ ] Existe robots.txt? Navegar a `/robots.txt`
- [ ] Canonical URLs correctas?
- [ ] Open Graph tags para redes sociales?

### PASO 5 — Enlaces e imagenes

Usando `link-checker.js` e `image-checker.js`:

- [ ] Hay enlaces rotos (404)?
- [ ] Hay enlaces a HTTP en sitio HTTPS (mixed content)?
- [ ] Las imagenes cargan correctamente?
- [ ] Las imagenes estan optimizadas? (no >500KB sin lazy load)
- [ ] Se usa lazy loading para imagenes below-the-fold?

### PASO 6 — Seguridad WordPress

Verificar via SSH si hay acceso, o via web:

```bash
# Si hay acceso SSH (SiteGround, etc.)
wp plugin list --ssh=[host]
wp core version --ssh=[host]
wp theme list --ssh=[host]
```

Checklist de seguridad:
- [ ] WordPress core actualizado?
- [ ] Todos los plugins actualizados?
- [ ] Temas no usados eliminados?
- [ ] Login URL cambiada? (WP Ghost Pro)
- [ ] XML-RPC desactivado?
- [ ] REST API restringida para usuarios no autenticados?
- [ ] Security Ninja Pro activo?
- [ ] Backup automatico configurado?
- [ ] SSL valido y no expirado?

### PASO 7 — Rendimiento

Usando `performance.js` y Lighthouse si esta disponible:

```bash
# Si lighthouse esta instalado
lighthouse [URL] --output json --output-path /tmp/lighthouse-[sitio].json \
  --chrome-flags="--headless --no-sandbox"
```

- [ ] LCP (Largest Contentful Paint) < 2.5s
- [ ] INP (Interaction to Next Paint) < 200ms
- [ ] CLS (Cumulative Layout Shift) < 0.1
- [ ] Hay render-blocking CSS/JS?
- [ ] Se usa cache de navegador?
- [ ] Se sirven imagenes en WebP/AVIF?

### PASO 8 — Estado de Elementor

La mayoria de sitios usan Elementor Pro:

- [ ] Elementor Pro actualizado? (comprobar version)
- [ ] Template conditions correctas? (header/footer en todas las paginas)
- [ ] No hay widgets huerfanos o secciones vacias?
- [ ] Global fonts/colors configurados (no inline)?

### PASO 9 — Generar informe

```markdown
## WordPress Health Check — [sitio] — [fecha]

### Resumen
[Semaforo: VERDE/AMARILLO/ROJO]
[1-2 frases con estado general]

### Puntuaciones
| Categoria | Estado | Detalles |
|-----------|--------|----------|
| Accesibilidad | OK/WARN/FAIL | [resumen] |
| SEO | OK/WARN/FAIL | [resumen] |
| Enlaces | OK/WARN/FAIL | [n] rotos de [total] |
| Imagenes | OK/WARN/FAIL | [n] sin alt, [n] sin optimizar |
| Seguridad | OK/WARN/FAIL | [resumen] |
| Rendimiento | OK/WARN/FAIL | LCP=[x]s, CLS=[x] |
| Plugins | OK/WARN/FAIL | [n] desactualizados |

### Acciones requeridas (priorizadas)
1. [CRITICAL] [accion]
2. [HIGH] [accion]
3. [MEDIUM] [accion]

### Lighthouse scores (si disponible)
- Performance: [x]/100
- Accessibility: [x]/100
- Best Practices: [x]/100
- SEO: [x]/100
```

---

## Modo batch

Si el usuario pide revisar todos los sitios:

1. Ejecutar PASO 2-8 para cada sitio
2. Generar un informe consolidado con tabla comparativa
3. Guardar en `$QA_PROPORTIONE_DIR/reports/{fecha}/wordpress/`

```markdown
## WordPress Fleet Health — [fecha]

| Sitio | A11y | SEO | Links | Security | Perf | Overall |
|-------|------|-----|-------|----------|------|---------|
| proportione.com | OK | WARN | OK | OK | WARN | AMARILLO |
| porqueviven.org | OK | OK | FAIL | OK | OK | AMARILLO |
| ... | | | | | | |
```

## Notas

- No modificar codigo del sitio sin aprobacion del usuario.
- Si un plugin tiene un CVE conocido, reportar como CRITICAL.
- Para sitios de desarrollo (dev.*), ser mas tolerante con SEO y rendimiento.
- El framework `@wp-qa/shared` se ejecuta con Node.js y requiere Playwright instalado.
