# Información del proyecto

El objetivo principal de este proyecto es mostrar y explicar una configuración detallada, robusta, segura y monitorizada del despliegue de un servidor [Zentyal] 7.0 en el proveedor de cloud de Amazon [AWS] para un entorno de producción.

**NOTA:** Es importante tener en cuenta que todo lo explicado en el proyecto es un **ejemplo** real de implementación, que puede usarse como base o guía para el diseño de vuestro propio entorno.

[Zentyal]: https://zentyal.com/
[AWS]: https://aws.amazon.com/es/what-is-aws/

Las funciones que tendrá este servidor será de actuar como servidor de correo para la organización y adicionalmente, como servidor de recursos compartidos para los distintos departamentos.

Finalmente, mencionar que también se realizarán configuraciones adicionales que bajo mi punto de vista mejoran la experiencia en la CLI durante las tareas de mantenimiento o troubleshooting.

## AWS

Como se ha explicado, se usará AWS para alojar el servidor Zentyal. Este servidor tendrá un coste mensual, el cual dependerá de varios factores, como por ejemplo:

* Tipo de servidor.
* Si se contrata la instancia con [Saving Plans].
* Número de volúmenes EBS.
* Tráfico que recibe el servidor.
* Políticas de copias de seguridad.
* Sistema de monitorización.

[Saving plans]: https://aws.amazon.com/es/savingsplans/

Para este proyecto en concreto, se harán uso de los siguientes servicios disponibles de AWS:

* [Route53](https://docs.aws.amazon.com/es_es/Route53/latest/DeveloperGuide/Welcome.html)
* [VPC](https://docs.aws.amazon.com/es_es/vpc/latest/userguide/what-is-amazon-vpc.html)
* [KMS](https://aws.amazon.com/es/kms/)
* [EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/concepts.html)
* [CloudWatch](https://docs.aws.amazon.com/es_es/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
* [ChatBot](https://aws.amazon.com/es/chatbot/)
* [SNS](https://docs.aws.amazon.com/es_es/sns/latest/dg/welcome.html)
* [Config](https://aws.amazon.com/es/config/)

## Zentyal

El servidor Zentyal usará la última versión estable disponible, que a fecha de hoy es 7.0. Además, no se usará una licencia comercial, aunque es altamente recomendable debido a las funcionalidades adicionales que ofrece como por ejemplo, tener la posibilidad de contactar con soporte.

Los módulos que se instalarán y configurarán serán:

* [Network](https://doc.zentyal.org/en/firststeps.html#network-configuration-with-zentyal)
* [Logs](https://doc.zentyal.org/en/logs.html)
* [Firewall](https://doc.zentyal.org/es/firewall.html)
* [Software](https://doc.zentyal.org/es/software.html)
* [NTP](https://doc.zentyal.org/es/ntp.html)
* [DNS](https://doc.zentyal.org/es/dns.html)
* [Controlador de dominio](https://doc.zentyal.org/es/directory.html)
* [Correo](https://doc.zentyal.org/es/mail.html)
* [Webmail](https://doc.zentyal.org/es/mail.html#cliente-de-webmail)
* [Antivirus](https://doc.zentyal.org/es/antivirus.html)
* [Mailfilter](https://doc.zentyal.org/es/mailfilter.html)
* [CA](https://doc.zentyal.org/es/ca.html)
* [OpenVPN](https://doc.zentyal.org/es/vpn.html)

Además, también se realizarán las siguientes configuraciones adicionales:

* Creación de una partición para el SWAP.
* Uso de varios volúmenes EBS para varios puntos de montaje del sistema de archivos.
* Generación de certificados con [Let's Encrypt].
* Fortalecer el servicio de correo con: [SPF], [DKIM] y [DMARC].
* Políticas de seguridad y rotación de contraseñas del dominio.

[Let's Encrypt]: https://letsencrypt.org/es/
[SPF]: https://support.google.com/a/answer/33786?hl=es-419
[DKIM]: https://support.google.com/a/answer/174124?hl=es-419
[DMARC]: https://support.google.com/a/answer/2466580?hl=es

## Requisitos

Para poder implementar o probar los pasos descritos en este proyecto, se requerirá de lo siguiente:

1. Conocimiento en entorno Linux, y más concretamente, en Ubuntu.
2. Conocimientos a nivel de administración de sistemas operativos.
3. Conocimientos en el manejo de la CLI (línea de comandos).
4. (Opcionalmente) Una cuenta en AWS para la creación y configuración de los recursos en caso de querer usarse.

## Consideraciones

A continuación se indican algunas consideraciones a tener en cuenta para poder implementar y/o probar el proyecto:

1. En caso de no tener conocimientos robustos sobre Linux, es recomendable usar la versión comercial, ya que suele venir con acceso a soporte, lo cual puede ser muy útil ante incidencias o actualizaciones de versiones.
2. No es necesario usar AWS como proveedor Cloud, se pueden usar otras soluciones o incluso servidores físicos de la organización.
3. Si se usa un proveedor Cloud, el despliegue tendrá un coste económico mensual.
4. Debido a los módulos instalados en Zentyal, el servidor requerirá de un mínimo de 4GB de RAM.
