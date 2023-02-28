# Recovery

En este documento se explicarán las tres casuísticas relativas a recoveries que pueden darse, desde la 'más' probable hasta la 'menos'. En los tres casos se harán uso de las políticas de copias de seguridad DLM definidas en el documento de backup.

## Pérdida de información en los recursos compartidos

En este apartado simularé que un usuario ha eliminado un archivo importante llamado `nomimas-2023.pdf` en un recurso compartido llamado `rrhh`. El proceso general consistirá en:

1. Comprobaremos la existencia del recurso y posteriormente lo eliminaremos.
2. Crearemos un volumen EBS de la última snapshot disponible.
3. Montaremos el volumen en una ubicación temporal.
4. Restauraremos el archivo eliminado.
5. Comprobaremos que el usuario vuelve a tener el archivo y que éste es accesible.

### Simulación del desastre

TODO


### Restauración de la snapshot

TODO

### Recuperación del archivo

TODO

### Comprobación final

TODO

## Pérdida de información en el correo

El objetivo de este apartado es simular que un usuario llamado `maria` a eliminado un email llamado `Presupuesto 2023` con un adjunto. El proceso general es muy similar al anterior, que consiste de forma general en:

1. Comprobaremos la existencia del email y posteriormente lo eliminaremos.
2. Crearemos un volumen EBS de la última snapshot disponible.
3. Montaremos el volumen en una ubicación temporal.
4. Restauraremos el email eliminado.
5. Comprobaremos que el usuario vuelve a tener acceso al email.

### Simulación del desastre

Para simular la pérdida de un email importante, usaré el webmail para verificar el correo y posteriormente, lo eliminaré.

1. Nos logeamos con el usuario y verificamos el email:

    !["Check the email"](images/aws/recovery-mail_disaster-1.png "Check the email")

2. Eliminamos el email:

    !["Removing the email 1"](images/aws/recovery-mail_disaster-2.png "Removing the email 1")
    !["Removing the email 2"](images/aws/recovery-mail_disaster-3.png "Removing the email 2")

### Restauración del correo

Ahora que tenemos simulado el desastre, procederemos a realizar las acciones necesarias para recuperar el email.

1. Desde `EC2 -> Elastic Block Store -> Snapshots -> Create volume from snapshot` seleccionamos la última snapshot:

    !["Getting the latest snapshot"](images/aws/recovery-mail_snapshot-1.png "Getting the latest snapshot")

2. Configuramos el volumen temporal:

    **NOTA:** Deberá crearse en la misma zona de disponibilidad.

    !["Creating the volume 1"](images/aws/recovery-mail_snapshot-2.png "Creating the volume 1")
    !["Creating the volume 2"](images/aws/recovery-mail_snapshot-3.png "Creating the volume 2")

3. Verificamos que el volumen haya sido creado con éxito y que esté disponible:

    !["Verifying the volume"](images/aws/recovery-mail_snapshot-4.png "Verifying the volume")

4. Asociamos el volumen a la instancia, para ello, vamos a `Actions -> Attach volume`:

    !["Attaching the volume"](images/aws/recovery-mail_snapshot-5.png "Attaching the volume")

5. Nos conectamos vía SSH al servidor y verificamos que el sistema operativo detecta el nuevo volumen:

    ```sh
    lsblk
    ```

    En mi entorno, el volumen ha sido montado como `nvme3n1`:

    ```text
    NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    nvme1n1      259:0    0   10G  0 disk
    └─nvme1n1p1  259:1    0   10G  0 part /var/vmail
    nvme0n1      259:2    0   30G  0 disk
    ├─nvme0n1p1  259:5    0 29.9G  0 part /
    ├─nvme0n1p14 259:6    0    4M  0 part
    └─nvme0n1p15 259:7    0  106M  0 part /boot/efi
    nvme2n1      259:3    0   10G  0 disk
    └─nvme2n1p1  259:4    0   10G  0 part /home
    nvme3n1      259:8    0   10G  0 disk
    └─nvme3n1p1  259:9    0   10G  0 part
    ```

