---

tags:
  - Zentyal

---

# Maintenance

On this page, I will explain some actions to periodically review on the Zentyal server to confirm its stability.

## Log files

The first and most important thing is to know if the important log files of the system show any errors. For this project, the most important log files are:

* **/var/log/zentyal/zentyal.log** -> Zentyal modules.
* **/var/log/syslog** -> Status of the services and other generic system events.
* **/var/log/samba/samba.log** -> Domain controller module.
* **/var/log/mail.log** -> Mail module.
* **/var/log/mail.err** -> Mail module.
* **/var/log/sogo/sogo.log** -> Webmail module.
* **/var/log/apache2/** -> Webmail module.
* **/var/log/clamav/** -> Antivirus module.
* **/var/log/letsencrypt/letsencrypt.log** -> Certificates issued by Let's Encrypt with Certbot.
* **/var/log/openvpn/** -> VPN module.
* **/var/log/auth.log** -> Local system authentication.

Below is an example of a search for warnings and errors in the Zentyal log:

```sh linenums="1"
egrep -i '(ERROR|WARN)>' /var/log/zentyal/zentyal.log
```

!!! info

    Warnings are usually not relevant.

The result of a harmless warning and an error:

```text linenums="1"
2023/02/04 20:06:31 WARN> zentyal.psgi:43 Plack::Sandbox::_2fusr_2fshare_2fzentyal_2fpsgi_2fzentyal_2epsgi::__ANON__ - Argument "Icecrown-RC-" isn't numeric in numeric eq (==) at /usr/share/perl5/EBox/OpenVPN/Model/ServerConfiguration.pm line 572.

2023/02/04 20:06:53 ERROR> MyDBEngine.pm:200 EBox::MyDBEngine::_connect - Connection DB Error: Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (13)
```

## State of the system package system

Another critical task to check is whether the server has any broken packages. This can be seen with the following command:

```sh linenums="1"
dpkg -l | egrep -v '^(ii|rc)'
```

An example of a system with no broken packages:

```text linenums="1"
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                  Version                           Architecture Description
+++-=====================================-=================================-============-===============================================================================
```

## System report

It is convenient to generate a system report once a week to see the general status of the server and detect possible incidents. The report can be generated using the CLI as shown below:

```sh linenums="1"
/usr/share/zentyal/smart-admin-report > zentyal-report_12-02-2023
```

Below are some of the most important sections of the report that should be reviewed carefully (**NOTE:** The result shown below is from a system in good condition):

* **Disk usage** -> Available space on disks.

    ```text linenums="1"
    Filesystem      Type      Size  Used Avail Use% Mounted on
    /dev/root       ext4       29G  8.0G   21G  28% /
    /dev/nvme2n1p1  ext4      9.8G   17M  9.3G   1% /var/vmail
    /dev/nvme1n1p1  ext4      9.8G  228K  9.3G   1% /home
    /dev/nvme0n1p15 vfat      105M  5.2M  100M   5% /boot/efi
    ```

* **Network Interfaces where were** -> Network failures.

    ```sh linenums="1"
    Network Interfaces where were 'Down': 0
    ```

* **Server packages** -> Broken packages or packages pending updates.

    ```text linenums="1"
    Broken packages: 0
    Upgradable packages:

    Expanded Security Maintenance for Applications is not enabled.

    0 updates can be applied immediately.

    Enable ESM Apps to receive additional future security updates.
    See https://ubuntu.com/esm or run: sudo pro status

    Last update by Zentyal: 2023-02-122
    ```

* **DNS users on DnsAdmins** -> The special user of the DNS module must exist and belong to the special domain group called DnsAdmins.

    ```text linenums="1"
    dns-arthas
    ```

* **Daemons' information** -> Status of the old demons of the domain controller (they should be inactive).

    ```text linenums="1"
    Status of the daemon: 'smbd': inactive
    State of the daemon: 'smbd': masked

    Status of the daemon: 'nmbd': inactive
    State of the daemon: 'nmbd': masked

    Status of the daemon: 'winbind': inactive
    State of the daemon: 'winbind': masked

    Status of the daemon: 'sssd': inactive
    State of the daemon: 'sssd':
    ```

* **Samba database check** -> Errors in the Samba database.

    ```text linenums="1"
    Checked 3763 objects (0 errors)
    ```

* **DNS alias** -> Special CNAME record in the domain for the domain controller.

    ```text linenums="1"
    cb8c94d6-fde3-4f61-9d61-8b7e6c1ce537._msdcs.icecrown.es is an alias for arthas.icecrown.es.
    ```

* **Mails status** -> The status of the emails managed by the email module.

    ```text linenums="1"
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
