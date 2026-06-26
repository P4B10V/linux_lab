Después de configurar el controlador de dominio, pasaré con el servidor encargado de los ficheros. Para ello simplemente haré una clonación enlazada del Debian 13 CLI que preparé para el rol de DC antes de instalar nada, configurarle una IP estática, el hostname y el resolver.

Tras realizar comprobaciones con `nslookup` y que podemos realizar queries a nuestro controlador de dominio y resuelve correctamente, lo primero será unir nuestro servidor al dominio, si recordamos tenemos la UO=Servidores dentro de UO=IESLinux.

Como estoy siguiento la [documentación](https://wiki.samba.org/index.php/Setting_up_Samba_as_a_Domain_Member) de Samba, empezaré preparando el servidor según el orden que me recomiendan, por lo que primero instalare `krb5-user` y configuraré `chrony`.



<p align="center">
  <img src="https://github.com/user-attachments/assets/5b3c69f9-6393-4df0-862b-fb8c1f75674f" width="600">
</p>
<p align="center"><em>Verificando Kerberos</em></p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/8e33d88f-d576-4af7-a9f6-4614d402c4e4" width="600">
</p>
<p align="center"><em>Verificando Chrony</em></p>


apt install samba-common-bin winbind libnss-winbind libpam-winbind smbclient krb5-user acl -y


```
[global]
   workgroup = AD
   security = ads
   realm = AD.PVAZQUEZ.NET
   idmap config * : backend = tdb
   idmap config * : range = 3000-7999
   idmap config AD : backend = rfc2307
   idmap config AD : range = 10000-999999
   idmap config AD : ldap_server = ad
   winbind use default domain = yes
   winbind nss info = rfc2307
   template shell = /bin/bash
```


net ads join -U Administrator


<p align="center">
  <img src="https://github.com/user-attachments/assets/1d8c4d39-afad-45bf-99a6-e4f01f4b7e23" width="772" height="37" >
</p>

--- 

Ahora es momento de preparar nuestro sistema de almacenamiento antes de compartir nada. Creé dos discos duros virtuales a mayores en FS1 y ahora lo que haré será configurar un RAID1 + LVM.

Primero compruebo que los discos existan con `lsblk`. Tengo para utilizar **/dev/sdb** **/dev/sdc**

Crear una partición primaria en cada uno con `fsdisk` configurando tipo de particion como **Linux raid auto**

- mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
- pvcreate /dev/md0
- vgcreate vg_data /dev/md0
- lvcreate -l 100%FREE --name lv_data vg_data

<p align="center">
  <img width="428" height="150" alt="imagen" src="https://github.com/user-attachments/assets/8bdb1486-9274-4ebd-af9c-294a71a12d8b" />
</p>

Ahora que ya tengo el espacio para mis carpetas, tendré que crearlas y compartirlas. Voy a definir la siguiente estructura:
```
.
├── antiguos
├── cursos
│   ├── 1asir
│   │   ├── FDH
│   │   ├── ISO
│   │   ├── LM
│   │   ├── PAR
│   │   └── XBD
│   ├── 1dam
│   ├── 1daw
│   ├── 2asir
│   │   ├── ABD
│   │   ├── ASO
│   │   ├── IAW
│   │   ├── SAD
│   │   └── SRI
│   ├── 2dam
│   └── 2daw
└── usuarios
    ├── alumnos
    │   ├── 1asir
    │   ├── 1dam
    │   ├── 1daw
    │   ├── 2asir
    │   ├── 2dam
    │   └── 2daw
    └── profesores
```

- /mnt/datos/usuarios para las carpetas personales de alumnos/profesores
- /mnt/datos/cursos para información relacionada con cada curso y sus asignaturas
- /mnt/datos/antiguos para alumnos que se dan de baja o terminaron estudios

Como ya tenemos grupos creados en Samba, tendrémos que usar acl para proporcionar accesos. 

h






