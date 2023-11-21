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

     - **“Exporte” un directorio y “móntelo” de forma remota sobre un túnel SSH.**[Crear una carpeta compartida]

     - **PARA PLANTEAR DE FORMA TEÓRICA.: Securice su sevidor considerando que únicamente dará servicio ssh para sesiones de usuario desde determinadas IPs.**

2. **Tomando como base de trabajo el servidor Apache2**

     - **Configure una Autoridad Certificadora en su equipo.**[Pasar de HTTP a HTTPS, uno va a ser la entidad certificadora, usar easy-rsa, crear dos ficheros uno público y otro privado. El otro va a ser el servidor y generar fichero público y privado. En el servidor se utiliza su clave pública, también la publica combinada con la privada de la entidad, y luego la pública del servidor. ]

       Actualizamos los repositorios e instalamos openssl

       ```sh
       apt uptdate -y && apt install -y openssl
       ```

       Luego descargamos y descomprimimos easy-rsa 

       ```sh
       wget https://github.com/OpenVPN/easy-rsa/archive/refs/tags/v3.1.7.tar.gz
       tar xzvf v3.1.7.tar.gz
       cd easyrsa3/
       ```

       Luego copiamos vars.examples en otro archivo y lo configuramos

       ```sh
       cp vars.example vars
       
       echo 'set_var EASYRSA_ALGO "ec"' >> vars
       echo 'set_var EASYRSA_DIGEST "sha512"' >> vars
       ```

       Luego editamos:

       ```sh
       nano x509-types/server
       extendedKeyUsage = serverAuth,clientAuth
       ```

       

     

     - **Cree su propio certificado para ser firmado por la Autoridad Certificadora. Bueno, y fírmelo.**
     - **Configure su Apache para que únicamente proporcione acceso a un determinado directorio del árbol web bajo la condición del uso de SSL. Considere que si su la clave privada está cifrada en el proceso de arranque su máquina le solicitará la correspondiente frase de paso, pudiendo dejarla inalcanzable para su sesión ssh de trabajo.**[Crear una carpeta en el raíz de la página web una carpeta securizada, que solo funcione con HTTPS]

3. **Tomando como base de trabajo el openVPN deberá configurar una VPN entre dos equipos virtuales del laboratorio que garanticen la confidencialidad entre sus comunicaciones.**[Crear una VPN, uno hace servidor y cuando el cliente se conecte le asigna una IP de un rango establecido, la conexión debe ir cifrada, clave compartida(pre-shared key). En el cliente se conecta y ambas máquinas hacen ifconfig y luego se hace ping de ambas máquinas]

4. **EN LA PRÁCTICA 1 se configuró una infraestructura con servidores y clientes NTP. Modifique la configuración para autenticar los equipos involucrados.**

5. **EN LA PRÁCTICA 1 se instalaron servidores y clientes de log. Configure un esquema que permita cifrar las comunicaciones.**

6. **En este punto, cada máquina virtual será servidor y cliente de diversos servicios (NTP, syslog, ssh, web, etc.). Configure un “firewall stateful” de máquina adecuado a la situación actual de su máquina.**

7. **Ejecute la utilidad de auditoría de seguridad *lynis* en su sistema y trate de identificar las acciones de securización detectadas así como los consejos sobre las que se deberían contemplar.**

8. **EN LA PRÁCTICA 2 se obtuvo un perfil de los principales sistemas que conviven en su red, puertos accesibles, fingerprinting, paquetería de red, etc. Seleccione un subconjunto de máquinas del laboratorio de prácticas y la propia red. Elabore el correspondiente informe de análisis de vulnerabilidades. Puede utilizar como apoyo al análisis la herramienta Nessus Essentials (disponible para educación en https://www.tenable.com/tenable-for-education/nessus-essentials bajo registro para obtener un código de activación) para su instalación en la máquina debian de prácticas. Como opción alternativa, también podría instalar Greenbone Vulnerability Management (GVM). Como referencia-plantilla puede utilizar.:**

     - **Writing a Penetration Testing Report del SANS (SysAdmmin Audit, Networking and Security) Institute. Muestra las etapas o fases del desarrollo de un “report”, describe el formato del “report” y finaliza con un ejemplo. http://www.sans.org/reading-room/whitepapers/bestprac/writing-penetration-testing-report-33343?show=writing-penetration-testing-report-33343&cat=bestprac**

     - **Plantilla de [vulnerabilityassessment.co.uk](http://www.vulnerabilityassessment.co.uk/report%20template.html).**
