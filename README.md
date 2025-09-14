Un paso a paso de cómo yo instalo Arch Linux
Verificación de conexión a internet

En caso de usar cable Ethernet ya deberías estar conectado de forma automática, podés verificar usando:

ip a


En caso de no usar cable Ethernet, deberás conectarte a través de WiFi con la herramienta iwctl, que ya viene instalada en Arch.

Entramos a la herramienta:

iwctl


Listamos los dispositivos:

device list


Nos debería mostrar algo como wlan0.

Escaneamos redes:

station wlan0 scan
station wlan0 get-networks


Para conectarnos:

station wlan0 connect NOMBRE_RED


Te pedirá la contraseña.

Salimos con exit y probamos la conexión:

ping -c 3 archlinux.org

Verificación y cambio de hora

Verificamos la hora:

timedatectl status


Nos mostrará: Local Time, Universal Time, RTC Time, System clock synchronized, NTP service, RTC in local TZ.

Para ver zonas horarias disponibles:

timedatectl list-timezones
timedatectl list-timezones | grep Montevideo


Definimos la zona horaria:

timedatectl set-timezone America/Montevideo


Luego confirmamos:

timedatectl status

Particionado de disco

Los discos se asignan a un dispositivo de bloque como /dev/sda o /dev/nvme0n1. Para identificarlos:

lsblk
fdisk -l


Los resultados que terminen en rom, loop o airootfs pueden ignorarse, al igual que boot0, boot1.

Elegimos el disco:

fdisk /dev/sda


Crear 4 particiones: / (raíz), /boot (arranque), swap y /home.

Comando n para nueva partición.

Primaria: p.

Partition number: default 1.

Primer sector: enter.

Último sector: tamaño deseado, ej. +20G.

p para verificar.

w para escribir cambios, q para salir sin guardar, m para ver más opciones.

Mis particiones:

/boot → directorio de arranque → 1G

/ → directorio raíz → 190G

/home → directorio de usuario → 235G

swap → partición de intercambio → 32G

Dándole formato a las particiones
mkfs.ext4 /dev/sda3       # Raíz
mkfs.fat -F 32 /dev/sda1  # /boot (EFI)
mkswap /dev/sda2           # Swap
mkfs.ext4 /dev/sdb1       # /home

Montar los sistemas de archivos
mount /dev/sda3 /mnt
mount -o mkdir /dev/sda1 /mnt/boot/efi
mount -o mkdir /dev/sdb1 /mnt/home
swapon /dev/sda2


En caso de que tu PC no soporte EFI, montar /boot directamente en /mnt/boot.

Instalar paquetes esenciales

Pacstrap instala el paquete base, kernel y firmware:

pacstrap -K /mnt base linux-firmware linux


pacstrap es una herramienta de Arch usada durante la instalación de paquetes en un sistema recién montado.

El kernel es el núcleo del sistema operativo, encargado de la comunicación entre hardware y software.

Elección del kernel:

linux → último kernel con mejoras pero puede ser propenso a fallos.

linux-lts → más estable pero sin las últimas mejoras.

Paquetes adicionales recomendados:

pacstrap -K /mnt base linux-firmware linux amd-ucode networkmanager vim man-db man-pages sudo bash-completion base-devel

Generar el archivo fstab
genfstab -U /mnt >> /mnt/etc/fstab


fstab indica qué particiones montar, dónde y con qué opciones cada vez que arranca el sistema.

Chroot
arch-chroot /mnt


Cambia la raíz al nuevo sistema instalado, permitiéndote trabajar como si ya estuvieras dentro de él.

Definir zona horaria
ln -sf /usr/share/zoneinfo/Region/Ciudad /etc/localtime
hwclock --systohc


/etc/adjtime se genera automáticamente y mantiene información sobre el reloj del hardware (RTC).

Confirmar con:

timedatectl status

Idioma del sistema

Por defecto, el idioma y la distribución del teclado están en inglés (US).

Si querés cambiarlo:

vim /etc/locale.gen       # Descomentar locale deseado, ej. es_ES.UTF-8 UTF-8
vim /etc/locale.conf      # LANG=es_ES.UTF-8
vim /etc/vconsole.conf    # KEYMAP=es (opcional)

Configurar hostname
vim /etc/hostname         # Nombre de tu equipo

Establecer contraseña root
passwd

Instalar gestor de arranque
pacman -S grub efibootmgr


Solo grub si usás BIOS tradicional.

GRUB en UEFI
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB

GRUB en BIOS (Legacy)
grub-install --target=i386-pc /dev/sda


Generar archivo de configuración de GRUB:

grub-mkconfig -o /boot/grub/grub.cfg

Reboot
reboot


¡Y listo! 😄
