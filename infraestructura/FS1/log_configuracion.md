Después de configurar el controlador de dominio, pasaré con el servidor encargado de los ficheros. Para ello simplemente haré una clonación enlazada del Debian 13 CLI que preparé para el rol de DC antes de instalar nada, configurarle una IP estática, el hostname y el resolver.

Tras realizar comprobaciones con `nslookup` y que podemos realizar queries a nuestro controlador de dominio y resuelve correctamente, lo primero será unir nuestro servidor al dominio, si recordamos tenemos la UO=Servidores dentro de UO=IESLinux.

Como estoy siguiento la [documentación](https://wiki.samba.org/index.php/Setting_up_Samba_as_a_Domain_Member) de Samba, empezaré preparando el servidor según el orden que me recomiendan, por lo que primero instalare `krb5-user` y configuraré `chrony`
