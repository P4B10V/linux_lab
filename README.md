Hola, soy Pablo.

Este laboratorio tiene como objetivo aprender sobre Linux y administración de sistemas.

Al principio empecé practicando con servicios aislados como SSH, nftables, VPN, DNS, DHCP... Más adelante evolucioné el proyecto hacia algo más elaborado, utilizando una máquina virtual con OPNsense actuando como firewall y una red interna en la que configuré dos máquinas virtuales: una actuando como controlador de dominio mediante Samba y otra como servidor de ficheros utilizando NFS.

Gracias a esta etapa aprendí conceptos relacionados con la autenticación centralizada mediante SSSD, así como el montaje automático de directorios utilizando PAM y autofs.

Después de experimentar con Ansible, decidí orientar el laboratorio hacia un escenario más realista: la infraestructura de un instituto.

La base de la infraestructura seguirá siendo similar a la que ya tenía. Todo comenzará con un firewall OPNsense y, detrás de él, una red interna que representará la red del centro educativo. De momento dejaré de lado la segmentación de red para centrarme en la propia infraestructura.

Dentro de la red interna utilizaré un controlador de dominio basado en Samba y un servidor dedicado exclusivamente al almacenamiento de ficheros. Una vez desplegada la infraestructura y configurados los permisos mediante ACL, el siguiente objetivo será automatizar tareas habituales mediante Ansible, como la creación de alumnos, los cambios de curso o la gestión de recursos del centro.

El objetivo principal no es únicamente aprender a configurar servicios, sino también comprender cómo diseñar, documentar y automatizar una infraestructura completa utilizando herramientas habituales en entornos Linux.

