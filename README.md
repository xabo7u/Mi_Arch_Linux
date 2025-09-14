🔹 Módulo 7: Configuración post-instalación
    • Crear usuario normal y darle privilegios (sudo).
    • Instalar y habilitar gestor de red (ej: NetworkManager).
    • Configurar actualizaciones iniciales (pacman -Syu).
🔹 Módulo 8: Entorno gráfico
    • Instalar servidor gráfico (Xorg o Wayland).
    • Instalar escritorio (ej: GNOME).
    • Instalar y habilitar gestor de display (ej: GDM).
    • Probar login gráfico.
🔹 Módulo 9: Personalización estilo macOS
    • Instalar GNOME Tweaks y extensiones.
    • Instalar temas visuales (WhiteSur, icnons, dock).
    • Configurar atajos, fuentes y apariencia.


Un paso a paso de como yo instalo arch linux

Verificación de conexion a internet.
En caso de usar cable ethernet ya deberias de estar conectado a internet de forma automatica, podes verificar usando ip a.

En caso de no estar usando cable ethernet deberas conectarte a traves de wifi con la herramienta iwctl que ya viene instalada en arch. 

Entremos a la herramienta usando iwctl, listamos los dispositivos con device list y nos deberia de mostrar algo como wlan0. Escaneamos redes con station wlan0 scan, vemos las redes disponibles station wlan0 get-networks, para conectarnos usamos station wlan0 connect NOMBRE_RED, te pedira la contraseña, luego salimos con exit, y probamos conexion ping -c 3 archlinux.org.


Verificación y cambio de hora en el dispositivo.
Usando el comando timedatectl status, veremos: Local Time, Universal Time, RTC Time, System clock synchronized, NTP service, RTC in local TZ.
En mi caso pone que estoy en UTC 0 y claramente no es asi. 

Para ver que zonas horarias hay usamos timedatectl list-timezones, Si le agregamos un grep para filtrar mejor, timedatectl list-timezones | grep Montevideo.

Una vez identificada la zona horaria utilizamos timedatectl set-timezone America/Montevideo en mi caso, toma un ratito en actualizarse. Luego verificamos con timedatectl status.

Particionado de disco.

Los discos se asignan a un dispositivo de bloque como /dev/sda o /dev/nvme0n1. Para identificar los dispositivos utilizamos lsblk o fdisk.

Voy a utilizar fdisk -l, con el mismo listaremos lo anteriormente dicho. Los resultados que terminen en rom, loop o airootfs pueden ignorarse al igual que boot0, boot1.

Una vez identificado el disco a utilizar, usamos fdisk /dev/sda en mi caso. Yo voy a armar 4 particiones, la raiz /, la particion de arranque /boot, el swap [swap], y la particion home /home.

Usando n creamos una nueva particion. Nos dara una serie de preguntas: si queremos que sea particion primaria o extendida y elegiremos primaria “p”, en partition number le pondremos el que uno quiera yo lo dejare default 1 y a medida que agregemos mas particiones ese numero crece de forma automatica, en primer sector daremos enter sin mas, y en el ultimo sector le pondremos el tamaño que queramos por ejemplo +20G. Con “p” verificamos las particiones creadas y con “w” escribimos los cambios en el disco, si algo no te parecio correcto usas “q”, para salir sin borrar cambios o usas “m” para ver el resto de opciones de fdisk.

En mi caso usare dos discos, entonces en el primer disco creo 3 particiones, 1G para la particion /boot aunque con 512Mb serian suficientes me gusta darle de mas. 32G al swap que es el doble de mi ram y lo que sobra se lo dejo a la raiz. En el otro disco armo una sola particion primaria usando todo el espacio del disco, que esa sera para el /home.

Mis particiones:
/boot → directorio de arranque → 1G
​/ → directorio raiz → 190G
/home → directorio de usuario → 235G 
swap → particion de intercambio → 32G

Dandole formato a las particiones: 

