# CLAUDE.md

## Propósito del repositorio
Repositorio de práctica del curso de Claude Code para TI de EDF. Simula los scripts
operativos de un equipo de TI (respaldos, rotación de logs, notificaciones).
Todos los datos son ficticios; hay secretos sembrados a propósito para la práctica.

## Restricciones
- No leas, muestres ni copies credenciales: archivos `.env*`, la carpeta `llaves/`,
  `~/.ssh/` ni valores de tokens o contraseñas. Si necesitas saber si una variable
  existe, pregunta o revisa el nombre, nunca el valor.

## Antes de cada commit
- Ejecuta `./scripts/revisa-secretos.sh`. Si encuentra algo nuevo (código de salida 1
  con hallazgos que no sean los 5 sembrados para la práctica), no hagas commit y avísame.

## Convenciones
- Todo en español: comentarios, mensajes de commit y documentación.
- Trabaja en una rama; no hagas commit directo a `main`.
- Las credenciales se leen de variables de entorno, nunca se escriben en el código.
- Scripts en bash portable (macOS bash 3.2, Linux y Git Bash). Las reglas detalladas
  están en `.claude/rules/scripts.md` y se cargan solas al trabajar en `scripts/**/*.sh`.

## Dónde está el detalle
Lee el documento que corresponda antes de cambiar algo de esa área:

| Documento | Contenido |
|-----------|-----------|
| `docs/scripts.md` | Qué hace cada script, uso, variables, códigos de salida y problemas conocidos |
| `docs/servidores.md` | Inventario de servidores, estructura de `config/app.conf` y variables de entorno |
| `docs/decisiones.md` | Decisiones del repositorio y por qué se tomaron |
| `docs/inventario-servidores.csv` | Fuente de datos del inventario |

Si una decisión nueva cambia cómo se trabaja aquí, agrégala a `docs/decisiones.md`.
