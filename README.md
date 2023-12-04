# CHEATSHEET UNIDAD 4: FTP y vsFTPd

## FTP

- **Entrar dentro del servidor ftp:** `ftp`
- **Conectarse con una dirección:** `open <dirección>`
- **Salir del servidor ftp:** `exit`
- **Salir del entorno ftp:** `quit`

## vsFTPd

- **Instalar vsftpd:** `sudo apt-get install vsftpd`
- **Dirección del archivo de configuración:** `/etc/vsftpd.conf`
- **Comprobar estado de vsftpd:** `systemctl status vsftpd`
- **Restablecer servicio de vsftpd:** `systemctl restart vsftpd`
- **Crear un backup de la configuración:** `sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.backup`

### .Conf:

#### Permitir acceso sólo a usuarios locales:
anonymous_enable=NO
local_enable=YES

## Habilitar la carga de archivos
write_enable=YES

## Encarcelar a los usuarios locales en su directorios de inicio
chroot_local_user=YES

## Tendremos problemas para conectarnos realizando el paso anterior, para solucionarlo: 
- Opción 1:
Simplemente permite que el directorio raíz al que nos conectamos pueda ser
escribible.
allow_writeable_chroot=YES
- Opción 2:
Definir un directorio de acceso por FTP distinto al home del usuario y sin
permisos de escritura.
user_sub_token=$USER
local_root=/home/$USER/ftp
Esto usa la variable de entorno $USER para cada usuario del sistema operativo.
- Opción 3: Enjaular al usuario en otra carpeta distinta
sudo usermod --home <carpeta> <usuario>
Restricción de usuarios:
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
Esto habilita la lista de usuarios, para añadir un usuario a esa lista basta con:
echo "<usuario>"| sudo tee -a /etc/vsftpd.userlist

## FTPS (File Transfer Protocol Secure)
Para generar un certificado autofirmado con OpenSSL, se utilizará este comando:
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.pem -out /etc/ssl/private/vsftpd.pem
Este comando genera un certificado SSL autofirmado válido por 365 días y guarda la clave privada y el certificado en /etc/ssl/private/vsftpd-cert.pem
Una vez creado habrá que modificar y añadir unas líneas al fichero vsftpd.conf:
Eliminaremos estas:
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO
Y añadiremos estas:
rsa_cert_file=/etc/ssl/private/vsftpd.pem
rsa_private_key_file=/etc/ssl/private/vsftpd.pem
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
require_ssl_reuse=NO
ssl_ciphers=HIGH
Abrir puertos para el canal de datos en la conexión pasiva:
pasv_enable=YES
pasv_min_port=1027
pasv_max_port=1030
pasv_address=X.X.X.X / Reemplazar por la IP de tu servidor.

## SFTP (SSH File Transfer Protocol)
Comprobar la conexión ssh -> ssh -i clave user2@ipserver
Conexión en terminal -> sftp -i <clave.pem> <usuario>@<dirección>
