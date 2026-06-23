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



