# Práctica 3

1. **Tomando como base de trabajo el SSH pruebe sus diversas utilidades:**

  - **Abra un shell remoto sobre SSH y analice el proceso que se realiza. Configure su fichero ssh_known_hosts para dar soporte a la clave pública del servidor.**
  - **Haga una copia remota de un fichero utilizando un algoritmo de cifrado determinado. Analice el proceso que se realiza.**
  - **Configure su cliente y servidor para permitir conexiones basadas en un esquema de autenticación de usuario de clave pública.**
  - **Mediante túneles SSH securice algún servicio no seguro.**

  - **“Exporte” un directorio y “móntelo” de forma remota sobre un túnel SSH.**

  - **PARA PLANTEAR DE FORMA TEÓRICA.: Securice su sevidor considerando que únicamente dará servicio ssh para sesiones de usuario desde determinadas IPs.**
2. **Tomando como base de trabajo el servidor Apache2**

  - **Configure una Autoridad Certificadora en su equipo.**

  - **Cree su propio certificado para ser firmado por la Autoridad Certificadora. Bueno, y fírmelo.**

  - **Configure su Apache para que únicamente proporcione acceso a un determinado directorio del árbol web bajo la condición del uso de SSL. Considere que si su la clave privada está cifrada en el proceso de arranque su máquina le solicitará la correspondiente frase de paso, pudiendo dejarla inalcanzable para su sesión ssh de trabajo.**
3. **Tomando como base de trabajo el openVPN deberá configurar una VPN entre dos equipos virtuales del laboratorio que garanticen la confidencialidad entre sus comunicaciones.**
4. **EN LA PRÁCTICA 1 se configuró una infraestructura con servidores y clientes NTP. Modifique la configuración para autenticar los equipos involucrados.**
5. **EN LA PRÁCTICA 1 se instalaron servidores y clientes de log. Configure un esquema que permita cifrar las comunicaciones.**
6. **En este punto, cada máquina virtual será servidor y cliente de diversos servicios (NTP, syslog, ssh, web, etc.). Configure un “firewall stateful” de máquina adecuado a la situación actual de su máquina.**
7. **Ejecute la utilidad de auditoría de seguridad *lynis* en su sistema y trate de identificar las acciones de securización detectadas así como los consejos sobre las que se deberían contemplar.**
8. **EN LA PRÁCTICA 2 se obtuvo un perfil de los principales sistemas que conviven en su red, puertos accesibles, fingerprinting, paquetería de red, etc. Seleccione un subconjunto de máquinas del laboratorio de prácticas y la propia red. Elabore el correspondiente informe de análisis de vulnerabilidades. Puede utilizar como apoyo al análisis la herramienta Nessus Essentials (disponible para educación en https://www.tenable.com/tenable-for-education/nessus-essentials bajo registro para obtener un código de activación) para su instalación en la máquina debian de prácticas. Como opción alternativa, también podría instalar Greenbone Vulnerability Management (GVM). Como referencia-plantilla puede utilizar.:**

  - **Writing a Penetration Testing Report del SANS (SysAdmmin Audit, Networking and Security) Institute. Muestra las etapas o fases del desarrollo de un “report”, describe el formato del “report” y finaliza con un ejemplo. http://www.sans.org/reading-room/whitepapers/bestprac/writing-penetration-testing-report-33343?show=writing-penetration-testing-report-33343&cat=bestprac**
  - **Plantilla de [vulnerabilityassessment.co.uk](http://www.vulnerabilityassessment.co.uk/report%20template.html).**