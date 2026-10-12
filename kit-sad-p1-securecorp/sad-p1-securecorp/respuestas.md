# Práctica 1 de SAD · SecureCorp — Respuestas

**Nombre y apellidos:** adrian fornell
**Usuario:** afornell

Responde con tus palabras, en 1-3 líneas. En la defensa te preguntaré lo mismo en voz alta.

**Contraseñas que has usado** (solo porque es un laboratorio; en una empresa, jamás en un fichero):

- Tu usuario: password123
- mtorres: password123

---

**1. (A1)** ¿Quién es el `issuer` de tu `ca.crt`? ¿Hasta qué fecha es válido? ¿Por qué el `subject`
y el `issuer` de la CA son iguales y los de `ldap.crt` no?

El emisor es afornell Root CA -adrian fornell y pertenece a SecureCorp.
Es valido hasta el 4 de octubre a las 17:33:51 del 2036

**2. (A3)** Pega el comando y el resultado de tus dos búsquedas:

```
a) miembros de rrhh: "ldapsearch -x -H ldaps://ldap.securecorp.local -b "ou=groups,dc=securecorp,dc=local" "cn=rrhh" member"

b) cn y mail de todas las personas: "ldapsearch -x -H ldaps://ldap.securecorp.local -b "ou=people,dc=securecorp,dc=local" cn mail"

```

**3. (A4)** ¿Por qué la clave `ldap.key` tiene que ser de `openldap` y tener permisos 600?

Porque los permisos 600 aseguran que solo ese usuario pueda leerla, evitando que otra gente robe la clave privada

**4. (A4)** ¿Qué valor has puesto en `SLAPD_SERVICES` y por qué?

Puse ldaps:/// ldapi:///. Con esto deshabilitamos el puerto inseguro (389) borrando ldap:///, y obligamos al servicio a escuchar únicamente por la red con cifrado TLS (636) y por el enchufe local interno del sistema

**5. (A4)** Antes de añadir `TLS_CACERT` en el cliente, `ldaps://` no funcionaba. ¿Por qué?

Porque el cliente necesita verificar la identidad del servidor para evitar ataques, Sin indicarle cuál era la CA en la que debía confiar, el cliente rechazaba la conexión por seguridad al ver un certificado desconocido.

**6. (B3)** Pega la salida de `klist` con tus dos tickets. ¿Para qué sirve cada uno? ¿Ha viajado tu
contraseña por la red?

```
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: afornell@SECURECORP.LOCAL

Valid starting       Expires              Service principal
10/12/26 23:33:00  10/13/26 09:33:00  krbtgt/SECURECORP.LOCAL@SECURECORP.LOCAL
        renew until 10/19/26 23:33:00
10/12/26 23:33:17  10/13/26 09:33:00  host/web.securecorp.local@SECURECORP.LOCAL
        renew until 10/19/26 23:33:00
```
El krbtgt  es el ticket principal (o como dijo en clase la "pulsera de discoteca") que te identifica frente a Kerberos y te permite pedir otros tickets. El host/web  es el ticket específico que te autoriza a usar el servicio web. La contraseña nunca viaja por la red; se usa localmente para descifrar la respuesta inicial


**7. (C)** En el `docker-compose.yml`, ¿qué diferencia hay entre `build:` e `image:`? ¿Qué
significa la línea `- "8081:80"` del servicio `phpldapadmin`?

image: descarga una imagen ya fabricada desde internet,mientras que 
build: construye una imagen nueva localmente siguiendo las instrucciones de un Dockerfile. La línea "8081:80" redirige el tráfico del puerto 8081 de tu ordenador real hacia el puerto 80 de ese contenedor.

**8. (C)** ¿Por qué en la máquina `web` no has tenido que escribir a mano `TLS_CACERT`, y en el
cliente sí? ¿Qué pasaría con esa línea del cliente si hicieras `./lab.sh reset`?

