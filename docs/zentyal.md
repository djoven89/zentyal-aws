# Zentyal

A continuación se detalla tanto la instalación como configuración del sistema operativo así como de los módulos.


## Objetivo

El objetivo principal de este servidor es configurar como servidor de correo, aunque también se usará para compartir recursos compartidos, los cuales serán accesibles a través del módulo de OpenVPN.

Las acciones que se realizarán son:

1. Instalación del sistema operativo.
2. Instalación y configuración de los siguientes módulos:
    * Network
    * Logs
    * Firewall
    * Software
    * NTP
    * DNS
    * Controlador de dominio
    * Correo
    * Webmail
    * Antivirus
    * Mailfilter
3. Configuración de varios volúmenes EBS (discos) adicionales para los buzones de correo así como para los recursos compartidos.
4. Se securizará el servicio de correo usando: SPF, DKIM y DMARC.

Adicionalmente, mencionar que la información que estableceré para la configuración del servidor será:

* **Nombre del servidor**: arthas
* **Dominio**: icecrown.es
* **IP:** 10.0.1.200/24
* **Tipo de red:** Interna


## Requisitos

* Antes de proceder a realizar las acciones que se explican en las siguientes secciones, es recomendable tener configurado el entorno de AWS tal y como se describe en [este] enlace. Si bien es cierto que no es necesario, es recomendable.
* El despliegue que se explica en este documento sólo ha sido probado sobre el proveedor cloud de Amazon (AWS).
* Se requiere que la instancia (servidor) tenga un mínimo de 2vCPU y 4GB de RAM, ya que el módulo de Antivirus consume bastante recursos.
* El sistema operativo **debe** de la instancia tiene que ser **Ubuntu 20.04 LTS**.


## Consideraciones

* En caso de no tener conocimientos robustos sobre Linux, es recomendable usar la versión comercial, ya que suele venir con acceso a soporte, lo cual puede ser muy útil ante incidencias o actualizaciones de versiones.
* La estabilidad del módulo de red es imperativa, ya que no se tiene acceso físico al servidor para resolver incidencias de dicha índole. Algunas recomendaciones son:
    * Definir previamente la configuración que se establecerá.
    * Asignar una IP concreta a la interfaz de red de la instancia.
    * Establecer la IP de la instancia como estática en Zentyal.
    * Se recomienda configurar la tarjeta de red en Zentyal como interna, así se evita bloquearse a uno mismo durante la configuración inicial.


## Instalación

Para la instalación, se usará [este] script disponible por parte de Zentyal. Mencionar que como es lógico, se instalará Zentyal sin entorno gráfico, ya que no tenemos acceso físico a la instancia.

[este]: https://doc.zentyal.org/es/installation.html#instalacion-sobre-ubuntu-20-04-lts-server-o-desktop

Las acciones a realizar son:

1. Nos conectamos a la instancia usando la clave privada que nos hemos descargado (key pair):

    ```
    ssh -i ~/.aws/keys/KP-Prod-Zentyal.pem ubuntu@arthas.icecrown.es
    ```

2. Actualizamos el sistema:

    ```
    sudo apt update
    sudo apt dist-upgrade -y
    ```

3. Nos creamos un usuario administrador adicional, el cual usaremos para administrar Zentyal - al menos inicialmente - :

    ```
    sudo useradd -m -d /home/djoven -G sudo -s /bin/bash djoven
    sudo passwd djoven
    ```

4. Nos logeamos con dicho usuario y le añadimos nuestra clave pública de SSH para que podamos conectarnos a través de SSH:

    ```
    su - djoven
    mkdir -v .ssh
    touch .ssh/authorized_keys
    vim .ssh/authorized_keys
    ```

5. Creamos un directorio donde almacenaremos el script de instalación de Zentyal:

    ```
    mkdir /opt/zentyal-install
    cd /opt/zentyal-install
    ```

6. Nos descargamos el script y le damos los permisos adecuados:

    ```
    sudo wget https://zentyal.com/zentyal_installer.sh
    sudo chmod 0750 zentyal_installer.sh
    ```

7. Instalamos Zentyal a través del script, contestando `n` a la pregunta sobre la instalación del entorno gráfico:

    ```
    ./zentyal_installer.sh
    Do you want to install the Zentyal Graphical environment? (n|y) n
    ```

    El script nos instalará los siguientes paquetes:

    * zentyal (meta-paquete)
    * zentyal-core
    * zentyal-software

8. Una vez que el script haya terminado, nos logearemos al panel de administración de Zentyal: <https://arthas.icecrown.es:8443>

9. Nos logeamos con el usuario administrador que hemos creado previamente, que en mi caso es `djoven`.

10. En el wizard de configuración inicial, únicamente instalaremos el módulo de [firewall], de esta forma se nos instalará como dependencia el módulo de [network] a su vez.

![Initial wizard - Packages](images/zentyal/01-wizard_packages.png "Initial wizard - Packages")

[firewall]: https://doc.zentyal.org/es/firewall.html
[network]: https://doc.zentyal.org/es/firststeps.html#configuracion-basica-de-red-en-zentyal

