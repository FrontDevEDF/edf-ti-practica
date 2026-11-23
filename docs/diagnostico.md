# Diagnóstico de los servicios (Sesión 3)

Laboratorio local con Docker Compose (`docker-compose.yml`). Datos ficticios.

## Servicios y puertos

| Servicio | Imagen | Puerto en el host → contenedor | Para qué |
|---|---|---|---|
| `web` | nginx:1.30.5 | `8080` → 80 | Sitio interno de inventario. Salud: `GET /salud` responde `ok` |
| `db` | postgres:17.11 | `5433` → 5432 | Base `inventario` (tablas `servidores` y `respaldos`) |

- Config de nginx: `servicios/web/default.conf` (montado de solo lectura).
- Esquema y datos de la base: `servicios/db/01-esquema.sql`, `02-datos.sql`, `03-lector.sql`.
  Solo se ejecutan la primera vez, con el volumen `datos-db` vacío.
- Exportaciones con `\copy ... TO 'reportes/...'` aparecen en `./reportes`.

## Dónde están los logs

| Qué | Dónde |
|---|---|
| nginx y postgres | Salida del contenedor: `docker compose logs web` / `docker compose logs db` (no hay archivos en el repo) |
| Incidente (práctica) | `incidente/auth.log`, `incidente/nginx-access.log`, `incidente/correo-sospechoso.eml` |
| Scripts operativos | `logs/accesos.log`, `logs/aplicacion.log`, `logs/respaldos.log` |

Los logs del incidente son grandes: no los leas completos. Empieza por `wc -l`,
`head`/`tail` y filtra con `grep`/`awk`. Ojo: hay desfase de reloj entre `auth.log`
y `nginx-access.log`; compáralo antes de armar una línea del tiempo.

## Comandos seguros (solo leen)

```bash
docker compose ps                         # estado y puertos
docker compose logs --tail 50 web         # últimas líneas del log (o db)
docker compose exec -T web nginx -t       # valida la config de nginx sin recargar
docker compose exec -T db pg_isready      # ¿postgres acepta conexiones?
curl -I http://localhost:8080             # ¿responde el sitio?
curl http://localhost:8080/salud          # verificación de salud
```

No uses `docker compose config` para diagnosticar: imprime las variables de `.env`
ya resueltas, contraseñas incluidas.

Para reproducir un síntoma, guarda el comando y su salida. Haz el cambio mínimo y
verifica con el mismo comando. Ejemplo de hoy: nginx no arrancaba por
`client_max_body_size 10mb;` (unidad inválida). Se cambió a `10m`, se corrió
`docker compose restart web` y luego `curl -I http://localhost:8080`.

`docker compose restart web` y `docker compose up -d` cambian el estado: no son
de diagnóstico, se hacen solo con permiso. Todo `docker *` pide confirmación
(`.claude/settings.json`, `ask`).

## Comandos prohibidos y por qué

Están en `deny` en `.claude/settings.json`:

| Comando | Por qué |
|---|---|
| `docker compose down -v`, `docker volume rm` | Borran el volumen `datos-db`: se pierde la base |
| `docker system prune` | Borra contenedores, imágenes y volúmenes que no están en uso, también de otros proyectos |
| `rm -rf` | Borrado sin vuelta atrás |
| `sudo`, `iptables`, `ufw` | Cambian la máquina o la red, no el laboratorio |
| `cat .env*`, leer `.env*`, `llaves/`, `~/.ssh/` | Exponen credenciales |

Tampoco hagas escrituras en la base (`INSERT`, `UPDATE`, `DELETE`, `ALTER`) mientras
diagnosticas. Si se pide una corrección, como "actualiza el SO de srv-app-dev-01",
se propone el SQL y lo ejecuta una persona con un rol que tenga permisos. El rol
`lector` la rechaza de todas formas.

## Conectarse a la base como lector

El rol `lector` (`servicios/db/03-lector.sql`) solo tiene `CONNECT` y `SELECT`, y
`default_transaction_read_only = on`. Su contraseña viene de `LECTOR_PASSWORD` en `.env`.
No la escribas en comandos ni en archivos.

Desde el contenedor (no pide contraseña: conexión local):

```bash
docker compose exec db psql -U lector -d inventario
# dentro: \dt  \d servidores  \d respaldos
```

Desde el host o desde Claude Code: servidor MCP `inventario` en `.mcp.json`
(host `localhost`, puerto `5433`, usuario `lector`). Toma la contraseña de la
variable `LECTOR_PASSWORD`, que debe estar exportada en la sesión antes de abrir `claude`.

Nunca te conectes como `postgres` para diagnosticar.
