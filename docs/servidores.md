# Servidores y configuración

Todo es ficticio. La fuente del inventario es `docs/inventario-servidores.csv`
(columnas `nombre,rol,sistema_operativo,entorno,responsable`); si cambia, actualiza
también esta página.

## Inventario por entorno

### Producción

| Servidor | Rol | Sistema operativo | Responsable |
|----------|-----|-------------------|-------------|
| `srv-app-prod-01` | Servidor de aplicaciones | RHEL 9.2 | Equipo Operaciones TI |
| `srv-app-prod-02` | Servidor de aplicaciones | RHEL 9.2 | Equipo Operaciones TI |
| `db-prod-01` | Base de datos PostgreSQL | RHEL 9.2 | Equipo Datos |
| `srv-batch-prod-01` | Procesos batch nocturnos | RHEL 8.8 | Equipo Operaciones TI |
| `srv-respaldos-01` | Almacenamiento de respaldos | Ubuntu 20.04 LTS | Equipo Infraestructura |
| `srv-monitoreo-01` | Monitoreo y alertas | Debian 12 | Equipo Infraestructura |
| `srv-ad-01` | Controlador de dominio | Windows Server 2022 | Equipo Infraestructura |

### Desarrollo

| Servidor | Rol | Sistema operativo | Responsable |
|----------|-----|-------------------|-------------|
| `srv-app-dev-01` | Servidor de aplicaciones | Ubuntu 22.04 LTS | Equipo Desarrollo |
| `db-dev-01` | Base de datos PostgreSQL | Ubuntu 22.04 LTS | Equipo Datos |

### QA

| Servidor | Rol | Sistema operativo | Responsable |
|----------|-----|-------------------|-------------|
| `srv-web-qa-01` | Servidor web Nginx | Debian 12 | Equipo Desarrollo |

## `config/app.conf`

Archivo INI. Aquí solo se listan las claves; no copies valores de este archivo (la línea 16
tiene una contraseña sembrada para la práctica).

| Sección | Claves |
|---------|--------|
| `[aplicacion]` | `nombre`, `entorno`, `puerto`, `zona_horaria`, `nivel_log` |
| `[base_datos]` | `host`, `puerto`, `nombre`, `usuario`, la contraseña, `pool_maximo` |
| `[rutas]` | `logs`, `respaldos` |
| `[notificaciones]` | `habilitadas`, `canal` |

## Variables de entorno

Solo nombres. Los valores van en el entorno del servidor, nunca en el repositorio (los
`.env` versionados son secretos sembrados para la práctica).

| Variable | La usa | Para qué |
|----------|--------|----------|
| `DB_HOST` | `respaldo.sh` | Servidor de PostgreSQL para `pg_dump` |
| `DB_USER` | `respaldo.sh` | Usuario de la base |
| `DB_PASSWORD` | `respaldo.sh` | Contraseña de la base (se pasa a `pg_dump` como `PGPASSWORD`) |
| `API_TOKEN` | ningún script todavía | Aparece en los `.env` de práctica |
| `WEBHOOK_URL` | ningún script todavía | Aparece en `config/.env.produccion`; `notifica.sh` aún la tiene escrita en el código |