6. Creamos un directorio temporal donde montaremos el nuevo disco:

    ```sh
    sudo mkdir -v /mnt/mail-recovery
    ```

7. Montamos el volumen:

    ```sh
    sudo mount /dev/nvme3n1p1 /mnt/mail-recovery
    ```

8. Buscamos el correo del usuario `maria` en el directorio donde hemos montado el disco temporal:

    ```sh
    sudo find /mnt/mail-recovery/icecrown.es/maria/ -type f -exec ls -l {} \;
    ```

    Ejemplo en mi servidor:

    ```text
    -rw------- 1 ebox ebox 2180 Feb 27 21:36 /mnt/mail-recovery/icecrown.es/maria/Maildir/dovecot.index.cache
    -rw------- 1 ebox ebox 384 Feb 27 21:33 /mnt/mail-recovery/icecrown.es/maria/Maildir/dovecot.list.index.log
    -rw------- 1 ebox ebox 8 Feb 27 21:33 /mnt/mail-recovery/icecrown.es/maria/Maildir/dovecot-uidvalidity
    -rw------- 1 ebox ebox 102 Feb 27 21:36 /mnt/mail-recovery/icecrown.es/maria/Maildir/dovecot-uidlist
    -rw------- 1 ebox ebox 24 Feb 27 21:36 /mnt/mail-recovery/icecrown.es/maria/Maildir/maildirsize
    -r--r--r-- 1 ebox ebox 0 Feb 27 21:33 /mnt/mail-recovery/icecrown.es/maria/Maildir/dovecot-uidvalidity.63fd1386
    -rw------- 1 ebox ebox 31900 Feb 27 21:36 '/mnt/mail-recovery/icecrown.es/maria/Maildir/cur/1677530165.M104169P13132.arthas,S=31900,W=32366:2,S'
    -rw------- 1 ebox ebox 1124 Feb 27 21:37 /mnt/mail-recovery/icecrown.es/maria/Maildir/dovecot.index.log
    ```

9. Una vez identificado el correo, procedemos a su restauración:

    ```sh
    sudo cp -vp '/mnt/mail-recovery/icecrown.es/maria/Maildir/cur/1677530165.M104169P13132.arthas,S=31900,W=32366:2,S' /var/vmail/icecrown.es/maria/Maildir/cur/
    ```

    Es importante que se use la opción `-p` para preservar los permisos del archivo, de lo contrario, el usuario no podrá acceder a el. Además, también será importante que la restauración se haga en el mismo directorio, que en mi caso es: `icecrown.es/maria/Maildir/cur/`.

10. Desde la cuenta del usuario de correo, verificamos que lo hemos recuperado junto con su adjunto.

    !["Confirming the email recovery"](images/aws/recovery-mail_restoration.png "Confirming the email recovery")
    !["Confirming the email recovery"](images/aws/recovery-mail_restoration-2.png "Confirming the email recovery")

11. Una vez restaurado con éxito el email, procedemos a desmontar el disco y eliminar el directorio temporal creado:

    ```sh
    sudo umount -v /mnt/mail-recovery
    sudo rmdir -v /mnt/mail-recovery
    ```

12. Desvinculamos el volumen EBS de la instancia, para ello, vamos a `Actions -> Detach volume`:

    !["Detaching the volumen"](images/aws/recovery-mail_detach.png "Detaching the volumen")

13. Finalmente, eliminamos el volumen EBS:

    !["Removing the volumen"](images/aws/recovery-mail_volumen-remove.png "Removing the volumen")

## Fallo en el sistema operativo

Para este apartado simularé que el sistema ha quedado totalmente inoperativo debido a que un administrador de sistemas ha eliminado accidentalmente el paquete `zentyal-core`. El proceso general consiste en:

1. Provocaremos el desastre.
2. Crearemos un volumen EBS de la última snapshot disponible.
3. Reemplazaremos el volumen de la instancia.
4. Comprobaremos que el servidor vuelve a estar operativo.

### Simulación del desastre

TODO

### Restauración de la snapshot

TODO

### Restauración del servicio

TODO

### Comprobación final

TODO
