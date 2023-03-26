# Certificados

Zentyal por defecto uso certificados auto firmados para sus módulos, incluyendo el uso del módulo de CA. Esta situación provoca que se muestren warning como por ejemplo, usando clientes de correo o al acceder al webadmin o webmail. Es por esto que en esta página se mostrará como generar certificados reconocidos emitidos por Let's Encrypt.

Las acciones que realizaré para el proyecto serán:

1. Emitiré 2 certificados, teniendo uno de ellos dos subdominios:
    * Módulo de webadmin: `arthas.icecrown.es`
    * Módulo de correo y webmail: `mail.icecrown.es` y `webmail.icecrown.es`
2. Usaré el [challenge] de tipo HTTP.
3. Usaré la cuenta de correo `it.infra@icecrown.es` como cuenta de correo para la recepción de notificaciones por parte de Let's Encrypt.

[challenge]: https://letsencrypt.org/docs/challenge-types/

A continuación se indican las acciones a realizar antes de proceder a la generación de los certificados.

1. Instalamos los paquetes necesarios para la generación de los certificados:

    ```bash
    sudo apt update
    sudo apt install -y certbot python3-certbot-apache
    ```

2. Creamos una regla temporal en el firewall de Zentyal como en el security group de AWS que permita el protocolo HTTP para que el certificado pueda expedirse:

    Para el firewall de Zentyal:
    ![Firewall rule in Zentyal](assets/images/zentyal/certificates_firewall-zentyal.png "Firewall rule in Zentyal")

    Para el security group de AWS:
    ![Firewall rule in AWS](assets/images/zentyal/certificates_firewall-aws.png "Firewall rule in AWS")

    !!! nota

        Podremos eliminar esta regla una vez hayamos emitidos los certificados o la mantendremos para evitar tener que volver a establecerla cuando toque la renovación de los certificados.

3. Creamos la cuenta de correo que recibirá las notificaciones:

    ![Mail account for notifications](assets/images/zentyal/certificates-email_account.png "Mail account for notifications")

4. Comprobaremos que desde el exterior podemos resolver los subdominios en cuestión:

    ```bash
    dig arthas.icecrown.es @8.8.8.8
    dig mail.icecrown.es @8.8.8.8
    dig webmail.icecrown.es @8.8.8.8
    ```

    El resultado que obtengo en mi caso es:

    ```text
    ## Webadmin
    ; <<>> DiG 9.16.1-Ubuntu <<>> arthas.icecrown.es @8.8.8.8
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25953
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 512
    ;; QUESTION SECTION:
    ;arthas.icecrown.es.		IN	A

    ;; ANSWER SECTION:
    arthas.icecrown.es.	120	IN	A	15.237.168.75

    ;; Query time: 20 msec
    ;; SERVER: 8.8.8.8#53(8.8.8.8)
    ;; WHEN: Sat Feb 18 22:20:39 CET 2023
    ;; MSG SIZE  rcvd: 63


    ## Mail
    ; <<>> DiG 9.16.1-Ubuntu <<>> mail.icecrown.es @8.8.8.8
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37980
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 512
    ;; QUESTION SECTION:
    ;mail.icecrown.es.		IN	A

    ;; ANSWER SECTION:
    mail.icecrown.es.	300	IN	CNAME	arthas.icecrown.es.
    arthas.icecrown.es.	120	IN	A	15.237.168.75

    ;; Query time: 16 msec
    ;; SERVER: 8.8.8.8#53(8.8.8.8)
    ;; WHEN: Sat Feb 18 22:20:39 CET 2023
    ;; MSG SIZE  rcvd: 82


    ## Webmail
    ; <<>> DiG 9.16.1-Ubuntu <<>> webmail.icecrown.es @8.8.8.8
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45405
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 512
    ;; QUESTION SECTION:
    ;webmail.icecrown.es.		IN	A

    ;; ANSWER SECTION:
    webmail.icecrown.es.	300	IN	CNAME	arthas.icecrown.es.
    arthas.icecrown.es.	120	IN	A	15.237.168.75

    ;; Query time: 28 msec
    ;; SERVER: 8.8.8.8#53(8.8.8.8)
    ;; WHEN: Sat Feb 18 22:20:40 CET 2023
    ;; MSG SIZE  rcvd: 85
    ```

## Webadmin

Para generar el certificado para el **Webadmin (panel de administración)** usaremos el paquete `python3-certbot-apache` en lugar de `python3-certbot-nginx` debido a que Zentyal ejecuta Nginx de forma personalizada, lo que provoca errores a la hora de generar certificados.

