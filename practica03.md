# Práctica 3

1. **Tomando como base de trabajo el SSH pruebe sus diversas utilidades:**[Cambiar el algoritmo de cifrado]

     - **Abra un shell remoto sobre SSH y analice el proceso que se realiza. Configure su fichero ssh_known_hosts para dar soporte a la clave pública del servidor.**

       ```sh
       root@debian:/home/lsi# ssh-keyscan 10.11.50.143 >> /etc/ssh/ssh_known_hosts
       
       # 10.11.50.143:22 SSH-2.0-OpenSSH_9.2p1 Debian-2
       # 10.11.50.143:22 SSH-2.0-OpenSSH_9.2p1 Debian-2
       # 10.11.50.143:22 SSH-2.0-OpenSSH_9.2p1 Debian-2
       # 10.11.50.143:22 SSH-2.0-OpenSSH_9.2p1 Debian-2
       # 10.11.50.143:22 SSH-2.0-OpenSSH_9.2p1 Debian-2
       ```

       

     - **Haga una copia remota de un fichero utilizando un algoritmo de cifrado determinado. Analice el proceso que se realiza.**

       Creamos un fichero y con ```scp``` lo pasamos a otra máquina, y con opción -c escogemos el tipo de cifrado.

       ```sh
       scp -c <tipoDeCifrado> fichero.txt lsi@10.11.50.143
       ```

       - **Tipos de cifrado**
         -  'aes128-ctr', 'aes128-cbc', 'aes192-ctr', 'aes192-cbc', 'aes256-ctr', 'aes256-cbc', 'blowfish-cbc', 'arcfour', 'arcfour128', 'arcfour256', 'cast128-cbc', and '3des-cbc'

       

     - **Configure su cliente y servidor para permitir conexiones basadas en un esquema de autenticación de usuario de clave pública.**[Conectarnos de una máquina a otra se puedo por comando o por ficheros]

       Primero generamos una clave pública y privada con el usuario lsi, que se guardarán en .ssh

       ```sh
       ssh-keygen -t rsa
       ```

       Luego vamos a la carpeta /home/lsi/.ssh y copiamos la clave pública en la máquina del compañero

       ```sh
       ssh-copy-id -i id_rsa.pub lsi@10.11.50.143
       ```

       Repetimos haciendo estos comandos con el usuario root, en este usuario las claves por defectos se guardan en /root/.ssh. Y ya podríamos conectarnos sin necesidad de contraseña.

       

     - **Mediante túneles SSH securice algún servicio no seguro.** [Securizar un servicio con un tunel ssh (puede ser apache) hay que esnifar el tráfico y abrir el paquete con Wireshark para que solo se vea SSH]

       Creamos un túnel con SSH, del puerto 8080 del cliente al 80 de mi servidor web

       ```sh
       ssh -L 8080:10.11.48.142:80 -N lsi@10.11.50.142
       ```

       Ahora el cliente puede acceder a la web desde su puerto 8080

       ```sh
       w3m http://localhost:8080
       ```

     - **"Exporte" un directorio y “móntelo” de forma remota sobre un túnel SSH.**[Crear una carpeta compartida]

     - **PARA PLANTEAR DE FORMA TEÓRICA.: Securice su sevidor considerando que únicamente dará servicio ssh para sesiones de usuario desde determinadas IPs.**

