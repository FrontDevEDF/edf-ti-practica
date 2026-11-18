# Scripts

Todos los scripts están en `scripts/`, se ejecutan desde la raíz del repositorio y usan
solo bash y herramientas estándar (en Windows, con Git Bash). Las reglas para escribirlos
están en `.claude/rules/scripts.md`.

## respaldo.sh

Comprime una carpeta en `respaldos/` con la fecha y hora en el nombre
(`respaldo-<carpeta>-AAAAMMDD-HHMMSS.tar.gz`).

- **Uso:** `./scripts/respaldo.sh <carpeta>`
- **Variables de entorno (obligatorias):** `DB_HOST`, `DB_USER`, `DB_PASSWORD`. Se validan
  con `${VAR:?mensaje}`, que se detiene si falta alguna sin mostrar su valor.
- **Base de datos:** si existe `pg_dump`, también guarda `respaldos/base-<fecha>.sql`. Si
  falla, borra el archivo a medias, avisa y continúa solo con los archivos.
- **Salida:** 0 si crea el respaldo; 2 si falta el argumento o la carpeta no existe.
- `respaldos/` está en `.gitignore`.

## rota-logs.sh

Renombra los `.log` de `logs/` con la fecha de rotación y los comprime con `gzip`.

- **Uso:** `./scripts/rota-logs.sh [dias]` (por omisión, 7).
- **Salida:** 0; si no existe `logs/`, avisa y termina sin hacer nada.
- **Problema conocido:** la línea 25 usa `find ... -mtime -"$DIAS"`, que selecciona los
  logs modificados hace **menos** de N días: rota los recientes, no los viejos. El original
  usaba `+"$DIAS"` y cambió en el commit `76ac01e` (rama de inicio de la Sesión 2),
  probablemente a propósito para un ejercicio. Está documentado, no corregido.

## notifica.sh

Envía un mensaje en JSON con `curl` a un webhook ficticio (dominio `.invalid`, nunca
resuelve), con encabezado de autorización `Bearer`.

- **Uso:** `./scripts/notifica.sh "mensaje"`
- **Salida:** 0 si se envió; 1 si `curl` falla (siempre, porque el webhook no existe);
  2 si falta el mensaje.
- `--max-time 10` evita que se quede colgado.
- **Problemas conocidos:**
  - El token del webhook está escrito en el código (línea 10). Es uno de los secretos
    sembrados; debería leerse de una variable de entorno.
  - El JSON se arma a mano para no depender de `jq`, sin escapar el mensaje: unas
    comillas dentro del mensaje producen JSON inválido.

## revisa-secretos.sh

Busca secretos expuestos y reporta archivo, línea y tipo de cada hallazgo, **nunca el
valor**. Se ejecuta antes de cada commit (ver `CLAUDE.md`).

- **Uso:** `./scripts/revisa-secretos.sh [carpeta]` (por omisión, la actual).
- **Tipos:** `archivo-env` (un `.env` cuenta una vez, como archivo completo),
  `llave-privada`, `token-hardcodeado`, `password-hardcodeado`.
- **Salida:** 0 sin hallazgos, 1 con hallazgos, 2 si la carpeta no existe.
- No revisa `.git/`, `node_modules/` ni `respaldos/`, y se excluye a sí mismo porque
  contiene los patrones.
- Un valor leído de una variable (`"$DB_PASSWORD"`) no es un hallazgo; solo cuentan los
  valores escritos tal cual.

### Secretos sembrados para la práctica

La salida esperada hoy tiene exactamente estos 5 hallazgos. Cualquier otro es nuevo y
bloquea el commit.

| Archivo | Línea | Tipo |
|---------|-------|------|
| `.env` | - | archivo-env |
| `config/.env.produccion` | - | archivo-env |
| `llaves/id_practica` | 1 | llave-privada |
| `scripts/notifica.sh` | 10 | token-hardcodeado |
| `config/app.conf` | 16 | password-hardcodeado |

`llaves/id_practica.pub` es la llave pública: no es un secreto.
