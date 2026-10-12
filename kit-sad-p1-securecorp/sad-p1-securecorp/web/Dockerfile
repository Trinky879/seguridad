# 1 . Imagen de partida
FROM debian:bookworm

ENV DEBIAN_FRONTEND=noninteractive

# 2 . Programas: el cliente de Kerberos y el de LDAP
RUN apt-get update && apt-get install -y \
    krb5-user ldap-utils ca-certificates \
 && apt-get clean && rm -rf /var/lib/apt/lists/*

# 3 . Que confíe en la CA de SecureCorp desde que nace
RUN echo "TLS_CACERT /pki/ca/ca.crt" >> /etc/ldap/ldap.conf

# 4 . Qué hace al arrancar: quedarse encendida sin hacer nada
CMD ["sleep", "infinity"]
