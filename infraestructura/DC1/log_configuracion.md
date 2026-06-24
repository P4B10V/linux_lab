Esta es la primera máquina virtual en configurar. Aquí escribiré todo lo relacionado con la configuración de este servidor. 

El servidor lo haré en un Debian 13 CLI. 

Aquí voy a definir el dominio, para este laboratorio usaré: `pvazquez.net`, como es un dominio que tengo comprado pues lo utilizaré por si en un futuro alguna funcion que ahora mismo desconozco lo necesite.

Para la gestión de usuarios utilizaré Samba, que permite un DNS interno. El DHCP de momento lo dejaré en el firewall. 

Al instalar Samba, el reino que configurare será `ad.pvazquez.net`.

Antes de entrar en la instalación de `samba-ad-dc` me gustaría apuntar lo siguiente:

Todavía no lo entiendo en profundidad, pero intentaré explicarlo lo mejor posible. Para compartir recursos, tenemos `NFS` y `SMB`. 

Si nuestro entorno sólo trabajamos con Linux, usaremos el protocolo NFS (Network File System) debido a que se integra mejor con SSSD (System Security Services Daemon), siendo este último el servicio que se encarga de autenticarse contra Active Directory.

Por otro lado, si tenemos la necesidad de compartir carpetas con clientes Windows, normalmente se utiliza el protocolo SMB. En estos entornos es habitual utilizar winbind, ya que está especialmente orientado a la integración entre Samba y Active Directory.

En conclusión, winbind lo podriamos usar igualmente para nuestro entorno Linux, y si en un futuro queremos añadir clientes windows ya estaría, pero como mi objetivo es dejarlo lo mas Linux posible, elegire la opción de `NFS+SSSD`

Esta explicación es una simplificación y la iré corrigiendo a medida que profundice en el funcionamiento de Samba, SSSD y Active Directory.

---

Estaré utilizando las recomendaciones escritas [aqui](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller)

Lo primero será escribir en `etc/hosts` y `/etc/hostname`: dc1.ad.pvazquez.net, este será el hostname de mi controlador de dominio. Como de momento no tengo configurado ningún DNS dejaré como resolver el firewall para descargar los paquetes necesarios. En `etc/network/interfaces` pondré una IP estática para mi servidor. 


Empezamos instalando: `apt install samba-ad-dc`

Después tendremos que realizar un **provision**, que es como la instalación del rol AD en Windows Server, al hacer `samba-tool domain provision` te pide lo siguiente:

- Realm: AD.PVAZQUEZ.NET
- Domain: AD
- Server Role: dc
- DNS backend: SAMBA_INTERNAL
- DNS forwarder IP address: 10.0.0.1

NOTE: Fue necesario borrar `/etc/samba/smb.conf` para realizar el provision correctamente.

Ahora que tenemos configurado OPNsense como nuestro forwarder, es hora de cambiar `/etc/resolv.conf`:
```
search ad.pvazquez.net
nameserver 10.0.0.10
``` 

Copiamos /var/lib/samba/private/krb5.conf en /etc/krb5.conf 

Zona inversa:
samba-tool dns zonecreate ad.pvazquez.net 0.0.10.in-addr.arpa -U Administrator

samba-tool dns add ad.pvazquez.net 0.0.10.in-addr.arpa 10 PTR dc1.ad.pvazquez.net -U Administrator 


--- 

Verificar DNS:

- host -t SRV _ldap._tcp.ad.pvazquez.net.
- host -t SRV _kerberos._udp.ad.pvazquez.net.
- host -t A dc1.ad.pvazquez.net.
- host -t PTR 10.0.0.10

--- 

Verificar kerberos (hace falta `krb5-user`)

- kinit Administrator
- klist


--- 

El último paso sería configurar nuestro controlador de dominio como servidor NTP. Para ello instalaremos `chrony` y añadiremos en `/etc/chrony/chrony.conf` allow 10.0.0.0/24 para permitir las consultas desde nuestra lan. Acordarse de reiniciar el servicio. 

Este paso según la documentación de Samba es muy importante, debido a que Kerberos necesita que las una sincronización en servidor/clientes. De momento lo dejaremos así pues no disponemos de clientes.


---

Ahora que tenemos un active directory mínimo y funcional, empezaremos a diseñar la infraestructura basada en unidades organizativas:

<img width="711" height="601" alt="Diagrama sin título drawio(5)" src="https://github.com/user-attachments/assets/cd4d3cc8-270a-4838-9c94-96f1a4625d1d" />


De momento así es suficiente.

Lo bueno sería montar un script que replicara la misma estructura de UO para si hay que volver a hacer todo, o simplemente añadir cosas a mayores, es decir, tener un control de lo que se hizo.


