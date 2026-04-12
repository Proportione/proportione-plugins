# Skill: Check Security

Auditoria de seguridad adaptada al stack multi-tecnologia de Proportione.
Combina revision manual de Claude con herramientas automatizadas (Semgrep, Checkov).

**Target:** $ARGUMENTS (proyecto, directorio, o ficheros especificos)

## Configuration

This skill references external paths. Set these environment variables or replace inline:

- `$QA_PROPORTIONE_DIR` — Root of the QA_Proportione repo (e.g. `/path/to/QA_Proportione`)

## Dependencias

```bash
# Verificar antes de empezar
which semgrep     # pip install semgrep (--break-system-packages en este Mac)
which checkov     # pip install checkov (solo para Terraform/Ansible)
```

Si no estan instaladas, informar al usuario y ofrecer instalarlas.

---

## Flujo

### PASO 1 — Identificar alcance

Pregunta o infiere:
1. Que proyecto? (automaticamente si se ejecuta dentro de un repo)
2. Auditoria completa o de un cambio especifico (PR/commit)?
3. Stack del proyecto? (leer CLAUDE.md)

### PASO 2 — Analisis automatizado con Semgrep

Ejecuta Semgrep con las reglas custom de Proportione + reglas de la comunidad:

```bash
# Reglas custom de Proportione
semgrep --config $QA_PROPORTIONE_DIR/rulesets/semgrep/ \
  --json --output /tmp/semgrep-results.json \
  [directorio-del-proyecto]

# Reglas de la comunidad (las mas relevantes)
semgrep --config "p/python" --config "p/owasp-top-ten" \
  --json --output /tmp/semgrep-community.json \
  [directorio-del-proyecto]
```

Para Terraform/Ansible, anadir Checkov:

```bash
checkov -d [directorio-terraform] --output json > /tmp/checkov-results.json
```

### PASO 3 — Analisis manual por categoria OWASP

Revisa el codigo buscando las 10 categorias principales:

#### A01 — Broken Access Control
- [ ] Se verifican permisos antes de cada accion sensible?
- [ ] Hay endpoints/rutas sin autenticacion que deberian tenerla?
- [ ] Se puede escalar privilegios? (IDOR, path traversal)
- WordPress: se usa `current_user_can()` antes de acciones admin?
- GCP: los service accounts tienen solo los permisos necesarios (least privilege)?

#### A02 — Cryptographic Failures
- [ ] Hay secrets en codigo fuente o ficheros committeados?
- [ ] Se transmiten datos sensibles sin cifrar?
- [ ] Se usan algoritmos obsoletos (MD5, SHA1 para passwords)?
- [ ] Los .env estan en .gitignore?

#### A03 — Injection
- [ ] SQL: se usa parametrizacion? (`$wpdb->prepare()`, SQLAlchemy params, `$1` en PostGIS)
- [ ] Command: hay `os.system()`, `subprocess` con `shell=True`, `exec()` con input del usuario?
- [ ] XSS: se escapan outputs en HTML? (`esc_html()` en WP, `{variable}` en React ya escapa)
- [ ] Template: hay `|safe` o `dangerouslySetInnerHTML` sin sanitizar?

#### A04 — Insecure Design
- [ ] Los flujos criticos tienen rate limiting?
- [ ] Hay validacion del lado servidor (no solo cliente)?
- [ ] Los formularios tienen CSRF protection? (nonces en WP)

#### A05 — Security Misconfiguration
- [ ] Headers de seguridad configurados? (CSP, X-Frame-Options, HSTS)
- [ ] Debug mode desactivado en produccion?
- [ ] Directory listing desactivado?
- [ ] WordPress: XML-RPC desactivado? REST API restringida?
- [ ] GCP: buckets privados por defecto?

#### A06 — Vulnerable Components
- [ ] Hay dependencias con CVEs conocidas?
```bash
# Python
pip audit

# Node.js
npm audit

# WordPress
wp plugin list --update=available --ssh=[host]
```

#### A07 — Authentication Failures
- [ ] Passwords almacenados con bcrypt/argon2?
- [ ] Tokens con expiracion razonable?
- [ ] Se invalidan sesiones al cambiar password?

#### A08 — Data Integrity Failures
- [ ] Se verifican firmas/checksums de datos externos?
- [ ] Las pipelines CI/CD estan protegidas?
- [ ] Se usa pinning de dependencias?

#### A09 — Logging Failures
- [ ] Se registran eventos de seguridad? (login fallido, acceso denegado)
- [ ] Los logs NO contienen datos sensibles? (passwords, tokens, PII)

#### A10 — SSRF
- [ ] Se validan URLs proporcionadas por usuarios antes de hacer fetch?
- [ ] Hay restricciones de red para requests internos?

### PASO 4 — Analisis de configuracion

#### WordPress
```bash
# Via SSH al servidor
wp config get --ssh=[host]
wp option get siteurl --ssh=[host]
# Verificar WP_DEBUG=false, DISALLOW_FILE_EDIT=true
```

#### GCP (automation-brain)
```bash
# Service accounts y sus permisos
gcloud iam service-accounts list --project=automation-brain
# Buckets publicos
gsutil ls -p automation-brain | xargs -I{} gsutil iam get {}
```

#### Terraform
```bash
checkov -d [terraform-dir] --framework terraform
```

### PASO 5 — Generar informe

```markdown
## Security Audit — [proyecto] — [fecha]

### Resumen ejecutivo
[2-3 frases con nivel de riesgo general: ALTO/MEDIO/BAJO]

### Herramientas ejecutadas
- Semgrep: [n] reglas, [n] hallazgos
- Checkov: [n] checks, [n] failed (si aplica)
- pip audit / npm audit: [n] vulnerabilidades

### Hallazgos

#### CRITICAL (riesgo alto, corregir inmediatamente)
1. **[OWASP-AXX]** [fichero:linea] — [descripcion]
   - Riesgo: [que puede explotar un atacante]
   - Remediacion: [codigo o pasos especificos]

#### HIGH (riesgo medio-alto, corregir esta semana)
...

#### MEDIUM (riesgo medio, planificar correccion)
...

#### LOW / INFO (riesgo bajo o informativo)
...

### Dependencias vulnerables
| Paquete | Version actual | CVE | Severidad | Version fix |
...

### Recomendaciones priorizadas
1. [Accion mas urgente]
2. [Segunda accion]
3. ...
```

### PASO 6 — Ofrecer correcciones

Para hallazgos CRITICAL y HIGH, pregunta:

> Quieres que aplique las correcciones de seguridad CRITICAL/HIGH?

Aplica fixes y vuelve a ejecutar Semgrep para verificar que se resolvieron.

---

## Notas

- NUNCA expongas secrets encontrados en el informe. Redacta con `***`.
- Si encuentras un secret en codigo commiteado, alerta inmediatamente: el secret esta comprometido y debe rotarse.
- Para WordPress, prioriza los sitios publicos (proportione.com, porqueviven.org) sobre los de desarrollo.
- Los resultados de Semgrep pueden tener falsos positivos. Verifica manualmente antes de reportar como CRITICAL.
