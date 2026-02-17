# Investigación

## Análisis de herramientas

### Kerberos

**Kerberos es un protocolo de autenticación que se utiliza para comprobar la identidad de un usuario o un host.**

Un Kerberos es un sistema o enrutador que proporciona una puerta de enlace entre los usuarios e Internet. Por lo tanto, ayuda a evitar que los ciberatacantes accedan a una red privada.

- Autenticación delegada

La autenticación Kerberos admite un mecanismo de delegación que permite que un servicio actúe en nombre de su cliente al conectarse con otros servicios.

- Inicio de sesión único

Con la autenticación Kerberos dentro de un dominio o en un bosque, el usuario o servicio puede tener acceso a los recursos permitidos por los administradores sin varias solicitudes de credenciales.

- Autenticación mutua

Al usar el protocolo Kerberos, una entidad que se encuentra en cualquiera de los dos extremos de una conexión de red puede comprobar que la otra entidad sea quien dice ser.


### Winbind

**Winbind es un componente de la suite SAMBA que otorga servicios de autenticación a través del Name Service Switch, consultando a un controlador de dominio.**

Se utiliza junto con el servicio de nombres para obtener la información de los usuarios y grupos desde un servidor Windows y permitir a Samba autorizar a los usuarios dentro de un servidor.

### SSSD

**El demonio de servicios de seguridad del sistema (SSSD) es un servicio del sistema que permite acceder a directorios remotos y mecanismos de autenticación.**

- El SSSD funciona en dos etapas:
  
  - Conecta al cliente con un proveedor remoto para recuperar la información de identidad y autenticación. 
  
  - Utiliza la información de autenticación obtenida para crear una caché local de usuarios y credenciales en el cliente. 

### Realmd

El sistema realmd proporciona una forma clara y sencilla de descubrir y unir dominios de identidad para lograr una integración directa con el dominio. Configura los servicios subyacentes del sistema Linux, como SSSD o Winbind, para conectarse al dominio.

-  The realmd system supports the following domain types:

   - Microsoft Active Directory
   - Red Hat Enterprise Linux Identity Management 

## Resolución de nombres

### resolv.conf

Es el nombre del archivo que, en la mayoría de los sistemas operativos tipo UNIX, se encarga de configurar parte del sistema de resolución de nombres de dominio. Es un archivo de texto sin formato (texto plano) y su localización en el arbol de directorios suele ser /etc/resolv.conf.

Este archivo es clave para la configuración porque es el que relaciona el dominio pry-jmg.local con la IP 192.168.56.113, siendo imprescindible ya que el protocole Kerberos requiere que el nombre del servidor se resuelva como está configurado en el certificado de seguridad.

### NTP (hora)

La configuración NTP es crítica porque Kerberos utiliza timestamps en sus tickets de seguridad haciendo que si los relojes de las máquinas no vayan sincronizados este rechace las conexiones.

## Estrategia

### Opcion clasica

La opción clásica es la que utiliza Samba + Winbind



## Mapeo de Identidad