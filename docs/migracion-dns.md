# Migración DNS de pixxoae.com

Plan para mover la zona `pixxoae.com` de los servidores actuales
(`ns1.eldefiar.com`, `ns2.eldefiar.com`) a un nuevo proveedor de DNS.
**Nada de este documento se ha aplicado.**

Inventario tomado el 2026-11-23 con `dig`, consultando directamente
`ns1.eldefiar.com` (autoritativo). ns1 y ns2 tienen el mismo serial (`2026110601`).

📧 = el registro afecta al correo.

## 1. Inventario actual

### Delegación y zona

| Dato | Valor | Nota |
|---|---|---|
| NS en la zona | `ns1.eldefiar.com`, `ns2.eldefiar.com` | TTL 86400 (24 h) |
| NS en el padre (`.com`) | los mismos | TTL **172800 (48 h)**, no se puede cambiar |
| SOA | `ns1.eldefiar.com. root.ds3.eldefiar.com. 2026110601 3600 1800 1209600 86400` | caché negativa de **24 h** (último campo) |
| DNSSEC | La zona publica DNSKEY, pero **no hay DS en `.com`** | DNSSEC no está activo; no bloquea la migración |
| Comodín (`*`) | No hay | |
| Transferencia (AXFR) | Rechazada | El inventario es parcial, ver §1.3 |

### Registros

| Nombre | Tipo | TTL | Valor | Uso | Correo |
|---|---|---|---|---|---|
| `pixxoae.com` | A | 600 | `216.24.57.1` | Sitio en Render | |
| `www` | CNAME | 600 | `sitio-pixxo.onrender.com.` | Sitio en Render | |
| `mail` | CNAME | 14400 | `pixxoae.com.` | Apunta al sitio. Parece heredado del hosting anterior; puede estar en clientes de correo antiguos | 📧? |
| `ftp` | CNAME | 14400 | `pixxoae.com.` | Heredado del hosting anterior | |
| `pixxoae.com` | **MX** | 14400 | `1 SMTP.GOOGLE.com.` | Entrada de correo (Google Workspace) | 📧 |
| `pixxoae.com` | TXT | 14400 | `v=spf1 include:_spf.google.com ~all` | **SPF** | 📧 |
| `_dmarc` | TXT | 14400 | `v=DMARC1; p=quarantine; sp=quarantine; adkim=r; aspf=r; pct=50; fo=1; rf=afrf; ri=86400; rua=mailto:soporte.interno@eldefiar.com; ruf=mailto:soporte.interno@eldefiar.com` | **DMARC** | 📧 |
| `google._domainkey` | TXT | 600 | `v=DKIM1;k=rsa;p=MIGfMA0…` (RSA 1024) | **DKIM de Google Workspace** | 📧 |
| `default._domainkey` | TXT | 14400 | `v=DKIM1; k=rsa; p=MIIBIjAN…` (RSA 2048, en dos cadenas) | DKIM del hosting anterior | 📧 |
| `s1._domainkey` | CNAME | 14400 | `s1.domainkey.u55864335.wl086.sendgrid.net.` | **DKIM de SendGrid** | 📧 |
| `s2._domainkey` | CNAME | 14400 | `s2.domainkey.u55864335.wl086.sendgrid.net.` | **DKIM de SendGrid** | 📧 |
| `pixxoae.com` | TXT | 14400 | `google-site-verification=rat6b_…` | Verificación de Google (Workspace y Search Console) | 📧 indirecto |
| `pixxoae.com` | TXT | 14400 | `apple-domain-verification=tGj8…` | Verificación de Apple | |
| `pixxoae.com` | TXT | 14400 | `anthropic-domain-verification-z5ntbs=…` | Verificación de Anthropic | |

No hay AAAA, SRV, CAA ni MTA-STS. Para copiar las llaves DKIM completas, no transcribas
esta tabla: usa la exportación de la zona o
`dig +short TXT <selector>._domainkey.pixxoae.com @ns1.eldefiar.com`.

