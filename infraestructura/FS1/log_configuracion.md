Después de configurar el controlador de dominio, pasaré con el servidor encargado de los ficheros. Para ello simplemente haré una clonación enlazada del Debian 13 CLI que preparé para el rol de DC antes de instalar nada, configurarle una IP estática, el hostname y el resolver.

Tras realizar comprobaciones con `nslookup` y que podemos realizar queries a nuestro controlador de dominio y resuelve correctamente, lo primero será unir nuestro servidor al dominio, si recordamos tenemos la UO=Servidores dentro de UO=IESLinux.

Como estoy siguiento la [documentación](https://wiki.samba.org/index.php/Setting_up_Samba_as_a_Domain_Member) de Samba, empezaré preparando el servidor según el orden que me recomiendan, por lo que primero instalare `krb5-user` y configuraré `chrony`.

<p align="center">
 **Verificando kerberos**
</p>
<p align="center">
 ##Verificando kerberos
  <img src="https://github.com/user-attachments/assets/5b3c69f9-6393-4df0-862b-fb8c1f75674f" width="600">
</p>

<p align="center">
 **Verificando chrony**
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/8e33d88f-d576-4af7-a9f6-4614d402c4e4" width="600">
</p>


