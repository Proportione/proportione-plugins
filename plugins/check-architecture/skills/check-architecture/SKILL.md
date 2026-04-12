# Skill: Check Architecture

Validacion de que el codigo respeta las convenciones y patrones arquitectonicos
definidos para cada proyecto de Proportione.

**Target:** $ARGUMENTS (proyecto, directorio, o PR/commit para verificar solo los cambios)

## Configuration

This skill references external paths. Set these environment variables or replace inline:

- `$QA_PROPORTIONE_DIR` — Root of the QA_Proportione repo (e.g. `/path/to/QA_Proportione`)

## Flujo

### PASO 1 — Cargar convenciones del proyecto

1. Lee el `CLAUDE.md` del proyecto para extraer:
   - Estructura de carpetas esperada
   - Convenciones de naming
   - Patrones de separacion de capas
   - Reglas de imports/dependencias
2. Lee `$QA_PROPORTIONE_DIR/CLAUDE.md` para contexto transversal.
3. Si `$ARGUMENTS` es un PR/commit, obten el diff para acotar el analisis.

### PASO 2 — Analisis por proyecto

Aplica las reglas especificas segun el proyecto detectado:

#### automation-brain (Python + Terraform)

Convenciones del CLAUDE.md:
- `clientes/{nombre}/` por cada cliente
- `comun/resources/` para base de conocimiento compartida
- `comun/tooling/` para herramientas comunes
- Nombres en `kebab-case` y minusculas
- Servicios: `clientes/*/proyectos/servicios/`
- Docs: `clientes/*/docs/`
- Tools: `clientes/*/tools/`
- Cada servicio con su `package.json`, `Dockerfile`, `deploy.sh`
- `.env` nunca commiteado (usar `.env.example`)

Verificar:
- [ ] Los ficheros nuevos estan en la carpeta correcta del cliente?
- [ ] El codigo compartido esta en `comun/` y no duplicado en clientes?
- [ ] Los nombres siguen kebab-case?
- [ ] Los servicios tienen los 3 ficheros obligatorios?
- [ ] Hay imports cruzados entre clientes? (no debe haberlos)
- [ ] Las herramientas reutilizables estan en `comun/tooling/`?

#### Aviaria / waze-del-aire (Python + React + Streamlit)

Estructura monorepo:
```
apps/
  platform-api/       # FastAPI backend
  military-dashboard/ # Streamlit dashboard
  frontend-pwa/       # React PWA
packages/
  core-logic/         # Logica de negocio (framework-agnostic)
```

Verificar:
- [ ] `core-logic` es independiente de framework? (sin imports de FastAPI, Streamlit o React)
- [ ] La logica de negocio esta en `core-logic`, no en las apps?
- [ ] Los endpoints de `platform-api` son thin wrappers que delegan a `core-logic`?
- [ ] `military-dashboard` no hace queries directas a la DB? (debe usar la API o core-logic)
- [ ] `frontend-pwa` consume solo la API REST, no accede a internals?
- [ ] Los modelos PostGIS estan en `core-logic`?

#### WordPress (PHP + Elementor)

Verificar por sitio:
- [ ] Se usa child theme, no se modifica el theme padre?
- [ ] Custom code esta en mu-plugins o child theme functions.php?
- [ ] No hay logica de negocio en templates de Elementor?
- [ ] Los snippets de WPCodeBox/Code Snippets estan documentados?
- [ ] Las custom post types se registran en un plugin propio, no en el theme?

#### IaC Aveiro (Terraform + Ansible)

Estructura esperada:
```
terraform/
  main.tf, providers.tf, variables.tf
  modules/             # Modulos reutilizables
ansible/
  site.yml
  inventory/hosts.yml
  roles/               # Un rol por servicio
```

Verificar:
- [ ] Los recursos de Terraform estan modularizados? (no todo en main.tf)
- [ ] Las variables sensibles usan `sensitive = true`?
- [ ] Cada rol de Ansible tiene `tasks/main.yml`, `defaults/main.yml`?
- [ ] El inventario no tiene passwords en claro?
- [ ] Los VLANs coinciden con el diseno? (10=servers, 20=dev, 50=backup, 99=WireGuard)

#### porqueviven-app (Nuxt + Firebase)

Verificar:
- [ ] Los composables estan en `composables/`?
- [ ] Los stores estan en `stores/`?
- [ ] Las pages no contienen logica de negocio compleja?
- [ ] Los componentes son reutilizables o estan acoplados a una page?
- [ ] Firebase config no esta hardcodeada?

### PASO 3 — Dependencias y acoplamiento

Para cualquier proyecto, verificar:

```bash
# Python: buscar imports circulares
grep -rn "^from\|^import" [directorio] | sort | uniq -c | sort -rn | head -20

# JavaScript: buscar imports entre capas que no deberian cruzarse
grep -rn "from '.*\.\./\.\." [directorio] --include="*.js" --include="*.ts" --include="*.tsx"
```

- [ ] Hay dependencias circulares entre modulos?
- [ ] Hay imports que cruzan capas? (UI importando DB, API importando UI)
- [ ] El grafo de dependencias fluye en una direccion? (UI -> API -> Core -> DB)
- [ ] Hay modulos "god object" que importan de todas partes?

### PASO 4 — Naming y organizacion

- [ ] Los ficheros siguen la convencion del proyecto? (kebab-case, camelCase, PascalCase segun stack)
- [ ] Los directorios tienen un proposito claro? (no carpetas genericas como `utils/`, `misc/`, `stuff/`)
- [ ] Hay ficheros en la raiz que deberian estar en subdirectorios?
- [ ] Los ficheros de configuracion estan donde se espera? (.env.example, Dockerfile, etc.)

### PASO 5 — Generar informe

```markdown
## Architecture Review — [proyecto] — [fecha]

### Resumen
[Nivel de adherencia: ALTA/MEDIA/BAJA]
[1-2 frases sobre el estado arquitectonico]

### Convenciones verificadas
| Regla | Estado | Detalle |
|-------|--------|---------|
| Estructura de carpetas | OK/DRIFT | [que viola] |
| Separacion de capas | OK/DRIFT | [imports cruzados] |
| Naming conventions | OK/DRIFT | [ficheros que no cumplen] |
| Dependencias | OK/CIRCULAR | [ciclos encontrados] |

### Drift detectado
1. **[fichero]** — [descripcion del drift]
   - Deberia estar en: [ubicacion correcta]
   - O deberia seguir: [patron correcto]

### Recomendaciones
1. [Refactor mas urgente]
2. [Siguiente paso]
```

---

## Notas

- El drift arquitectonico es normal y gradual. No alarmar por detalles menores.
- Priorizar violaciones que afectan mantenibilidad (imports cruzados, logica en capa incorrecta) sobre naming.
- Si el proyecto no tiene CLAUDE.md, sugerir crearlo con las convenciones observadas.
- Para proyectos pequenos (OpenClaw, Hispanidad), ser pragmatico: no exigir arquitectura enterprise.