2. **Tomando como base de trabajo el servidor Apache2**

     - **Configure una Autoridad Certificadora en su equipo.**[Pasar de HTTP a HTTPS, uno va a ser la entidad certificadora, usar easy-rsa, crear dos ficheros uno público y otro privado. El otro va a ser el servidor y generar fichero público y privado. En el servidor se utiliza su clave pública, también la publica combinada con la privada de la entidad, y luego la pública del servidor. ]

       EN LA ENTIDAD CERTIFICADORA

       Actualizamos los repositorios e instalamos openssl
       
       ```sh
       apt update -y && apt install -y openssl easy-rsa
       ```

       Vamos a /home/lsi y creamos una carpeta para tener todo lo de easy-rsa
       
       ```sh
       cd /home/lsi/
       make-cadir easyrsa
       cd easyrsa
       ```

       Creamos un PKI y la Entidad Certificadora, cuando pregunte el FQDN ponemos la IP de la Entidad Certificadora
       
       ```sh
       ./easyrsa init-pki
       ./easyrsa build-ca nopass
       ```

       EN EL SERVIDOR WEB

       Vamos a la carpeta de configuración de apache y creamos la carpeta de easy-rsa
       
       ```sh
       cd /etc/apache2
       make-cadir easyrsa
       cd easyrsa
       ```
       
       Luego creamos un PKI y generamos una petición de firma a la Entidad Certificadora, poniendo el nombre de la entidad. Y copiamos la petición generada en  la carpeta /tmp de la entidad
       
       ```sh
       ./easyrsa init-pki
       ./easyrsa gen-req 10.11.50.143
       scp pki/reqs/10.11.50.143.req lsi@10.11.50.143:/tmp
       ```
       
       EN LA ENTIDAD CERTIFICADORA
       
       Importamos la petición, la firmamos y la llevamos la carpeta /tmp del servidor web
       
       ```sh
       ./easyrsa import-req /tmp/10.11.50.143.req 10.11.50.143
       ./easyrsa sign-req server 10.11.50.143
       scp pki/issued/10.11.50.143.crt lsi@10.11.50.142:/tmp
       ```
       
       EN EL SERVIDOR WEB
       
       Copiamos el certificado firmado en pki/private
       
       ```sh
       cp /tmp/10.11.50.143.crt pki/private/
       ```
       
       Después, habilitamos el módulo de ssl, editamos el archivo sites-available/default-ssl.conf y cambiamos las siguientes líneas
       
       ```sh
       a2enmod ssl
       nano sites-available/default-ssl.conf
       
       SSLCertificateFile      /etc/apache2/easyrsa/pki/private/10.11.50.143.crt
       SSLCertificateKeyFile   /etc/apache2/easyrsa/pki/private/10.11.50.143.key
       ```
       
       Y reiniciamos el servidor apache poniendo la contraseña que pusimos durante la creación de las claves del servidor
       
       ```sh
       root@debian:/etc/apache2# systemctl restart apache2
       🔐 Enter passphrase for SSL/TLS keys for 127.0.0.1:443 (RSA): ****
       ```
       
       

     - **Cree su propio certificado para ser firmado por la Autoridad Certificadora. Bueno, y fírmelo.**

       Todo en el apartado de arriba

     - **Configure su Apache para que únicamente proporcione acceso a un determinado directorio del árbol web bajo la condición del uso de SSL. Considere que si su la clave privada está cifrada en el proceso de arranque su máquina le solicitará la correspondiente frase de paso, pudiendo dejarla inalcanzable para su sesión ssh de trabajo.**[Crear una carpeta en el raíz de la página web una carpeta securizada, que solo funcione con HTTPS]

       Creamos una carpeta en el servidor Apache

       ```sh
       mdkir /var/www/html/privada
       ```

       Luego editamos el fichero de configuración para las conexiones HTTP para bloquear el acceso a todo lo que cuelgue de la carpeta privada

       ```sh
       nano /etc/apache2/sites-available/000-default.conf
       
       <VirtualHost *:80>
               ServerAdmin webmaster@localhost
               DocumentRoot /var/www/html
       
               ErrorLog ${APACHE_LOG_DIR}/error.log
               CustomLog ${APACHE_LOG_DIR}/access.log combined
       
       		# Añadimos esta etiqueta
               <LocationMatch "^/privada">
                       Require all denied
               </LocationMatch>
       </VirtualHost>
       ```

       Y reiniciamos Apache

       ```sh
       systemctl reload apache2
       ```

       

3. **Tomando como base de trabajo el openVPN deberá configurar una VPN entre dos equipos virtuales del laboratorio que garanticen la confidencialidad entre sus comunicaciones.**[Crear una VPN, uno hace servidor y cuando el cliente se conecte le asigna una IP de un rango establecido, la conexión debe ir cifrada, clave compartida(pre-shared key). En el cliente se conecta y ambas máquinas hacen ifconfig y luego se hace ping de ambas máquinas]

4. **EN LA PRÁCTICA 1 se configuró una infraestructura con servidores y clientes NTP. Modifique la configuración para autenticar los equipos involucrados.**

5. **EN LA PRÁCTICA 1 se instalaron servidores y clientes de log. Configure un esquema que permita cifrar las comunicaciones.**

6. **En este punto, cada máquina virtual será servidor y cliente de diversos servicios (NTP, syslog, ssh, web, etc.). Configure un "firewall stateful" de máquina adecuado a la situación actual de su máquina.**[Crear un script que borre las reglas, poner todo restrictivo, ponemos las reglas, luego un temporizador de 1 minuto o dos, restablecer las políticas por defecto (las permisivas), y se borran las reglas. Todas las salidas mías (new) permitirlas, y todas las respuestas que me hacen (established) también]

     Kike

