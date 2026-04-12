# Skill: Check Terraform

Validacion de seguridad y correctitud de infraestructura como codigo.

**Target:** $ARGUMENTS (directorio con ficheros .tf, o proyecto completo)

## Configuration

This skill references external paths. Set these environment variables or replace inline:

- `$QA_PROPORTIONE_DIR` — Root of the QA_Proportione repo (e.g. `/path/to/QA_Proportione`)

## Dependencias

```bash
which terraform   # brew install terraform
which checkov     # pip install checkov
which semgrep     # pip install semgrep
```

Si no estan instaladas, informar al usuario y ofrecer instalarlas.

---

## Flujo

### PASO 1 — Identificar alcance

1. Detectar directorio con ficheros `.tf`
2. Leer CLAUDE.md del proyecto si existe
3. Identificar provider (GCP, AWS, Azure)

### PASO 2 — terraform validate

```bash
cd [directorio-terraform]
terraform init -backend=false  # Solo providers, no state
terraform validate
```

Reportar errores de sintaxis o configuracion.

### PASO 3 — terraform plan (si hay state)

Solo si el usuario lo autoriza (puede tener coste o side-effects):

```bash
terraform plan -no-color 2>&1 | tee /tmp/tf-plan.txt
```

Analizar el plan:
- Recursos que se DESTRUYEN (flag critico)
- Cambios inesperados
- Recursos nuevos sin tags

### PASO 4 — Checkov scan

```bash
checkov -d [directorio-terraform] \
  --quiet \
  --compact \
  --framework terraform \
  2>&1 | tee /tmp/checkov-results.txt
```

Clasificar findings:
- **CRITICAL/HIGH**: bloqueantes — deben resolverse
- **MEDIUM**: evaluar con contexto
- **LOW**: informativo

### PASO 5 — Semgrep con reglas custom

```bash
semgrep --config $QA_PROPORTIONE_DIR/rulesets/semgrep/terraform.yml \
  [directorio-terraform]
```

### PASO 6 — Revision manual Claude

Revisar los ficheros .tf buscando:

1. **Secrets hardcoded** — passwords, tokens, API keys en texto plano
2. **IAM excesivo** — roles/owner, roles/editor, allUsers
3. **Recursos publicos** — buckets, Cloud Run, VM sin firewall
4. **Sin encryption** — discos, bases de datos sin CMEK
5. **Sin logging** — audit logs desactivados
6. **Lifecycle rules** — prevent_destroy en recursos criticos
7. **State file** — verificar que no esta en el repo (.gitignore)

### PASO 7 — Generar informe

```markdown
# Check Terraform — [proyecto]
**Fecha:** YYYY-MM-DD
**Directorio:** [path]

## Resumen
- terraform validate: OK/FAIL
- Checkov findings: X CRITICAL, Y HIGH, Z MEDIUM
- Semgrep findings: X ERROR, Y WARNING
- Revision manual: X issues

## Findings criticos
[lista]

## Recomendaciones
[lista priorizada]
```

Guardar en `reports/{YYYY-MM-DD}/{proyecto}/check-terraform.md`
