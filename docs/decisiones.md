# Decisiones

Registro corto de las decisiones del repositorio. Cada una dice el contexto, qué se
decidió y qué implica. Agrega las nuevas al final con su fecha.

## 1. Secretos ficticios versionados a propósito (Sesión 1)

- **Contexto:** la práctica consiste en construir una herramienta que encuentre secretos.
- **Decisión:** `.env`, `config/.env.produccion`, `llaves/` y dos credenciales escritas en
  archivos se versionan a propósito, con datos inventados. No están en `.gitignore`.
- **Consecuencia:** `revisa-secretos.sh` siempre reporta 5 hallazgos (ver
  `docs/scripts.md`). En un repositorio real, `.env*` y `llaves/` van en `.gitignore`.

## 2. Solo bash portable, sin dependencias (Sesión 1)

- **Contexto:** el equipo usa macOS (bash 3.2 y herramientas BSD), Linux y Windows con
  Git Bash.
- **Decisión:** scripts en bash con herramientas estándar (`grep -E`, `find`, `tar`,
  `gzip`, `curl`); nada de `jq`, `grep -P` ni opciones exclusivas de GNU.
- **Consecuencia:** a veces el código es más largo (por ejemplo, el JSON se arma a mano en
  `notifica.sh`). Las reglas están en `.claude/rules/scripts.md`.

## 3. Credenciales solo por variable de entorno (Sesión 1)

- **Decisión:** ningún script trae credenciales escritas; se leen del entorno y se validan
  con `${VAR:?mensaje}` sin mostrar el valor.
- **Consecuencia:** `notifica.sh` incumple esta regla a propósito (es un secreto sembrado).

## 4. Finales de línea LF (Sesión 1)

- **Contexto:** con `core.autocrlf=true` en Windows, los scripts con CRLF fallan en Git Bash.
- **Decisión:** `.gitattributes` fuerza `eol=lf` en todo el repositorio.

## 5. `revisa-secretos.sh` antes de cada commit (Sesión 1)

- **Decisión:** se ejecuta antes de cada commit y nunca imprime el valor de un secreto,
  solo archivo, línea y tipo.
- **Consecuencia:** si aparece un hallazgo que no es de los 5 sembrados, no se hace commit.

## 6. Permisos de Claude Code (Sesión 2, 2026-11-18)

- **Contexto:** la regla `deny` sobre `git commit` en `.claude/settings.json` bloqueaba los
  commits aunque hubiera un permiso en otro archivo.
- **Decisión:** Claude Code aplica primero `deny`, luego `ask` y luego `allow`, sin
  importar en qué archivo esté cada regla. Se quitó el `deny` de `git commit` (commit
  `35786c9`). `git push`, `docker`, `ssh`, `systemctl` y `./scripts/*` quedan en `ask`, y
  la lectura de `.env*` y `llaves/` sigue en `deny`.
- **Consecuencia:** las ramas creadas desde `upstream/s2-inicio` sin ese commit siguen
  bloqueando `git commit`. Las reglas `deny` son una barrera, no un aislamiento: un script
  puede leer por dentro rutas que Claude no puede leer directamente.
