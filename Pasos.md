# 99-network.conf

## Este archivo ajusta parámetros de sysctl en Linux para endurecer la pila de red:

Desactiva el reenvío de paquetes (IPv4 e IPv6) → el host no actúa como router.

Habilita TCP SYN cookies → protección contra ataques de denegación de servicio (SYN flood).

Bloquea redirecciones ICMP (aceptar y enviar) → evita manipulaciones de rutas y posibles ataques MITM.

Se coloca en el directorio **/etc/sysctl.d** y una vez creado se ejecuta el comando **sysctl --system**