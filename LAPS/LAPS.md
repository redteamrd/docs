# Local Administrator Password Solution (LAPS)

Este documento ofrece una breve descripción general y un resume de los pasos fundamentales para implementar **L**ocal **A**dministrator **P**assword **S**olution o sus siglas en ingles LAPS.

## Qué es LAPS?

Es una solución de manejador de contraseñas automática de cuentas locales en máquinas unidos al dominio.
Esta ofrece:
1.	Una contraseña única en cada máquina.
2.	Las contraseñas se generan aleatoriamente.
3.	Las contraseñas se almacenan en el Controlador de Dominio y están protegidas por ACL, por lo que solo el usuario autorizado puede leer o solicitar su restablecimiento.

### Arquitectura:

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/1.png)

### Funcionamiento:
La solución realiza las siguientes tareas:
1.	Comprueba en cada equipo si la contraseña de la cuenta de administrador local ha caducado o no.
2.	Genera una nueva contraseña cuando contraseña anterior expiro o antes de la fecha de expiración.
3.	Cambia la contraseña de la cuenta administrador local.
4.	Reporta la contraseña al Controlador de Dominio, almacenándola en un atributo confidencial con la cuenta de la máquina.
5.	La contraseña puede ser leída desde el Controlador de Dominio por los usuarios que están autorizado.
6.	Los usuarios autorizados pueden forzar el cambio de contraseña.

### Características:

**Seguridad**
1.	Contraseña aleatoria, que cambia automáticamente con regularidad en las máquinas administradas por el dominio.
2.	La contraseña está protegida durante el transporte mediante cifrado Kerberos.
3.	Mitiga efectivamente ataque de Pass-the-hash.
4.	La contraseña en el dominio está protegida por ACL, por lo que el modelo de seguridad granular se puede implementar fácilmente.

**Capacidad**
1.	Parámetros de contraseña configurables {Antigüedad, Complejidad, Longitud}
2.	Capacidad para forzar el restablecimiento de la contraseña por máquina.
3.	Fácil implementación y mínimo espacio requerido.
4.	Modelo de seguridad integrado con DC ACL.

**Requerimientos**

Los requerimientos son:
-   Controlador de Dominio.
    - Windows 2003 SP1 y superior.
-   Equipos gestionados:
    - Windows Vista o superior x86 o x64
    - Windows Server 2003 SP1 y superior
-   Herramientas administrativas
    - .NET Framework 4.0
    - PowerShell 2.0 o superior 


## Instalación
Descargar LAPS desde Microsoft
https://www.microsoft.com/en-us/download/details.aspx?id=46899
> LAPS.x64.msi = Version 64bit.

### Método de Instalación:
**Instalación Manual:** Se debe instalar LAPS.x64.msi en todas las estaciones de máquinas y Servidores.

Doble Click en LAPS.x64.msi

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/2.png)


Click en **Next** 

En esta instalación habilitaremos todas las opciones, click en **next**

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/3.png)

Click en **install**

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/4.png)

Click **Finish**

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/5.png)

Una vez terminada la instalación podemos verificar en **Control Panel > Programs and Features**
![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/6.png)


## Instalación vía Política de Dominio:
Lo primero que debemos es crear una carpeta para que todos las máquinas y servidores del dominio puedan descargar el instalador de LAPS, para ellos iremos a **“C:\Windows\SYSVOL\sysvol\laps.local\scripts\”**

Aquí crearemos una carpeta, la cual llamaremos **“DeployLAPS”** y copiaremos **“LAPS.x64.msi”** en ella.

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p7.png)

Ahora nuestro siguiente paso es crear una política con la cual haremos el despliegue a todos las máquinas y servidores del dominio. 

Para esto iremos a nuestro **“Server Manager > Group Policy Management”** creamos una política con el nombre **"GPO_Deploy_LAPS"** editamos en **"Computer Configuration > Policies > Software Settings"**

Ej:

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p8.png)

Click derecho en **"Software Installation"** Click New > package…

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p9.png)

En este punto buscaremos la ruta donde colocamos el paquete de instalación, pero vía red.

Ej: **“\\garen01\NETLOGON\DeployLAPS\LAPS.x64.msi"**

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p10.png)

Seleccionamos el paquete, click en open.

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p11.png)

## Configuración de LAPS

Para definir quienes tendrán acceso a ver las contraseñas de los administradores locales de todas las máquinas y servidores, se puede brindar de dos maneras **por grupo(s) de dominio o por usuario(s).**

### Actualizar el esquema del dominio.

> Antes de brindar el acceso por grupo(s) o usuario(s) de dominio debemos actualizar el esquema del dominio.

Momentáneamente agregaremos el usuario con el cual trabajaremos al Grupo de Dominio llamado **"Schema Admins". En Active Directory Users and Computers > Find Users, Contacts, and Groups > Find Now**

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p12.png)

Doble click en el grupo **“Schema Admins”** y aquí agregamos a nuestro usuario, en nuestro caso fue **“garen”**

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p13.png)

Una vez agregado el usuario al grupo de dominio “Schema Admins”, procederemos actualizaremos el schema desde dominio via PowerShell.

Abrimos PowerShell con privilegio de administrador y ejecutamos

```
PS> Import-Module AdmPwd.ps
```
![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p14.png)

```
PS> Update-AdmPwdAdSchema
```
![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p15.png)

Al finalizar debemos remover el usuario del Grupo **"Schema Admin"**, para continuar.

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p16.png)

### Tipos de acceso
**Crear grupo(s) de dominio.**