1. Generaremos el certificado:

    ```bash
    sudo certbot certonly \
        --apache \
        --preferred-challenges http \
        -m it.infra@icecrown.es \
        --agree-tos \
        -d arthas.icecrown.es
    ```

    Un ejemplo del resultado:

    ```text
    Saving debug log to /var/log/letsencrypt/letsencrypt.log
    Plugins selected: Authenticator apache, Installer apache
    Obtaining a new certificate
    Performing the following challenges:
    http-01 challenge for arthas.icecrown.es
    Enabled Apache rewrite module
    Waiting for verification...
    Cleaning up challenges

    IMPORTANT NOTES:
    - Congratulations! Your certificate and chain have been saved at:
    /etc/letsencrypt/live/arthas.icecrown.es/fullchain.pem
    Your key file has been saved at:
    /etc/letsencrypt/live/arthas.icecrown.es/privkey.pem
    Your cert will expire on 2023-05-19. To obtain a new or tweaked
    version of this certificate in the future, simply run certbot
    again. To non-interactively renew *all* of your certificates, run
    "certbot renew"
    - If you like Certbot, please consider supporting our work by:

    Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
    Donating to EFF:                    https://eff.org/donate-le
    ```

2. Con el certificado generado, tendremos que modificar la plantilla de configuración ([stub]) del módulo, para que dicho cambio sea persistente ante futuras actualizaciones del módulo por parte de Zentyal. Para ello, crearemos los directorios necesarios:

    ```bash
    sudo mkdir -vp /etc/zentyal/stubs/core
    ```

3. Copiamos la plantilla a modificar:

    ```bash
    sudo cp -v /usr/share/zentyal/stubs/core/nginx.conf.mas /etc/zentyal/stubs/core/
    ```

4. Modificamos en la plantilla recién copiada los siguientes parámetros de configuración:

    ```text
    ## Custom certificates issued on 18-02-2023 by Daniel
    # ssl_certificate <% $zentyalconfdir %>ssl/ssl.pem;
    # ssl_certificate_key <% $zentyalconfdir %>ssl/ssl.pem;
    ssl_certificate  /etc/letsencrypt/live/arthas.icecrown.es/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/arthas.icecrown.es/privkey.pem;
    ```

5. Opcionalmente, también modificaré los siguientes parámetros de configuración, cuyo valores ha han sido generados desde [esta](https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=intermediate&openssl=1.1.1k&hsts=false&ocsp=false&guideline=5.6) página web.

    ```text
    ## Custom configuration applied on 18-02-2023 by Daniel
    ## https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=intermediate&openssl=1.1.1k&hsts=false&ocsp=false&guideline=5.6
    # ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    # ssl_ciphers "ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!3DES:!MD5:!PSK";
    # ssl_prefer_server_ciphers on;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ```

6. Paramos el módulo del Webadmin, recargaremos Systemd y volvemos a iniciarlo para aplicar estos cambios:

    ```bash
    sudo zs webadmin stop
    sudo systemctl daemon-reload
    sudo zs webadmin restart
    ```

7. Finalmente, accedemos al webadmin para confirmar que el certificado es correcto:

    ![Webadmin certificate verification](assets/images/zentyal/certificate-webadmin_login.png "Webadmin certificate verification")

[stub]: https://doc.zentyal.org/en/appendix-c.html#stubs

## Mail y Webmail

Para los módulos **Mail** y **Webmail** usaré el mismo certificado, es decir, el certificado será emitido con 2 subdominios en lugar de 1.

1. Generaremos el certificado:

    ```bash
    sudo certbot certonly \
        --apache \
        --preferred-challenges http \
        -m it.infra@icecrown.es \
        --agree-tos \
        -d mail.icecrown.es \
        -d webmail.icecrown.es
    ```

    Un ejemplo del resultado:

    ```text
    Saving debug log to /var/log/letsencrypt/letsencrypt.log
    Plugins selected: Authenticator apache, Installer apache
    Obtaining a new certificate
    Performing the following challenges:
    http-01 challenge for mail.icecrown.es
    http-01 challenge for webmail.icecrown.es
    Enabled Apache rewrite module
    Waiting for verification...
    Cleaning up challenges

    IMPORTANT NOTES:
    - Congratulations! Your certificate and chain have been saved at:
    /etc/letsencrypt/live/mail.icecrown.es/fullchain.pem
    Your key file has been saved at:
    /etc/letsencrypt/live/mail.icecrown.es/privkey.pem
    Your cert will expire on 2023-05-19. To obtain a new or tweaked
    version of this certificate in the future, simply run certbot
    again. To non-interactively renew *all* of your certificates, run
    "certbot renew"
    - If you like Certbot, please consider supporting our work by:

    Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
    Donating to EFF:                    https://eff.org/donate-le
    ```

Con el certificado correctamente generado, procederé a configurar los ambos módulos para que hagan uso de el.

### Webmail