A las particiones se les debe dar formato, yo al directorio raiz y directorio de usuario les dare ext4, a la particion de arranque fat32 y a la particion swap formato swap.
Para el sistema de archivos ext4:
mkfs.ext4 /dev/sda3
Para el sistema de archivos fat32:
mkfs.fat -F 32 /dev/sda1
Para el swap:
mkswap /dev/sda2

Montar los sistemas de archivo: 

Para comenzar hay que montar el volumen raiz en /mnt, con el  comando mount:
mount /dev/sda3 /mnt
mount –mkdir /dev/sda1 /mnt/boot/efi
mount –mkdir /dev/sdb1 /mnt/home
swapon /dev/sda2

En caso de tu pc no soporte efi montar la particion /boot simplemente en /mnt/boot
Instalar paquetes esenciales:

Utilizamos pacstrap para instalar el paquete base, un kernel y un firmware de hardware comun, pacstrap es una herramienta de arch usada durante la instalacion de paquetes en un sistema recien montado.
El kernel es el nucleo del sistema operativo, encargado de la comunicacion entre hardware y software.
El kernel a usar depende del uso, el kernel linux es el ultimo con todas las mejoras disponibles pero puede ser propenso a fallos en cambio el kernel-lts es mas estable pero no va a tener las ultimas mejoras.

pacstrap -K /mnt base linux-firmware linux

Ademas agregaremos herramientas que nos podrian servir mas adelante y otro tipo de cosas ejemplos: actualizaciones de microcodigo de la CPU (amd-ucode o intel-ucode), NetworkManager, vim, man-db, man-pages, sudo, bash-completion, base-devel. 

Todo junto queda algo asi: 
pacstrap -K /mnt base linux-firmware linux amd-ucode networkmanager vim man-db man-pages sudo bash-completion base-devel

Generar el archivo fstab:

fstab se utiliza para generar un archivo de configuración que le indica al sistema Linux qué particiones montar, dónde montarlas y con qué opciones, cada vez que arranca.

genfstab -U /mnt >> /mnt/etc/fstab

Chroot:
Cambia la raiz al nuevo sistema instalado, permitiendote trabajar como si ya estuvieras dentro de el. Esto es necesario para configurar el sistema, instalar el bootloader, crear usuarios, etc.

arch-chroot /mnt

Definir zona horaria:

ln -sf /usr/share/zoneinfo/Region/Ciudad /etc/localtime
En los los nombre de region y ciudad deben de ser cambiados por ejemplo America/Montevideo.

Luego ejecutamos:
hwclock –systohc
Para generar el archivo /etc/adjtime, este archivo mantiene informacion sobre el reloj del harward (RTC) y como se debe de ajustar la hora del sistema. Al final confirmamos con el comando timedatectl status.

Idioma del sistema:

Por defecto el idioma y la distribucion del teclado estan en ingles (US), en mi caso lo voy a dejar como esta. Si uno quiere cambiarlo, puede hacerlo de la siguiente forma:
Edite el archivo /etc/locale.gen y descomente el locale necesario ejemplo es_ES.UTF-8 UTF-8.
Cree el archivo locale.conf y defina la variable LANG: vim /etc/locale.conf  y dentro del mismo escribir LANG=es_ES.UTF-8.
Si fuese necesario defina la distribucion del teclado en vconsole.conf,  vim /etc/vconsole.conf y escribimos KEYMAP=es.

Configurar hostname:

Cree el archivo hostname:
vim /etc/hostname → nombre de su equipo

Establecer contraseña root:

passwd

Instalar gestor de arranque:

pacman -S grub efibootmgr

Si estas usando BIOS tradicional solo es necesario grub.

Instalar GRUB en UEFI:
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB

Instalar GRUB en BIOS (Legacy):
grub-install --target=i386-pc /dev/sda

Generar el archivo configuracion: 
grub-mkconfig -o /boot/grub/grub.cfg

Y ahora hacer un reboot :(