### 1.3 Límites del inventario

`dig` solo encuentra nombres que se le preguntan: probé unos 35 nombres comunes.
Como AXFR está bloqueado, **puede haber registros que no aparecen aquí**. El caso más
probable son los de SendGrid para el dominio de retorno y los enlaces
(`em1234`, `url1234`, `1234`), que tienen números que no se pueden adivinar.
**Antes de empezar, pide a eldefiar la exportación de la zona** (archivo de zona o
BIND) y compárala con esta tabla. También puedes consultar en SendGrid
(Settings → Sender Authentication) qué registros espera.

### 1.4 Hallazgos antes de migrar

- **DMARC envía los reportes a `soporte.interno@eldefiar.com`**, un buzón del
  proveedor que vamos a dejar. Decide antes si deben llegar a un buzón de pixxoae.com.
  Cambiarlo es un cambio aparte; no lo mezcles con la migración.
- **`default._domainkey`** parece del hosting anterior. Cópialo igual: si algo
  todavía firma con ese selector y desaparece, el correo falla DKIM.
- **El SPF no incluye SendGrid.** Es normal si SendGrid usa su propio dominio de
  retorno (`em…`), así que es otra razón para tener la exportación completa.

## 2. Plan por pasos

Si hoy es el **día 0**, el cambio de NS se hace el **día 3**. Ese día debe ser martes
o miércoles por la mañana, nunca viernes ni antes de un día festivo.

### Paso 1. Preparar la zona nueva (día 0, sin impacto)

1. Pide a eldefiar la exportación de la zona y concílala con §1.
2. Crea la zona en el nuevo proveedor y copia **todos** los registros con el mismo
   valor. Empieza por los de correo (MX, SPF, DMARC, los cuatro DKIM) y revísalos
   dos veces.
3. En la zona nueva, pon el último campo del SOA (caché negativa) en **300**. Si algún
   registro faltara, el error dura 5 minutos en lugar de 24 horas.
4. **No actives DNSSEC** en el nuevo proveedor todavía. Hoy no hay DS en `.com`; si
   activas DNSSEC a medias, el dominio deja de resolver.
5. Desde este momento **se congelan los cambios de DNS** hasta el paso 6. Si alguno es
   urgente, se hace en las dos zonas.

**Verificación:** preguntar a los servidores nuevos **antes** de delegar en ellos.
Todas las líneas deben salir `OK`:

```bash
NUEVO=ns1.proveedor-nuevo.example   # el NS que te asigne el proveedor
for q in "pixxoae.com A" "www.pixxoae.com CNAME" "mail.pixxoae.com CNAME" \
         "ftp.pixxoae.com CNAME" "pixxoae.com MX" "pixxoae.com TXT" \
         "_dmarc.pixxoae.com TXT" "google._domainkey.pixxoae.com TXT" \
         "default._domainkey.pixxoae.com TXT" "s1._domainkey.pixxoae.com CNAME" \
         "s2._domainkey.pixxoae.com CNAME"; do
  set -- $q
  a=$(dig +short "$2" "$1" @ns1.eldefiar.com | sort)
  b=$(dig +short "$2" "$1" @"$NUEVO" | sort)
  [ "$a" = "$b" ] && echo "OK    $1 $2" || echo "DIFF  $1 $2"
done
```

Añade al bucle los registros extra que traiga la exportación (§1.3).

### Paso 2. Bajar los TTL (día 1, al menos 48 h antes del cambio)

En **eldefiar**:

| Registro | TTL actual | TTL nuevo | Por qué esperar |
|---|---|---|---|
| NS de la zona | 86400 | 3600 | Hay que esperar las 24 h del TTL anterior |
| Registros de 14400 (MX, TXT, DKIM, CNAME) | 14400 | 300 | Hay que esperar 4 h |
| Registros de 600 (A, www, DKIM google) | 600 | 300 | Hay que esperar 10 min |

En la zona nueva, usa también TTL 300 durante la transición.

