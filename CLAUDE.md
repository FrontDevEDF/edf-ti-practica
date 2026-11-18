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
  La lista de los 5 está en `docs/scripts.md` (sección "Secretos sembrados").

## Cómo probar
- No hay pruebas automatizadas. Un script se prueba ejecutándolo desde la raíz en
  Git Bash, también con argumentos que fallen, y revisando el código de salida
  (0 bien, 1 hallazgo o fallo esperado, 2 error de uso).
- `respaldo.sh` exige `DB_HOST`, `DB_USER` y `DB_PASSWORD`; `notifica.sh` siempre sale
  con 1 porque el webhook es ficticio.
- Los problemas conocidos de `docs/scripts.md` (p. ej. `-mtime` en `rota-logs.sh`) son
  intencionales para ejercicios: no los corrijas sin preguntar.

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
