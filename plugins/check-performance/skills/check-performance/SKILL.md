# Skill: Check Performance

Analisis de rendimiento adaptado al stack multi-tecnologia de Proportione.
Cubre backend (Python/PHP), frontend (React/Nuxt), web (Lighthouse/Web Vitals)
e infraestructura (GCP, Terraform).

**Target:** $ARGUMENTS (URL, proyecto, directorio, o ficheros especificos)

## Configuration

This skill references external paths. Set these environment variables or replace inline:

- `$WP_QA_DIR` — WordPress shared QA framework (e.g. `/path/to/Wordpress/_comun/qa`)

## Flujo

### PASO 1 — Identificar tipo de analisis

Segun `$ARGUMENTS`, determinar:
1. **Web app en produccion** -> Web Vitals + Lighthouse
2. **Backend Python** -> queries, I/O, connection pooling
3. **Frontend React/Nuxt** -> bundle, renders, lazy loading
4. **WordPress** -> page load + plugins + hosting
5. **Infraestructura GCP** -> sizing, costes por uso
6. **Codigo especifico** -> complejidad algoritmica, I/O

### PASO 2 — Analisis de Web Vitals (para URLs)

Usar el framework `@wp-qa/shared` que ya tiene `performance.js`:

```bash
# Ejecutar medicion programatica
cd $WP_QA_DIR
node -e "
const { measureWebVitals, checkPerformanceBudget } = require('./lib/performance');
// Requiere Playwright page object — usar via Playwright MCP
"
```

O medir manualmente via Playwright MCP:

1. Navegar a la URL con Playwright MCP
2. Ejecutar `performance.getEntriesByType('navigation')` para TTFB
3. Observar LCP, CLS via PerformanceObserver

**Presupuestos por tipo de sitio:**

| Metrica | WordPress | Aviaria PWA | porqueviven |
|---------|-----------|-------------|-------------|
| LCP | < 2.5s | < 2.0s | < 2.5s |
| INP | < 200ms | < 150ms | < 200ms |
| CLS | < 0.1 | < 0.05 | < 0.1 |
| TTFB | < 800ms | < 400ms | < 600ms |

Si Lighthouse CLI esta disponible:

```bash
lighthouse $URL --output json --output-path /tmp/lighthouse-report.json \
  --chrome-flags="--headless --no-sandbox" \
  --only-categories=performance
```

### PASO 3 — Backend Python (Aviaria, automation-brain)

#### Queries N+1
Buscar el patron: loop que hace una query por iteracion.

```python
# PATRON N+1 — buscar esto:
for item in items:
    related = db.query(Related).filter_by(item_id=item.id).first()  # N+1!

# FIX: eager loading o query batch
related_map = {r.item_id: r for r in db.query(Related).filter(Related.item_id.in_(ids))}
```

Verificar:
- [ ] Hay queries dentro de loops? (grep `for.*:` seguido de `.query(` o `execute(`)
- [ ] Se usa eager loading / joinedload en SQLAlchemy?
- [ ] Hay `SELECT *` donde solo se necesitan 2-3 columnas?
- [ ] Las queries PostGIS usan indices espaciales? (`CREATE INDEX ... USING GIST`)

#### I/O y concurrencia
- [ ] Hay I/O sincrono en endpoints async de FastAPI?
- [ ] Se usa `await` correctamente? (no bloquear el event loop)
- [ ] Los file reads usan `with open()` y cierran correctamente?
- [ ] Hay connection pooling para la DB? (Aviaria ya lo tiene — verificar config)

#### Caching
- [ ] Se cachean respuestas que no cambian frecuentemente?
- [ ] Los calculos costosos se memoizan?
- [ ] Se usan ETags o Cache-Control headers?

### PASO 4 — Frontend React/Nuxt

#### Bundle size
```bash
# React (Aviaria PWA)
cd apps/frontend-pwa
npm run build 2>&1 | tail -20  # Ver tamanos de chunks

# Nuxt (porqueviven)
npx nuxi analyze  # Genera reporte de bundle
```