En **Active Directory Users and Computers** en nuestro laboratorio crearemos diferentes grupos tales como:
1.	LAPS_DCAdmin
2.	LAPS_PCAdmin 
3.	LAPS_Servidores
4.	LAPS_Soporte

Estos grupos son lo que tendrán privilegios para ver la contraseña de administrador local:

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p17.png)

## Configuración de permisos.
Para configurar el permiso para grupo(s) o usuario de dominio debemos realizar los siguientes pasos en el Controlador de Dominio. 

    - Acceso de lectura a máquinas y servidores.

Ahora debemos establecer el permiso para que los usuarios puedan leer la contraseña de administrador local, deberemos agregar los atributos "ms-Mcs-AdmPwd" y "ms-Mcs-AdmPwdExpirationTime" con el siguiente comando.

Abrimos PowerShell con privilegio de administrador y ejecutamos
```
PS> Set-AdmPwdComputerSelfPermission -Identity Computadora
```
![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p18.png)
Nota: "Computadora" es la OU raíz en mi controlador de dominio desde el cual tengo todas mi computadora y servidores
 
- Verificación de permisos Generales.
```
PS> Find-AdmPwdExtendedRights -Identity Computadora | Format-Table
```
![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p19.png)

Notaremos que solo {NT AUTHORITY\SYSTEM, LAPS\Domain Admins} tienen privilegios.

### Acceso por grupo(s) de dominio.
```
PS> Set-AdmPwdReadPasswordPermission -Identity "PC" -AllowedPrincipals "LAPS_Soporte"
```
![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p19.png)

```
PS> Set-AdmPwdReadPasswordPermission -Identity "PC" -AllowedPrincipals "LAPS_PCAdmin"
```
![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p20.png)

```
PS> Set-AdmPwdReadPasswordPermission -Identity "PC_ADMINISTRADORES" -AllowedPrincipals "LAPS_PCAdmin"
```
![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p21.png)

```
PS> Set-AdmPwdReadPasswordPermission -Identity "Servidores" -AllowedPrincipals "LAPS_SERVIDORES"
```
![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p22.png)

```
PS> Set-AdmPwdReadPasswordPermission -Identity "Servidores_Dominio" -AllowedPrincipals "LAPS_DCAdmin"
```
![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p23.png)

Verificación de acceso por Grupos.
```
PS> Find-AdmPwdExtendedRights -Identity Computadora | Format-Table
```
![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p24.png)

Con esto volvemos a verificar que permisos tenemos en cada OU la imagen mas arriba muestra ahora cambios en las OU que hemos agregado como en **PC, SERVIDORES, PC_ADMINISTRADORES, SERVIDORES_DOMINIO.**

**Acceso por usuario de dominio**
```
PS> Set-AdmPwdReadPasswordPermission -Identity "PC" -AllowedPrincipals "leona"
```
![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p25.png)

> Nota: recomendamos para implementación a gran escala hacer uso de {Grupo(s) y OU} que dar acceso por usuario puntual.

## Política de Dominio para Configuración LAPS.

Es necesario definir los parámetros para LAPS tales como **Antigüedad, Complejidad, Longitud.** En este laboratorio crearemos dos tipos de Política una para computadora y otra para servidores y dominio.

### Configuración de Política para Computadora
Esta política la llamaremos **"GPO_Computadora_ConfigLAPS"** y editamos en **"Computer Configuration > Policies > Adminitrative Tempalte > LAPS"**

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p26.png)

Doble click   "Enable local admin password management" **– Enable**

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p27.png)

Doble click **"Password Settings"**
1.	Fourteen (14) characters long,
2.	Password Age of 12 days

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p28.png)

### Configuración de Política para Servidores

Esta política la llamaremos **"GPO_Servidores_ConfigLAPS"** y editamos en **"Computer Configuration > Policies > Adminitrative Tempalte > LAPS"**

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p29.png)

Doble click   "Enable local admin password management" **– Enable**

Doble click **"Password Settings"**
1.	Fourteen (20) characters long,
2.	Password Age of 3 days.

![image](https://github.com/redteamrd/docs/blob/main/LAPS/images/p30.png)

> Nota: Si por alguna razón al tratar de crear la política de parámetro no ve LAPS deberá copia ADML y ADMX en SYSVOL desde

Copiar **"AdmPwd.adml"** desde C:\Windows\PolicyDefinitions\en-US\
en C:\Windows\SYSVOL\laps.local\Policies\PolicyDefinitions\en-us\
 
Copiar **"AdmPwd.admx"** desde C:\Windows\PolicyDefinitions\
en C:\Windows\SYSVOL\laps.local\Policies\PolicyDefinitions\

## Comprobación de Configuración.

Si todo lo indicado hasta este punto, se ha realizado tal cual, podremos verificar la contraseña del administrador local desde cualquier maquina(s) o servidor(s) con el siguiente comando desde powershell.

```  
 PS> Get-AdmPwdPassword -ComputerName "WRK01"
```
## Acrónimos

- **ACL:** Access Control List
- **LAPS:**	Local Administrator Password Solution
- **ADMX:**	Configuraciones de directiva basadas en el registro
- **ADML:**	Group Policy Language-Specific Administrative Template
- **OU:** Organizational Unit
- **PASS-THE-HASH:** Es una técnica donde se utiliza el hash del password del usuario para autenticarse en un equipo remoto, sin la necesidad de la contraseña.
- **KERBEROS:** Protocolo de autenticación de redes de ordenador creado por el MIT.


