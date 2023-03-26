# Bugfixing

En esta página se explicarán brevemente y se propondrán soluciones a los bugs encontrados a fecha de (2023-02) que afectan al despliegue de Zentyal que hemos realizado en este proyecto.

## Webadmin

A continuación los bugs en el módulo de Webadmin encontrados.

### Saving changes

En la versión `7.0.5` del módulo `zentyal-core` existe [este](https://github.com/zentyal/zentyal/issues/2100) bug que hace que la interfaz de administración se quede siempre con el mensaje *guardando cambios*.

Para solucionar temporalmente la incidencia, hay que realizar lo siguiente:

1. En el archivo `/usr/share/perl5/EBox/WebAdmin.pm` modificamos el método `_daemons` de la línea **99** por el siguiente contenido:

    ```perl
    sub _daemons
    {
        return [
            { name => 'zentyal.webadmin-uwsgi' },
            { name => 'zentyal.webadmin-nginx' }
        ];
    }
    ```

2. Reiniciamos el módulo de Webadmin:

    ```sh
    sudo zs webadmin restart
    ```

Con esto, el bug se solucionará temporalmente.

## Webmail

A continuación los bugs en el módulo de Webmail encontrados.

### IMAPS

La configuración establecida por Zentyal para el protocolo IMAPS en el módulo de Webmail es correcta siempre y cuando se tenga un certificado reconocido. En caso de usar el que está configurado por defecto, al logearnos nos mostrará un error indicando que no puede acceder al buzón del usuario.

Este bug se encuentra en la versión `7.0.0` del paquete `zentyal-sogo`. Podemos ver la versión de nuestro paquete ejecutando el comando: `sudo dpkg -l zentyal-sogo`.

Hay varias soluciones a este problema:

1. Podemos modificar el parámetro de configuración para que permita conexiones certificados inseguros en localhost.
2. Podemos habilitar temporalmente el protocolo `IMAP` desde `Mail -> General`.
3. Podemos usar un certificado reconocido en el módulo de Webmail (Sogo) como se explica en la página `Certificados`.

En caso de que queramos aplicar la primera opción, tendremos que realizar las siguientes acciones:

1. En el archivo `/usr/share/perl5/EBox/SOGo.pm` editamos la línea **265** y establecemos el siguiente contenido:

    ```sh
    my $imapServer = ($mail->imap() ? '127.0.0.1:143' : '"imaps://127.0.0.1:993/?tlsVerifyMode=allowInsecureLocalhost"');
    ```

2. Reiniciamos los módulos: Webmail y Sogo:

    ```sh
    sudo zs webadmin restart
    sudo zs sogo restart
    ```

3. Tratamos de iniciar nuevamente sesión en el Webmail para confirmar la solución.
