# Línea del tiempo del incidente (datos ficticios)

Fuentes: `incidente/auth.log` e `incidente/nginx-access.log`.

El 22 de noviembre a las 05:13:41 alguien entró por SSH como **`soporte`**, con contraseña,
desde **198.51.100.23**. Fue después de unas 5 horas y media de fuerza bruta lenta desde
4 IPs coordinadas. Una vez dentro intentó hacerse root y lo rechazaron. Ningún límite
vigilaba intentos repartidos entre conexiones e IPs, así que nada los frenó.

## Desfase de reloj

El reloj de **nginx va 7 min 02 s (422 s) por delante** del de auth.log, y el desfase es constante.

- Se midió con el usuario `monitor` (10.20.0.15): entra por SSH y luego consulta `/salud`.
  Las 102 parejas dan exactamente 422 s (`auth.log:18` ↔ `nginx-access.log:6`).
- Lo confirma jlopez: entra por SSH a las 22:10:15 (`auth.log:46`) y su primera petición web
  marca 22:17:40 en nginx (`nginx-access.log:65`), 22:10:38 ya corregida.

Los logs no dicen cuál de los dos relojes es el correcto. **Todas las horas de este documento
están en el reloj de auth.log**; a las de nginx ya se les restaron 7:02.

## Resumen

| | |
|---|---|
| **Inicio** | 21 nov **23:37:10**: primer escaneo web de 203.0.113.45 (`nginx-access.log:453`) |
| **Primer intento SSH del grupo** | 21 nov 23:41:07, desde 203.0.113.45 |
| **Acceso exitoso** | 22 nov **05:13:41**, `soporte` desde 198.51.100.23 (`auth.log:2110`) |
| **Fin** | 22 nov **05:28:10**: el atacante cierra la sesión (`auth.log:2158-2160`) |
| **IPs atacantes** | 203.0.113.45 (51 fallos), 198.51.100.23 (50), 192.0.2.77 (46), 203.0.113.118 (43) |
| **Usuarios intentados** | `soporte` (las 4 IPs), `admin`, `root`, `test`, `administrador`, `usuario`, `operador`, `respaldos`, `sistemas`, `deploy`, `oracle`, `postgres`, `ubuntu`, `ftpuser`, `guest`, `mysql` |

## Línea del tiempo

| Hora (auth) | Evento | Línea |
|---|---|---|
| 21/11 22:00–06:10 | Ruido de fondo: unas 60 IPs prueban 1 a 3 usuarios típicos (`pi`, `root`…) y se van | `auth.log:6` … `:2274` |
| 23:37:10–23:37:23 | **203.0.113.45** (zgrab) escanea la web: `/.aws/credentials`, `/.git/config`, `/backup.zip` → todo 404 | `nginx-access.log:453-464` |
| 23:41:07 | 203.0.113.45 empieza la fuerza bruta SSH | auth.log |
| 23:49:01 | **198.51.100.23** (python-requests) escanea la web: `/.env`, `/wp-login.php`… | `nginx-access.log:505-510` |
| 23:52:40 | 198.51.100.23 prueba `soporte` por primera vez | `auth.log:362` |
| 22/11 00:01:42 | **192.0.2.77** (Go-http-client) escanea y luego ataca SSH desde las 00:05:14 | `nginx-access.log:532-537` |
| 00:16:18 | **203.0.113.118** (curl) escanea y luego ataca SSH desde las 00:19:52 | `nginx-access.log:561-566` |
| 00:00–05:00 | Las 4 IPs alternan: entre 6 y 10 intentos por hora cada una, más sondeos web cada 20–40 min (todos 404) | `nginx-access.log:576-1091` |
| 01:31:09–01:31:42 | 192.0.2.150 lanza 12 intentos en 33 s y desaparece. Usuarios genéricos, sin `soporte`; parece otro bot | auth.log |
| 04:46:06 | Último fallo de 198.51.100.23 contra `soporte` antes del éxito | `auth.log:1989` |
| **05:13:41** | **`Accepted password for soporte from 198.51.100.23 port 37117`**, sesión 5225 | **`auth.log:2110-2112`** |
| 05:13:45–05:15:00 | Últimos fallos de las otras 3 IPs; el grupo deja la fuerza bruta SSH justo tras el éxito | auth.log |
| **05:15:02** | **`soporte : user NOT in sudoers ; ... COMMAND=/bin/bash`**: intentó hacerse root y fue rechazado | **`auth.log:2130`** |
| 05:19:50 | 203.0.113.118 sigue sondeando la web (`/wp-login.php`, `/server-status`) | `nginx-access.log:1150-1151` |
| **05:28:10** | 198.51.100.23 cierra la sesión tras 14 min 29 s dentro | **`auth.log:2158-2160`** |

198.51.100.23 **no hizo ninguna petición web durante la sesión**. Lo que hizo dentro no quedó
en estos logs, salvo el `sudo`. Pendiente: revisar `/home/soporte/.bash_history`, procesos,
`crontab` y `authorized_keys` de `soporte`.

Los otros accesos con contraseña son legítimos: mgarcia (10.20.0.61) falló 2 veces
(`auth.log:226`) y luego entró. jlopez y monitor usan llave pública.

## ¿Por qué no lo detuvo el límite de intentos?

1. **Un solo intento por conexión.** Las 294 líneas `Failed password` vienen de 294 procesos
   `sshd` distintos. Un límite por conexión (`MaxAuthTries`) nunca se alcanza; no aparece ni una
   vez `maximum authentication attempts exceeded`.
2. **Repartido entre 4 IPs.** `soporte` recibió 32 intentos (10 de .23, 8 de .45, 7 de .77 y
   7 de .118). Un bloqueo por IP solo vería de 7 a 10 intentos en 5 horas.
3. **Lento a propósito.** De 6 a 10 intentos por hora por IP, por debajo de un umbral típico
   de fail2ban (5 en 10 min), y alternando usuarios.
4. **Ningún bloqueo actuó.** No hay líneas de fail2ban ni de `Ban` en los logs.
5. **De fondo:** `soporte` acepta contraseña por SSH desde internet, y la contraseña era débil
   o se había filtrado.

No hay `sshd_config` ni configuración de fail2ban en el repo. Los puntos 1 a 4 se deducen del
comportamiento en los logs. La propuesta de corrección está en `propuestas/fail2ban.local`.
