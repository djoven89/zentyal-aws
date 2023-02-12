# Backup

## AWS DLM



## Módulo de backup



## Funcionalidad backup de configuración

1. Probamos el comando desde la CLI:

    ```sh
    sudo /usr/share/zentyal/make-backup --description "CLI backup on `date '+%d-%m-%Y'`"
    ```

2. Comprobamos el éxito desde la GUI:

    ![Configuration backup from CLI](images/zentyal/backup-zentyal_conf.png "Configuration backup from CLI")

3. También lo comprobamos desde la CLI:

    ```sh
    sudo ls -l /var/lib/zentyal/conf/backups/
    ```

    Un ejemplo:

    ```sh
    -rw-rw---- 1 ebox ebox 3368960 Feb 12 17:26 2023-02-12-172557.tar
    ```

3. Una vez confirmado, creamos la tarea programada. Para ello, creamos el archivo `/etc/cron.d/custom-backup_conf`:

    ```sh
    ## Configuration backup created on 12-02-2023 by Daniel
    30 02 * * * root /usr/share/zentyal/make-backup --description "Cronjob backup on `date '+\%d-\%m-\%Y'`" >/dev/null 2>&1
    ```

    **NOTA:** Es recomendable establecer una hora próxima para confirmarlo.

4. Finalmente, revisamos que la tarea programada:

    ![Configuration backup from Cronjob](images/zentyal/backup-zentyal_conf-cronjob.png "Configuration backup from Cronjob")