El TTL de 48 h de los NS en `.com` **no se puede bajar**: lo fija el registro del
dominio. Por eso el cambio de proveedor tarda hasta 48 h en llegar a todos los
resolvedores, y la vuelta atrás también. Mientras tanto, unos resolvedores preguntan
a eldefiar y otros al proveedor nuevo. No pasa nada porque las dos zonas son idénticas.

**Verificación:** `dig +noall +answer MX pixxoae.com @ns1.eldefiar.com` debe mostrar
TTL 300, y `dig +noall +answer NS pixxoae.com @ns1.eldefiar.com` TTL 3600.

### Paso 3. Cambiar los NS en el registrador (día 3, por la mañana)

1. Repite la verificación del paso 1. Todo debe salir `OK`.
2. En el panel del **registrador** del dominio (no en eldefiar) cambia los NS a los
   del proveedor nuevo.
3. **No borres ni cambies la zona en eldefiar.** Es el plan de vuelta atrás y siguen
   llegando consultas durante 48 h.

**Verificación:**

```bash
dig +noall +authority NS pixxoae.com @a.gtld-servers.net   # el padre ya publica los NS nuevos
dig +short NS pixxoae.com @1.1.1.1                          # pueden tardar hasta 48 h
dig +short NS pixxoae.com @8.8.8.8
```

### Paso 4. Verificar servicios (día 3, en la primera hora, y otra vez a las 24 h y 48 h)

| Servicio | Comprobación |
|---|---|
| Sitio | `curl -sI https://pixxoae.com` y `curl -sI https://www.pixxoae.com` → 200 o 301 |
| 📧 Recibir | Enviar desde una cuenta externa (Gmail, Outlook) a un buzón de pixxoae.com |
| 📧 Enviar | Enviar desde Workspace a una cuenta externa y revisar en "Mostrar original" que salga `SPF: PASS`, `DKIM: PASS` y `DMARC: PASS` |
| 📧 SendGrid | Enviar un correo de prueba y comprobar `dkim=pass` con `s1` o `s2` en los encabezados |
| 📧 Workspace | La consola de administración no debe mostrar alertas de verificación ni de MX |
| Verificaciones TXT | Revisar que Google Search Console, Apple y Anthropic sigan mostrando el dominio verificado |

### Paso 5. Volver a subir los TTL (día 5, cuando lleve 48 h estable)

En la zona nueva, vuelve a los valores originales (14400 y 600; NS 86400 y caché
negativa 86400, o lo que recomiende el proveedor). Vuelve a ejecutar la verificación
del paso 4.

### Paso 6. Cerrar (día 10 en adelante)

- Deja la zona en eldefiar intacta **al menos 7 días** después del cambio de NS y
  después dala de baja.
- Levanta la congelación de cambios.
- Haz los cambios que quedaron pendientes, **de uno en uno** y fuera de la migración:
  - destinatario de DMARC (§1.4)
  - quitar `default._domainkey`, `mail` y `ftp` si se confirma que nadie los usa
  - activar DNSSEC en el nuevo proveedor y publicar el DS en el registrador

## 3. Cómo regresar

| Momento | Acción | Cuánto tarda |
|---|---|---|
| Antes del paso 3 | Nada; el tráfico no ha cambiado. Solo hay que volver a subir los TTL | Inmediato |
| Del paso 3 al 6 | Volver a poner `ns1.eldefiar.com` y `ns2.eldefiar.com` en el registrador. La zona vieja sigue intacta | Hasta 48 h (TTL del padre). Mientras tanto, las dos zonas responden igual |
| Un registro concreto falla (por ejemplo, el MX) | Corregirlo en la zona nueva. Con TTL 300, el arreglo tarda unos 5 minutos | 5 min |
| Después del paso 6 | Ya no hay zona vieja: habría que recrearla desde la exportación | Horas |

**Criterio para volver atrás:** si a la hora del cambio no llega correo, o DKIM y
DMARC fallan y no se arregla en 30 minutos, vuelve a los NS de eldefiar.
