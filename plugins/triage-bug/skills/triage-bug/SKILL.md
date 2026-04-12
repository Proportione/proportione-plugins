# Skill: Triage Bug

Triaje automatico de bugs con analisis de causa raiz y generacion de fix.

**Input:** $ARGUMENTS (error message, stack trace, URL, screenshot, o descripcion)

---

## Flujo

### PASO 1 — Recopilar contexto

1. **Parsear el error**: extraer mensaje, stack trace, fichero, linea
2. **Identificar servicio**: que proyecto/servicio esta afectado?
3. **Leer CLAUDE.md** del servicio para entender arquitectura
4. **Clasificar severidad**:
   - **P0** — Servicio caido, datos corruptos, seguridad
   - **P1** — Funcionalidad rota para usuarios
   - **P2** — Bug menor, workaround disponible
   - **P3** — Mejora, no es un bug real

### PASO 2 — Analisis de causa raiz

1. **Leer el codigo** del fichero/linea del error
2. **git blame**: cuando se introdujo? que commit?
3. **git log**: cambios recientes en los ficheros afectados
4. **Buscar patron**: el mismo error ocurre en otros sitios del codigo?
5. **Verificar tests**: hay tests que cubran este caso? por que no lo detectaron?

### PASO 3 — Diagnostico

Producir un diagnostico estructurado:

```markdown
## Diagnostico

**Error:** [mensaje]
**Servicio:** [nombre]
**Fichero:** [path:linea]
**Severidad:** P0/P1/P2/P3
**Causa raiz:** [explicacion clara, no tecnica si es para cliente]
**Introducido en:** [commit hash + fecha]
**Tests existentes:** Si/No — [por que no lo detectaron]
```

### PASO 4 — Proponer fix

1. **Escribir el fix** — minimo cambio necesario, sin refactoring extra
2. **Escribir test** que reproduzca el bug y verifique el fix
3. **Verificar** que los tests existentes siguen pasando
4. **Revisar** que el fix no introduce regresiones

### PASO 5 — Crear PR (si el usuario lo autoriza)

```bash
git checkout -b fix/[descripcion-corta]
# Aplicar cambios
git add [ficheros]
git commit -m "fix: [descripcion]

Causa raiz: [explicacion]
Closes: #[issue-number si aplica]"
```

Proponer `gh pr create` con:
- Titulo descriptivo
- Cuerpo con diagnostico del Paso 3
- Label `bug` + severidad

### PASO 6 — Generar informe

```markdown
# Triage Bug — [servicio]
**Fecha:** YYYY-MM-DD
**Severidad:** P0/P1/P2/P3
**Tiempo de triaje:** Xmin

## Error
[mensaje completo]

## Causa raiz
[explicacion]

## Fix aplicado
[resumen del cambio]

## Prevencion
[que test/regla se anade para evitar recurrencia]
```

Guardar en `reports/{YYYY-MM-DD}/{proyecto}/triage-bug.md`

---

## Reglas de refinamiento (se actualizan con cada uso)

1. No asumir que el primer error es la causa raiz — seguir la cadena completa
2. Verificar siempre en el entorno correcto (dev vs prod)
3. El fix debe ser el minimo viable — no refactorizar codigo adyacente
4. Siempre anadir un test que reproduzca el bug
5. Si el bug es de seguridad (P0), notificar inmediatamente
