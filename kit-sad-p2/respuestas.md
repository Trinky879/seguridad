# Respuestas · Práctica 2 · Bodegas Atlántico

> Rellena este fichero en tu ordenador. Va en la entrega (`./lab.sh entregar`) y en la defensa te pueden preguntar por él.

## Bloque A · Piensa como un atacante (nmap)

Desde la calle: `./lab.sh cliente` y dentro `nmap -Pn -p 22,80,443,5432,8080 198.51.100.254`. Pega la salida:

```text
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-09 17:15 UTC
Nmap scan report for sad-p2-fw.sad-p2_calle (198.51.100.254)
Host is up (0.000066s latency).

PORT     STATE    SERVICE
22/tcp   filtered ssh
80/tcp   open     http
443/tcp  open     https
5432/tcp filtered postgresql
8080/tcp filtered http-proxy
MAC Address: 62:DF:E9:AF:B3:9A (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.29 seconds

```

## Bloque A · El contador

Después de `./lab.sh comprobar`, mira `./lab.sh ver`. ¿Cuántos paquetes lleva el contador de A6? ¿Quién los ha mandado?

>

## Bloque B · La prueba del cifrado

En el firewall (`./lab.sh fw`): `tcpdump -ni wan host 198.51.100.62 -c 6`. Mientras, en otra terminal, el comercial
(`./lab.sh comercial`) hace `curl http://172.16.10.20:8080`. Pega lo que sale en el firewall:

```text
19:31:03.401296 IP 198.51.100.62.45930 > 198.51.100.254.51820: UDP, length 32
19:31:28.488375 IP 198.51.100.62.45930 > 198.51.100.254.51820: UDP, length 32
19:31:31.864048 IP 198.51.100.62.45930 > 198.51.100.254.51820: UDP, length 96
19:31:31.864186 IP 198.51.100.254.51820 > 198.51.100.62.45930: UDP, length 96
19:31:31.864262 IP 198.51.100.62.45930 > 198.51.100.254.51820: UDP, length 96
19:31:31.864298 IP 198.51.100.62.45930 > 198.51.100.254.51820: UDP, length 176
```

En dos líneas: ¿qué se ve por la calle y qué NO se ve?

>

## Bloque C · Para la defensa (C4)

Del **mismo ataque** (la inyección a la web): ¿qué ve el vigilante de la red (el IDS) y qué ve el portero de la web? ¿Por
qué hacen falta los dos?

**¿Qué ve Suricata?**
Suricata es un IDS de red colocado en la interfaz `wan` del cortafuegos (antes del DNAT). Ve los paquetes de red que entran hacia la IP pública de la bodega (`198.51.100.254`), analiza cabeceras IP/TCP y patrones en el tráfico, y **solo genera alertas** (es pasivo, no bloquea el tráfico). Además, no entiende la lógica completa de la aplicación web y quedaría ciego si el ataque fuera cifrado (HTTPS) o muy ofuscado a nivel HTTP.

**¿Qué ve el WAF (ModSecurity + OWASP CRS)?**
El WAF actúa en línea en la capa de aplicación (capa 7 / HTTP). Lee e interpreta la petición web completa: la ruta (`/catas`), el método (`GET`/`POST`), las cabeceras y cada parámetro concreto (`id`, `notas`), decodificando posibles evasiones. Suma puntos de anomalía y, si se alcanza el umbral, **bloquea activamente la petición** devolviendo `403 Forbidden` antes de que llegue al servidor web.

**¿Por qué hacen falta los dos?**
Porque aplican **defensa en profundidad** cubriendo capas y vectores distintos:
- El WAF protege específicamente la aplicación web (puerto 80) bloqueando ataques como inyecciones SQL o XSS y permitiendo excepciones precisas por página y campo, pero no ve ataques contra otros servicios ni tráfico de red (como un ping, un escaneo de puertos o fuerza bruta al SSH en el puerto 22).
- Suricata vigila toda la red perimetral (todos los puertos y protocolos) para dar visibilidad global y detectar reconocimientos o ataques de red que nunca pasarían por el WAF.

> 

## Bloque D · La nota del servidor (D0 y D4)

Nota del análisis **antes** de bastionar (D0):

>

Nota **después** (D4). ¿Por qué no llega a 100? ¿Significa que el servidor sigue inseguro? Comenta un aviso que siga saliendo.
No llega a 100 porque Lynis evalúa una batería genérica de controles pensada para servidores completos de producción (particiones separadas, contraseñas de arranque, módulos del kernel, herramientas extra de auditoría, etc.), muchos de los cuales no aplican a un contenedor o no son necesarios para este servicio. Un 63/100 no significa que el servidor sea inseguro: se han cerrado los vectores críticos (eliminación del usuario puerta trasera y del binario SUID, SSH sin root ni contraseñas, Fail2ban contra fuerza bruta y cabeceras de seguridad en Nginx).
>