7. **Ejecute la utilidad de auditoría de seguridad *lynis* en su sistema y trate de identificar las acciones de securización detectadas así como los consejos sobre las que se deberían contemplar.**[Ejecutar el análisis más completo de Lynis, leer el archivo de retorno y explicar cómo se soluciona lo que nos diga]

   Instalamos **lynis**, creamos una carpeta y ejecutamos un reporte

     ```sh
     apt update -y && apt install -y lynis
     mkdir /home/lsi/lynis
     cd /home/lsi/lynis
     
     lynis audit system --verbose > lynis.txt
     ```

     Leemos el fichero con ```cat```, y buscamos los servicios que nos interesan

     Para SSH obtenemos esta salida:
     ```sh
     [+] Soporte SSH
     ------------------------------------
       - Checking running SSH daemon                               [ ENCONTRADO ]
         - Searching SSH configuration                             [ ENCONTRADO ]
         - OpenSSH option: AllowTcpForwarding                      [ SUGERENCIA ]
         - OpenSSH option: ClientAliveCountMax                     [ SUGERENCIA ]
         - OpenSSH option: ClientAliveInterval                     [ OK ]
         - OpenSSH option: Compression                             [ SUGERENCIA ]
         - OpenSSH option: FingerprintHash                         [ OK ]
         - OpenSSH option: GatewayPorts                            [ OK ]
         - OpenSSH option: IgnoreRhosts                            [ OK ]
         - OpenSSH option: LoginGraceTime                          [ OK ]
         - OpenSSH option: LogLevel                                [ SUGERENCIA ]
         - OpenSSH option: MaxAuthTries                            [ SUGERENCIA ]
         - OpenSSH option: MaxSessions                             [ SUGERENCIA ]
         - OpenSSH option: PermitRootLogin                         [ OK ]
         - OpenSSH option: PermitUserEnvironment                   [ OK ]
         - OpenSSH option: PermitTunnel                            [ OK ]
         - OpenSSH option: Port                                    [ SUGERENCIA ]
         - OpenSSH option: PrintLastLog                            [ OK ]
         - OpenSSH option: StrictModes                             [ OK ]
         - OpenSSH option: TCPKeepAlive                            [ SUGERENCIA ]
         - OpenSSH option: UseDNS                                  [ OK ]
         - OpenSSH option: X11Forwarding                           [ SUGERENCIA ]
         - OpenSSH option: AllowAgentForwarding                    [ SUGERENCIA ]
         - OpenSSH option: AllowUsers                              [ NO ENCONTRADO ]
         - OpenSSH option: AllowGroups                             [ NO ENCONTRADO ]
     ```

     Para NTP:

     ```sh
     [+] Tiempo y sincronización
     ------------------------------------
       - NTP daemon found: ntpd                                    [ ENCONTRADO ]
       - Checking for a running NTP daemon or client               [ OK ]
       - Checking valid association ID's                           [ ENCONTRADO ]
       - Checking high stratum ntp peers                           [ OK ]
       - Checking unreliable ntp peers                             [ NINGUNO ]
       - Checking selected time source                             [ OK ]
       - Checking time source candidates                           [ NINGUNO ]
       - Checking falsetickers                                     [ OK ]
       - Checking NTP version                                      [ ENCONTRADO ]
     ```

     Cómo warnings y sugerencias nos retorna (acorté poniendo los warnings que nos interesan, de NTP y SSH):
     ```sh
     Warnings (4):
       ----------------------------
       ! Found one or more vulnerable packages. [PKGS-7392]
           https://cisofy.com/lynis/controls/PKGS-7392/
     
       ! Nameserver 10.10.102.27 does not respond [NETW-2704]
           https://cisofy.com/lynis/controls/NETW-2704/
     
       ! Nameserver 10.8.12.48 does not respond [NETW-2704]
           https://cisofy.com/lynis/controls/NETW-2704/
     
       ! iptables module(s) loaded, but no rules active [FIRE-4512]
           https://cisofy.com/lynis/controls/FIRE-4512/
     
       Suggestions (56):
       ----------------------------
       * Consider hardening SSH configuration [SSH-7408]
         - Details  : AllowTcpForwarding (set YES to NO)
           https://cisofy.com/lynis/controls/SSH-7408/
     
       * Consider hardening SSH configuration [SSH-7408]
         - Details  : ClientAliveCountMax (set 3 to 2)
           https://cisofy.com/lynis/controls/SSH-7408/
     
       * Consider hardening SSH configuration [SSH-7408]
         - Details  : Compression (set YES to NO)
           https://cisofy.com/lynis/controls/SSH-7408/
     
       * Consider hardening SSH configuration [SSH-7408]
         - Details  : LogLevel (set INFO to VERBOSE)
           https://cisofy.com/lynis/controls/SSH-7408/
     
       * Consider hardening SSH configuration [SSH-7408]
         - Details  : MaxAuthTries (set 6 to 3)
           https://cisofy.com/lynis/controls/SSH-7408/
     
       * Consider hardening SSH configuration [SSH-7408]
         - Details  : MaxSessions (set 10 to 2)
           https://cisofy.com/lynis/controls/SSH-7408/
     
       * Consider hardening SSH configuration [SSH-7408]
         - Details  : Port (set 22 to )
           https://cisofy.com/lynis/controls/SSH-7408/
     
       * Consider hardening SSH configuration [SSH-7408]
         - Details  : TCPKeepAlive (set YES to NO)
           https://cisofy.com/lynis/controls/SSH-7408/
     
       * Consider hardening SSH configuration [SSH-7408]
         - Details  : X11Forwarding (set YES to NO)
           https://cisofy.com/lynis/controls/SSH-7408/
     
       * Consider hardening SSH configuration [SSH-7408]
         - Details  : AllowAgentForwarding (set YES to NO)
           https://cisofy.com/lynis/controls/SSH-7408/
     
       * Enable logging to an external logging host for archiving purposes and additional protection [LOGG-2154]
           https://cisofy.com/lynis/controls/LOGG-2154/
     
       * Check ntpq peers output for time source candidates [TIME-3128]
           https://cisofy.com/lynis/controls/TIME-3128/
     ```
   
   - **Consider hardening SSH configuration [SSH-7408]**
     - **https://cisofy.com/lynis/controls/SSH-7408/**
       - **Details  : AllowTcpForwarding (set YES to NO)**
       
         Permite o no, a los usuarios reenviar puertos TCP a través de SSH, lo más seguro sería establecerlo a NO. Se cambia en /etc/ssh/sshd_config.
       
       - **Details  : ClientAliveCountMax (set 3 to 2)**
       
         Cambia el número de paquetes keepalive de 3 a 2, hace que las conexiones inactivas se desconecten antes.
       
       - **Details  : Compression (set YES to NO)**
       
         Comprime la conexión, pero la hace menos segura. Estableciéndola a NO en /etc/ssh/sshd_config se soluciona el problema
       
       - **Details  : LogLevel (set INFO to VERBOSE)**
       
         En VERBOSE muestra toda la información de SSH, también se configura en /etc/ssh/sshd_config
       
       - **Details  : MaxAuthTries (set 6 to 3)**
       
         Cambia el número máximo de intentos para una conexión de 6 a 3
       
       - **Details  : MaxSessions (set 10 to 2)**
       
         Determina el número máximo de conexiones simultáneas
       
       - **Details  : Port (set 22 to )**
       
         Habría que cambiar el puerto por el que el servidor de SSH escucha las conexiones, para que no sea tan obvio que en ese puerto está el SSH
       
       - **Details  : TCPKeepAlive (set YES to NO)**
       
         Estos paquetes los manda el servidor al cliente cada cierto tiempo para comprobar si una sesión sigue activa. Establecerlo en NO, ayuda a que las conexiones inactivas se cierren rápidamente
       
       - **Details  : AllowAgentForwarding (set YES to NO)**
       
         Esta opción deshabilita el reenvío del agente SSH, lo que hace que no se pueda acceder a la clave privada
       
         
     
   - **Enable logging to an external logging host for archiving purposes and additional protection [LOGG-2154]**
     - **https://cisofy.com/lynis/controls/LOGG-2154/**
     
       Recomiendo guardar los logs en un sitio remoto
     
   - **Check ntpq peers output for time source candidates [TIME-3128]**
     - **https://cisofy.com/lynis/controls/TIME-3128/**
     
       Lynis comprueba si los servidores de NTP están en la descripción general por pares. Las diferencias entre la configuración activa y la almacenada en el disco pueden provocar que la configuración NTP no funcione después de un reinicio.
   
   ​    
   
8. **EN LA PRÁCTICA 2 se obtuvo un perfil de los principales sistemas que conviven en su red, puertos accesibles, fingerprinting, paquetería de red, etc. Seleccione un subconjunto de máquinas del laboratorio de prácticas y la propia red. Elabore el correspondiente informe de análisis de vulnerabilidades. Puede utilizar como apoyo al análisis la herramienta Nessus Essentials (disponible para educación en https://www.tenable.com/tenable-for-education/nessus-essentials bajo registro para obtener un código de activación) para su instalación en la máquina debian de prácticas. Como opción alternativa, también podría instalar Greenbone Vulnerability Management (GVM). Como referencia-plantilla puede utilizar.:**

     - **Writing a Penetration Testing Report del SANS (SysAdmmin Audit, Networking and Security) Institute. Muestra las etapas o fases del desarrollo de un “report”, describe el formato del “report” y finaliza con un ejemplo. http://www.sans.org/reading-room/whitepapers/bestprac/writing-penetration-testing-report-33343?show=writing-penetration-testing-report-33343&cat=bestprac**

     - **Plantilla de [vulnerabilityassessment.co.uk](http://www.vulnerabilityassessment.co.uk/report%20template.html).**