Para este módulo, no será necesario editar un [stub](https://doc.zentyal.org/en/appendix-c.html#stubs), sino que simplemente habrá que modificar el archivo de configuración de Apache.

1. Modificamos el archivo de configuración `/etc/apache2/sites-available/default-ssl.conf`:

    ```text
    ## Custom certificates issued on 18-02-2023 by Daniel
    #SSLCertificateFile	/etc/ssl/certs/ssl-cert-snakeoil.pem
    #SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
    SSLCertificateFile  /etc/letsencrypt/live/mail.icecrown.es/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/mail.icecrown.es/privkey.pem
    ```

2. Opcionalmente, también añadiré los siguientes parámetros de configuración al final del archivo de configuración. Los valores de los parámetros han sido generados desde [esta](https://ssl-config.mozilla.org/#server=apache&version=2.4.41&config=intermediate&openssl=1.1.1k&hsts=false&ocsp=false&guideline=5.6) página web.

    ```text
    ## Custom configuration applied on 18-02-2023 by Daniel
    ## https://ssl-config.mozilla.org/#server=apache&version=2.4.41&config=intermediate&openssl=1.1.1k&hsts=false&ocsp=false&guideline=5.6
    SSLProtocol             all -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite          ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
    SSLHonorCipherOrder     off
    SSLSessionTickets       off
    ```

3. Reiniciamos el servicio de Apache:

    ```sh
    sudo systemctl restart apache2
    ```

4. Finalmente, accedemos al webadmin para confirmar que el certificado es correcto:

    ![Webmail certificate verification](assets/images/zentyal/certificate-webmail.png "Webmail certificate verification")

### Mail

Para este módulo habrá que modificar dos plantillas ([stubs](https://doc.zentyal.org/en/appendix-c.html#stubs)), una para el servicio SMTP (Postfix) y otra para IMAP/POP3 (Dovecot):

* main.cf.mas
* dovecot.conf.mas

Las acciones a realizar son:

1. Creamos el directorio donde ubicaremos las plantillas:

    ```bash
    sudo mkdir -vp /etc/zentyal/stubs/mail
    ```

2. Copiaremos las plantillas a modificar:

    ```bash
    sudo sudo cp -v /usr/share/zentyal/stubs/mail/{main.cf,dovecot.conf}.mas /etc/zentyal/stubs/mail/
    ```

3. Modificamos la plantilla `main.cf.mas` para el servicio Postfix (SMTP):

    ```text
    ## Custom certificates issued on 18-02-2023 by Daniel
    # smtpd_tls_key_file  = <% $keyFile  %>
    # smtpd_tls_cert_file = <% $certFile %>
    smtpd_tls_key_file  = /etc/letsencrypt/live/mail.icecrown.es/privkey.pem
    smtpd_tls_cert_file = /etc/letsencrypt/live/mail.icecrown.es/fullchain.pem
    ```

4. Modificamos la otra plantilla `dovecot.conf.mas` para el servicio Dovecot (IMAP/POP3):

    ```text
    ## Custom certificates issued on 18-02-2023 by Daniel
    # ssl_cert =</etc/dovecot/private/dovecot.pem
    # ssl_key =</etc/dovecot/private/dovecot.pem
    ssl_key  =</etc/letsencrypt/live/mail.icecrown.es/privkey.pem
    ssl_cert =</etc/letsencrypt/live/mail.icecrown.es/fullchain.pem
    ```

5. Reiniciamos el módulo de correo para que se apliquen los cambios:

    ```sh
    sudo zs mail restart
    ```

6. Finalmente, confirmamos que ambos servicios están usando correctamente el nuevo certificado. Para realizar dicha acción, podremos usar cliente de correo como Thunderbird o el comando `openssl` como será mi caso:

    ```bash
    openssl s_client -starttls smtp -showcerts -connect mail.icecrown.es:465 -servername mail.icecrown.es
    openssl s_client -showcerts -connect mail.icecrown.es:993 -servername mail.icecrown.es
    ```

    El resultado obtenido en mi caso ha sido:

    ```text
    ## Para SMTP
    CONNECTED(00000003)
    depth=2 C = US, O = Internet Security Research Group, CN = ISRG Root X1
    verify return:1
    depth=1 C = US, O = Let's Encrypt, CN = R3
    verify return:1
    depth=0 CN = mail.icecrown.es
    verify return:1
    ---
    Certificate chain
    0 s:CN = mail.icecrown.es
    i:C = US, O = Let's Encrypt, CN = R3


    ## Para IMAP
    CONNECTED(00000003)
    depth=2 C = US, O = Internet Security Research Group, CN = ISRG Root X1
    verify return:1
    depth=1 C = US, O = Let's Encrypt, CN = R3
    verify return:1
    depth=0 CN = mail.icecrown.es
    verify return:1
    ---
    Certificate chain
    0 s:CN = mail.icecrown.es
    i:C = US, O = Let's Encrypt, CN = R3
    ```