```
samba-tool create ou add ou=IESLinux

samba-tool create ou add ou=Equipos,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
    samba-tool create ou add ou=Clases,ou=Equipos,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
        samba-tool create ou add ou=1ASIR,ou=Clases,ou=Equipos,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
        samba-tool create ou add ou=2ASIR,ou=Clases,ou=Equipos,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
        samba-tool create ou add ou=1DAW,ou=Clases,ou=Equipos,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
        samba-tool create ou add ou=2DAW,ou=Clases,ou=Equipos,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
        samba-tool create ou add ou=1DAM,ou=Clases,ou=Equipos,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
        samba-tool create ou add ou=2DAM,ou=Clases,ou=Equipos,OU=IESLinux,DC=ad,DC=pvazquez,DC=net

  samba-tool create ou add ou=Servidores,ou=Equipos,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
  samba-tool create ou add ou=Impresoras,ou=Equipos,OU=IESLinux,DC=ad,DC=pvazquez,DC=net

samba-tool create ou add ou=Usuarios,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
    samba-tool create ou add ou=Alumnos,ou=Usuarios,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
        samba-tool create ou add ou=1ASIR,ou=Alumnos,ou=Usuarios,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
        samba-tool create ou add ou=2ASIR,ou=Alumnos,ou=Usuarios,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
        samba-tool create ou add ou=1DAW,ou=Alumnos,ou=Usuarios,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
        samba-tool create ou add ou=2DAW,ou=Alumnos,ou=Usuarios,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
        samba-tool create ou add ou=1DAM,ou=Alumnos,ou=Usuarios,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
        samba-tool create ou add ou=2DAM,ou=Alumnos,ou=Usuarios,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
    samba-tool create ou add ou=Profesores,ou=Usuarios,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
    samba-tool create ou add ou=Administradores,ou=Usuarios,OU=IESLinux,DC=ad,DC=pvazquez,DC=net

samba-tool create ou add ou=Grupos,OU=IESLinux,DC=ad,DC=pvazquez,DC=net
```

Y ahora definir los grupos:

```
samba-tool group add GRP_ALUMNOS --groupou="OU=Grupos,OU=IESLinux"

samba-tool group add GRP_PROFESORES --groupou="OU=Grupos,OU=IESLinux"

samba-tool group add GRP_ADMINISTRADORES --groupou="OU=Grupos,OU=IESLinux"

samba-tool group add GRP_1ASIR_ALUMNOS --groupou="OU=Grupos,OU=IESLinux"
samba-tool group add GRP_1ASIR_PROFESORES --groupou="OU=Grupos,OU=IESLinux"
samba-tool group add GRP_2ASIR_ALUMNOS --groupou="OU=Grupos,OU=IESLinux"
samba-tool group add GRP_2ASIR_PROFESORES --groupou="OU=Grupos,OU=IESLinux"
samba-tool group add GRP_1DAW_ALUMNOS --groupou="OU=Grupos,OU=IESLinux"
samba-tool group add GRP_1DAW_PROFESORES --groupou="OU=Grupos,OU=IESLinux"
samba-tool group add GRP_2DAW_ALUMNOS --groupou="OU=Grupos,OU=IESLinux"
samba-tool group add GRP_2DAW_PROFESORES --groupou="OU=Grupos,OU=IESLinux"
samba-tool group add GRP_1DAM_ALUMNOS --groupou="OU=Grupos,OU=IESLinux"
samba-tool group add GRP_1DAM_PROFESORES --groupou="OU=Grupos,OU=IESLinux"
samba-tool group add GRP_2DAM_ALUMNOS --groupou="OU=Grupos,OU=IESLinux"
samba-tool group add GRP_2DAM_PROFESORES --groupou="OU=Grupos,OU=IESLinux"

samba-tool group addmembers GRP_ALUMNOS GRP_1ASIR_ALUMNOS
samba-tool group addmembers GRP_ALUMNOS GRP_2ASIR_ALUMNOS
samba-tool group addmembers GRP_ALUMNOS GRP_1DAW_ALUMNOS
samba-tool group addmembers GRP_ALUMNOS GRP_2DAW_ALUMNOS
samba-tool group addmembers GRP_ALUMNOS GRP_1DAM_ALUMNOS
samba-tool group addmembers GRP_ALUMNOS GRP_2DAM_ALUMNOS

samba-tool group addmembers GRP_ALUMNOS GRP_1ASIR_PROFESORES
samba-tool group addmembers GRP_ALUMNOS GRP_2ASIR_PROFESORES
samba-tool group addmembers GRP_ALUMNOS GRP_1DAW_PROFESORES
samba-tool group addmembers GRP_ALUMNOS GRP_2DAW_PROFESORES
samba-tool group addmembers GRP_ALUMNOS GRP_1DAM_PROFESORES
samba-tool group addmembers GRP_ALUMNOS GRP_2DAM_PROFESORES

```

De momento aquí finaliza el DC, no voy a configurar nada mas, a partir de aqui voy a crear el servidor de ficheros. Cuando haga mas modificaciones editaré este documento.



