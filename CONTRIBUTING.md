# Contributing

Gracias por considerar contribuir a este marketplace. Estas son las pautas que seguimos nosotros mismos y que facilitan que los cambios externos se integren rápido.

## Antes de empezar

- Lee el [README](README.md) para entender qué son los plugins y las guías.
- Busca en [Issues](https://github.com/Proportione/proportione-plugins/issues) y [Discussions](https://github.com/Proportione/proportione-plugins/discussions) si el tema ya está abierto.
- Para cambios grandes (nuevo plugin, nueva guía, refactor), abre primero una **Discussion** antes del PR. Evita trabajo perdido.

## Cómo reportar un bug

Abre un [Issue](https://github.com/Proportione/proportione-plugins/issues/new/choose) con la plantilla "Bug report". Necesitamos:

- Plugin o guía afectada.
- Claude Code versión (`claude --version`).
- Sistema operativo.
- Pasos para reproducir y resultado esperado vs. observado.
- Si aplica, salida de `claude plugin list` o log relevante.

## Cómo proponer una mejora

Para añadir un plugin nuevo:

1. El plugin debe resolver un problema real que hayas tenido al usar Claude Code en producción.
2. Incluye `plugin.json` válido (ver ejemplos en `plugins/`).
3. Añade un `README.md` corto que explique qué hace y un ejemplo mínimo.
4. Si toca archivos sensibles (claves, configuración), explica el riesgo en la descripción.

Para una guía nueva en `guides/`:

1. Tono: práctico, ejemplos reales, sin teoría vacía.
2. Máximo 4 líneas por párrafo.
3. Código real, no pseudocódigo. Rutas absolutas cuando aplique.

## Proceso de PR

1. Haz fork del repositorio.
2. Crea rama desde `main`: `feat/nombre-corto` o `fix/nombre-corto`.
3. Commits con mensajes descriptivos en imperativo: `feat(deep-review): …`, `fix(check-wordpress): …`.
4. Abre PR contra `main`. La plantilla te pedirá el contexto y el test plan.
5. Esperamos respuesta en 48-72 horas laborables.

## Estilo

- Los plugins siguen el formato de Claude Code (`plugins/<name>/plugin.json` + `plugins/<name>/skill.md`).
- Un PR, una razón. Nada de "y ya de paso limpio X".
- Si un cambio necesita documentación, incluye la documentación en el mismo PR.

## Licencia

Al contribuir aceptas que tu aportación se publique bajo la licencia [MIT](LICENSE) del proyecto.

## Contacto

Para temas sensibles o preguntas que no encajan en Issues: [security@proportione.com](mailto:security@proportione.com) (temas de seguridad) o [contacto@proportione.com](mailto:contacto@proportione.com).