Verificar:
- [ ] Hay imports de librerias enteras donde se usa 1 funcion? (`import _ from 'lodash'` vs `import { debounce } from 'lodash'`)
- [ ] Se usa `React.lazy()` / `defineAsyncComponent()` para rutas pesadas?
- [ ] Las imagenes se importan optimizadas? (WebP, srcset, lazy loading)
- [ ] Hay dependencias duplicadas en el bundle?

#### Renders innecesarios (React)
- [ ] Componentes que reciben props complejas usan `React.memo()`?
- [ ] Se crean objetos/arrays nuevos en cada render? (causa re-renders de hijos)
- [ ] `useEffect` tiene dependencias correctas? (ni de mas ni de menos)
- [ ] Se usa `useMemo`/`useCallback` donde el calculo es costoso?

#### Renders innecesarios (Nuxt/Vue)
- [ ] Los computed properties estan bien definidos? (no recalculan innecesariamente)
- [ ] Se usa `v-once` para contenido estatico?
- [ ] Los watchers tienen `{ deep: true }` solo cuando es necesario?

### PASO 5 — WordPress

Ademas de Web Vitals (PASO 2):

- [ ] Cuantos plugins activos? (>20 es senal de alerta)
- [ ] Hay plugins de cache activos? (WP Super Cache, W3 Total Cache, LiteSpeed)
- [ ] Se usa CDN? (Cloudflare, SiteGround CDN)
- [ ] Las imagenes estan en formato WebP?
- [ ] Hay render-blocking CSS/JS de plugins?
- [ ] Elementor carga assets de widgets no usados?
- [ ] Se usa Object Cache? (Redis, Memcached)
- [ ] PHP version es 8.x?
- [ ] La base de datos tiene overhead? (`wp db optimize --ssh=[host]`)

### PASO 6 — Infraestructura GCP

```bash
# Cloud Run: verificar CPU/memory allocation vs usage
gcloud run services describe [service] --region=[region] --format=json | \
  jq '.spec.template.spec.containers[0].resources'

# Cloud SQL: verificar tier vs usage
gcloud sql instances describe [instance] --format="value(settings.tier)"
```

- [ ] Los Cloud Run services tienen min-instances=0? (cold starts vs coste)
- [ ] Cloud SQL tier es adecuado? (db-f1-micro para dev, db-custom para prod)
- [ ] Se usa Cloud CDN o caching en Cloud Run?
- [ ] Hay VMs encendidas que deberian estar apagadas?

### PASO 7 — Generar informe

```markdown
## Performance Review — [proyecto/URL] — [fecha]

### Resumen
[Semaforo: RAPIDO / ACEPTABLE / LENTO]
[1-2 frases con el principal cuello de botella]

### Web Vitals (si aplica)
| Metrica | Valor | Presupuesto | Estado |
|---------|-------|-------------|--------|
| LCP | Xs | <2.5s | OK/FAIL |
| INP | Xms | <200ms | OK/FAIL |
| CLS | X | <0.1 | OK/FAIL |
| TTFB | Xms | <800ms | OK/FAIL |

### Lighthouse (si aplica)
- Performance: X/100
- Oportunidades: [lista de recomendaciones de Lighthouse]

### Hallazgos de codigo
1. **[fichero:linea]** — [problema de rendimiento]
   - Impacto estimado: [alto/medio/bajo]
   - Fix: [codigo o descripcion]

### Recomendaciones priorizadas
1. [Quick win — mayor impacto con menor esfuerzo]
2. [Siguiente]
3. [Siguiente]

### Coste de infraestructura (si aplica)
- Actual: X EUR/mes
- Optimizado: X EUR/mes
- Ahorro: X EUR/mes
```

---

## Notas

- Medir antes de optimizar. No optimizar por intuicion.
- Los Web Vitals de lab (Lighthouse) difieren de campo (CrUX). Priorizar datos reales si existen.
- Para WordPress en SiteGround, muchas optimizaciones vienen del hosting (SG Optimizer). Verificar que esta activo.
- Aviaria ya tiene connection pooling y lazy imports (verificado 2026-03). No proponer cambios redundantes.
- El framework `performance.js` de `@wp-qa/shared` ya mide LCP, CLS, TTFB. Reutilizarlo.
