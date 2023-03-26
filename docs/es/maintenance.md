# Mantenimiento

En esta página explicaré algunas acciones a revisar periódicamente en el servidor Zentyal para confirmar su estabilidad.

## Archivos de logs

Lo primero y más importante es saber si los archivos de logs importantes del sistema muestra algún error. Para este proyecto, los archivos de logs más importantes son:

* **/var/log/zentyal/zentyal.log** -> Módulos de Zentyal.
* **/var/log/syslog** -> Estado de los servicios y otros eventos genéricos del sistema.
* **/var/log/samba/samba.log** -> Módulo de controlador de dominio.
* **/var/log/mail.log** -> Módulo de correo.
* **/var/log/mail.err** -> Módulo de correo.
* **/var/log/sogo/sogo.log** -> Módulo de webmail.
* **/var/log/apache2/** -> Módulo de webmail.
* **/var/log/clamav/** -> Módulo de antivirus.
* **/var/log/letsencrypt/letsencrypt.log** -> Certificados emitidos por Let's Encrypt con Certbot.
* **/var/log/openvpn/** -> Módulo de vpn.
* **/var/log/auth.log** -> Autenticación locales del sistema.

A continuación un ejemplo de una búsqueda de warnings y errores en el log de Zentyal:

```sh
egrep -i '(ERROR|WARN)>' /var/log/zentyal/zentyal.log
```

!!! info

    Los warning no suele ser relevantes.

El resultado de un warning inofensivo y un error:

```text
2023/02/04 20:06:31 WARN> zentyal.psgi:43 Plack::Sandbox::_2fusr_2fshare_2fzentyal_2fpsgi_2fzentyal_2epsgi::__ANON__ - Argument "Icecrown-RC-" isn't numeric in numeric eq (==) at /usr/share/perl5/EBox/OpenVPN/Model/ServerConfiguration.pm line 572.

2023/02/04 20:06:53 ERROR> MyDBEngine.pm:200 EBox::MyDBEngine::_connect - Connection DB Error: Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (13)
```

## Estado del sistema de paquetes del sistema

Otra tarea crítica a revisar es si el servidor tiene algún paquete roto. Esto lo se puede ver con el siguiente comando:

```sh
dpkg -l | egrep -v '^(ii|rc)'
```

Un ejemplo de un sistema sin ningún paquete roto:

```text
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                  Version                           Architecture Description
+++-=====================================-=================================-============-===============================================================================
```

## Reporte del sistema

Es conveniente generar un reporte del sistema una vez a la semana para ver el estado general del servidor y detectar posibles incidencias. El reporte se puede generar usando la CLI como se muestra a continuación:

```sh
/usr/share/zentyal/smart-admin-report > zentyal-report_12-02-2023
```

A continuación algunas de las secciones más importantes del reporte que hay que revisar con detenimiento (**NOTA:** El resultado mostrado a continuación es de un sistema en buen estado):

* **Disk usage** -> Espacio disponible en los discos.

    ```text
    Filesystem      Type      Size  Used Avail Use% Mounted on
    /dev/root       ext4       29G  8.0G   21G  28% /
    /dev/nvme2n1p1  ext4      9.8G   17M  9.3G   1% /var/vmail
    /dev/nvme1n1p1  ext4      9.8G  228K  9.3G   1% /home
    /dev/nvme0n1p15 vfat      105M  5.2M  100M   5% /boot/efi
    ```

* **Network Interfaces where were** -> Fallos de red.

    ```sh
    Network Interfaces where were 'Down': 0
    ```

* **Server packages** -> Paquetes rotos o pendientes por actualizar.

    ```text
    Broken packages: 0
    Upgradable packages:

    Expanded Security Maintenance for Applications is not enabled.

    0 updates can be applied immediately.

    Enable ESM Apps to receive additional future security updates.
    See https://ubuntu.com/esm or run: sudo pro status

    Last update by Zentyal: 2023-02-122
    ```

* **DNS users on DnsAdmins** -> El usuario especial del módulo de DNS debe existir y pertenecer al grupo especial del dominio llamado `DnsADmins`.

    ```text
    dns-arthas
    ```

* **Daemons' information** -> Estado de los demonios antiguos del controlador del dominio (deben estar *inactivos*).

    ```text
    Status of the daemon: 'smbd': inactive
    State of the daemon: 'smbd': masked

    Status of the daemon: 'nmbd': inactive
    State of the daemon: 'nmbd': masked

    Status of the daemon: 'winbind': inactive
    State of the daemon: 'winbind': masked

    Status of the daemon: 'sssd': inactive
    State of the daemon: 'sssd':
    ```

* **Samba database check** -> Errores en la base de datos de Samba.

    ```text
    Checked 3763 objects (0 errors)
    ```

* **DNS alias** -> Registro especial de tipo CNAME en el dominio para el controlador de dominio.

    ```text
    cb8c94d6-fde3-4f61-9d61-8b7e6c1ce537._msdcs.icecrown.es is an alias for arthas.icecrown.es.
    ```

* **Mails status** -> El estado de los emails gestionados por el módulo de correo.

    ```text
    Mail queue:
    Mail queue is empty
    Mails sent: 2
    Mails rejected: 0
    Mails bounced: 0
    Mails analized by Mailfilter: 1
    Mails with virus: 0
    Mails block by SPAM: 0
    Mails block by File Type: 0
    ```
