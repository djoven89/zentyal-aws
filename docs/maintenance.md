# Mantenimiento

## Archivos de logs

```sh
egrep -i '(ERROR|WARN)>' /var/log/zentyal/zentyal.log
```

**NOTA:** Los warning no suele ser relevantes.

Ejemplo de un warning inofensivo y un error:

```sh
2023/02/04 20:06:31 WARN> zentyal.psgi:43 Plack::Sandbox::_2fusr_2fshare_2fzentyal_2fpsgi_2fzentyal_2epsgi::__ANON__ - Argument "Icecrown-RC-" isn't numeric in numeric eq (==) at /usr/share/perl5/EBox/OpenVPN/Model/ServerConfiguration.pm line 572.

2023/02/04 20:06:53 ERROR> MyDBEngine.pm:200 EBox::MyDBEngine::_connect - Connection DB Error: Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (13)
```

## Estado del sistema de paquetes del sistema


```sh
dpkg -l | egrep -v '^(ii|rc)'
```

```sh
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                  Version                           Architecture Description
+++-=====================================-=================================-============-===============================================================================
```

## Reporte del sistema

```sh
/usr/share/zentyal/smart-admin-report > zentyal-report_12-02-2023
```

Algunos detalles del reporte:

* Disk usage -> Espacio disponible en los discos.

```sh
Filesystem      Type      Size  Used Avail Use% Mounted on
/dev/root       ext4       29G  8.0G   21G  28% /
/dev/nvme2n1p1  ext4      9.8G   17M  9.3G   1% /var/vmail
/dev/nvme1n1p1  ext4      9.8G  228K  9.3G   1% /home
/dev/nvme0n1p15 vfat      105M  5.2M  100M   5% /boot/efi
```

* Network Interfaces where were -> Fallos de red.

```sh
## Network Interfaces where were 'Down': 0
```

* Server packages -> Paquetes rotos o pendientes por actualizar.

```sh
Broken packages: 0
Upgradable packages:

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

Last update by Zentyal: 2023-02-122
```

* DNS users on DnsAdmins -> El usuario debe pertenecer al grupo especial del dominio `DnsADmins`.

```sh
dns-arthas
```

* Daemons' information -> Todos los demonios deberán estar inactivos.

```sh
Status of the daemon: 'smbd': inactive
State of the daemon: 'smbd': masked

Status of the daemon: 'nmbd': inactive
State of the daemon: 'nmbd': masked

Status of the daemon: 'winbind': inactive
State of the daemon: 'winbind': masked

Status of the daemon: 'sssd': inactive
State of the daemon: 'sssd':
```

* Samba database check -> Sin errores en la base de datos de Samba.

```sh
Checked 3763 objects (0 errors)
```

* DNS alias -> Deberá haber un alias.

```sh
cb8c94d6-fde3-4f61-9d61-8b7e6c1ce537._msdcs.icecrown.es is an alias for arthas.icecrown.es.
```

* Mails status -> El estado de los emails gestionados por los servidores de Zentyal.

```sh
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
