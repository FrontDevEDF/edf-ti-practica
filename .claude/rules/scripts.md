---
paths:
  - "scripts/**/*.sh"
---

# Reglas para scripts de bash

Deben funcionar igual en macOS (bash 3.2, herramientas BSD), Linux y Git Bash.
Lo aprendido en la Sesión 1 y en la revisión del PR #3; el porqué está en
`docs/decisiones.md`.

## Estructura
- Empieza con `#!/usr/bin/env bash`, un comentario de qué hace y cómo se usa, y
  `set -euo pipefail`. Comentarios cortos y en español.
- Códigos de salida: 0 bien, 1 hallazgo o fallo esperado, 2 error de uso. Los errores
  van a stderr (`>&2`).
- Rutas relativas a la raíz: `RAIZ="$(cd "$(dirname "$0")/.." && pwd)"`.

## bash 3.2
- Nada de arreglos asociativos (`declare -A`), `mapfile`/`readarray`, `${x,,}`, `${x^^}`
  ni `&>>`. Para contar por tipo, usa una variable por tipo.

## grep
- `grep -E`, nunca `grep -P` (no existe en macOS).
- `[[:space:]]` y `[0-9]` en lugar de `\s` y `\d`.
- Con `--include` y `--exclude`, pon `--include` primero: en GNU grep, si el primer filtro
  es `--exclude`, se cuelan archivos de más.
- Con `pipefail`, un grep sin coincidencias sale con 1 y termina el script: usa
  `|| true`. Pero un código mayor que 1 es un error real (permisos, archivo ilegible):
  no lo ocultes con `2>/dev/null` sin revisar el código.
- `grep -I` se salta archivos binarios completos; tenlo en cuenta si buscas secretos.

## Otras herramientas
- Nada de opciones exclusivas de GNU: `sed -i`, `date -d`, `readlink -f`, `find -printf`,
  `stat -c`.
- En `find -mtime`, `+N` es "hace más de N días" y `-N` es "hace menos de N días".
- Nombres de archivo con espacios: sepáralos con NUL (`grep --null`, `find -print0`) y
  léelos con `while IFS= read -r -d '' f`.
- Si el texto puede traer acentos mal codificados, usa `LC_ALL=C`.

## Secretos
- Nunca imprimas el valor de un secreto: ni la salida de `grep -n` tal cual ni "para
  depurar". Reporta solo archivo, línea y tipo.
- Valida variables obligatorias con `: "${VAR:?Falta la variable VAR}"`, que no muestra el
  valor.
- Un script que lee secretos empieza con `set +x`: con `bash -x` se imprimirían.
- Credenciales siempre desde variables de entorno, nunca escritas en el script.

## Antes de terminar
- Pruébalo en Git Bash y, si puedes, con argumentos que fallen (falta un argumento, una
  carpeta que no existe).
- Ejecuta `./scripts/revisa-secretos.sh` y confirma que solo salen los 5 sembrados.