11. Configuramos la red como `estática` e `internal` tal y como se ha explicado en el apartado de *consideraciones*.

    **NOTA:** Es posible que al terminar de configurarse la red, se nos reproduzca el bug reportado [aquí]. Si es el caso, simplemente modificar la URL por: <https://arthas.icecrown.es:8443>

![Initial wizard - Network 1](images/zentyal/02-wizard_network-1.png "Initial wizard - Network 1")
![Initial wizard - Network 2](images/zentyal/03-wizard_network-2.png "Initial wizard - Network 2")

12. Una vez que se haya terminado de guardar cambios, podremos empezar a gestionar Zentyal a través del dashboard.

![Zentyal initial dashboard](images/zentyal/04-dashboard_initial.png "Zentyal initial dashboard")

13. Finalmente, antes de procedes a la configuración, realizaremos las siguientes comprobaciones para confirmar la estabilidad del servidor en AWS:

    1. Que los módulos estén habilitados.
    2. Que la máquina tiene acceso a Internet.

        ```
        ping google.es
        ```

    3. Que no haya habido ningún error en el log `/var/log/zentyal/zentyal.log`. A continuación un ejemplo de los registros que fueron registrados en nuestro archivo de log:

        ```
        2022/10/23 08:17:51 DEBUG> PAM.pm:83 Authen::Simple::PAM::check - Successfully authenticated user 'djoven' using service 'zentyal'.
        2022/10/23 08:20:29 INFO> install-packages:61 main:: - Starting package installation process
        2022/10/23 08:20:39 INFO> Base.pm:256 EBox::Module::Base::saveConfig - Saving config for module: network
        2022/10/23 08:20:39 INFO> Base.pm:256 EBox::Module::Base::saveConfig - Saving config for module: network
        2022/10/23 08:20:40 INFO> Service.pm:965 EBox::Module::Service::restartService - Restarting service for module: network
        2022/10/23 08:20:43 INFO> Base.pm:256 EBox::Module::Base::saveConfig - Saving config for module: network
        2022/10/23 08:20:43 INFO> Base.pm:256 EBox::Module::Base::saveConfig - Saving config for module: firewall
        2022/10/23 08:20:43 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: firewall
        2022/10/23 08:20:44 INFO> Service.pm:965 EBox::Module::Service::restartService - Restarting service for module: firewall
        2022/10/23 08:20:49 INFO> install-packages:121 main:: - Package installation process finished
        2022/10/23 08:23:15 INFO> Network.pm:89 EBox::Network::CGI::Wizard::Network::_processWizard - Configuring ens5 as 10.0.1.200/255.255.255.0
        2022/10/23 08:23:15 INFO> Network.pm:93 EBox::Network::CGI::Wizard::Network::_processWizard - Adding gateway 10.0.1.1 for iface ens5
        2022/10/23 08:23:15 INFO> Network.pm:108 EBox::Network::CGI::Wizard::Network::_processWizard - Adding nameserver 8.8.8.8
        2022/10/23 08:23:15 INFO> Network.pm:114 EBox::Network::CGI::Wizard::Network::_processWizard - Adding nameserver 8.8.4.4
        2022/10/23 08:23:17 INFO> GlobalImpl.pm:571 EBox::GlobalImpl::saveAllModules - First installation, enabling modules: network firewall webadmin logs audit firewall
        2022/10/23 08:23:17 INFO> GlobalImpl.pm:574 EBox::GlobalImpl::saveAllModules - Enabling module network
        2022/10/23 08:23:17 INFO> GlobalImpl.pm:574 EBox::GlobalImpl::saveAllModules - Enabling module firewall
        2022/10/23 08:23:17 INFO> GlobalImpl.pm:574 EBox::GlobalImpl::saveAllModules - Enabling module webadmin
        2022/10/23 08:23:18 INFO> GlobalImpl.pm:574 EBox::GlobalImpl::saveAllModules - Enabling module logs
        2022/10/23 08:23:18 INFO> GlobalImpl.pm:574 EBox::GlobalImpl::saveAllModules - Enabling module audit
        2022/10/23 08:23:19 INFO> GlobalImpl.pm:574 EBox::GlobalImpl::saveAllModules - Enabling module firewall
        2022/10/23 08:23:19 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: network
        2022/10/23 08:23:23 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: firewall
        2022/10/23 08:23:23 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: logs
        2022/10/23 08:23:23 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: audit
        2022/10/23 08:23:23 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: firewall
        2022/10/23 08:23:24 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: sysinfo
        2022/10/23 08:23:25 INFO> GlobalImpl.pm:660 EBox::GlobalImpl::saveAllModules - Saving configuration: webadmin
        2022/10/23 08:23:25 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: webadmin
        2022/10/23 08:24:51 INFO> Index.pm:187 EBox::Dashboard::CGI::Index::masonParameters - dashboard1
        ```

    4. Reiniciamos el servidor para asegurarnos de que es capaz de iniciar sin ningún tipo de problema de red.

        ```
        reboot
        ```

    5. Finalmente, volvemos a conectarnos tanto vía SSH como desde la GUI de Zentyal para confirmar que la instalación de Zentyal fue exitosa y que es estable.


### General

### Network

### Logs

### Firewall

### NTP

### DNS

### Controlador de dominio

### Correo

### Webmail

### Antivirus

### Mailfilter

### OpenVPN
