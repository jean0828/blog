---
title: (ESP) Administracion de cuentas locales en el Directorio Activo con LAPS
date: 2022-05-26 22:20:00 -0500
categories: [security, Active Directory]
tags: [windows,active directory,laps]     # TAG names should always be lowercase
---

En ambientes empresariales es dificil realizar una administracion adecuada de las cuentas locales. Por ello, Microsoft tiene una herramienta gratuita llamada LAPS para administrar automaticamente las credenciales de los equipos que estan en el Directorio Activo. LAPS nos asegura que la contraseña de la cuenta local administradora es distinta en cada equipo.

* Pre-requisitos: 
1. SO cliente - Windows Server 2019, Windows Server 2016, Windows 10, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2, Windows Server 2008, Windows Server 2003, Windows 7, Windows 8, Windows Vista, Windows 8.1
2. Active Directory – Windows Server 2003 SP1 o superior
3. PowerShell 2.0 o superior, .Net Framework 4.0 o superior
4. Tener un ambiente con Directorio Activo. En este caso el dominio se llama: ```lab.local```
5. Cuenta con permisos de Domain Admin

# Plantillas administrativas

Para poder implementar LAPS se debe tener las plantillas administrativas que se pueden descargar [aca](https://docs.microsoft.com/en-us/troubleshoot/windows-client/group-policy/create-and-manage-central-store). En especifico se necesitan la plantilla administrativa llamada ```AdmPwd```

Para este tutorial se creará un repositorio centralizado. Esto es recomendable en ambientes donde se tienen mas de un controldo de dominio.

ir a la ruta ```\\lab.local\SYSVOL\lab.local\Policies``` y crear la carpeta ```PolicyDefinitions```
* copiar el archivo *AdmPwd.admx* en la carpeta ```PolicyDefinitions```
* copiart el archivo *AdmPwd.adml* en la carpeta ```PolicyDefinitions\en-US```

![admpwd](https://i.imgur.com/b07fG2J.png)
![admpwd2](https://i.imgur.com/OeZAz35.png)
# Preparar el ambiente para activar LAPS

## Identificar la OU en la que se activará LAPS

En este tutorial, La OU para activar LAPS se llama servers en el que solo tenemos un equipo.

![server](https://i.imgur.com/Z2GDHLM.png)

## Instalar extension de LAPS en los equipos que se requiere administrar la cuenta local

de este [link](https://www.microsoft.com/en-us/download/details.aspx?id=46899) se descarga el archivo .msi e iniciamos el que se ajuste a nuestra arquitectura.

Para la instalacion solamente necesitamos seguir con las instrucciones que traen por defecto

![inst](https://i.imgur.com/1oVeiDo.png)
![inst2](https://i.imgur.com/vK8cvhG.png)
![inst3](https://i.imgur.com/X51gjhI.png)
![inst4](https://i.imgur.com/aGtxGXX.png)

NOTA: Esto tambien se puede desplegar por medio de una politica en el Directorio Activo

## Actualizar el Schema del Directorio Activo

LAPS usa 2 atributois que por defecto no los tiene:

1. *ms-Mcs-AdmPwd* : Save the administrator password in clear text
2. *ms-Mcs-AdmPwdExpirationTime* : Save the timestamp of password expiration.

Para extender el schema necesitamos instalar primero LAPS pero habilitando solamente la extension de GPO

![inst-da](https://i.imgur.com/PRYH0OH.png)

Posteriormente, se procede a abrir una consola PowerShell en modo administrador y actualizamos el schema de la siguiente mandera:
```
PS C:\Users\testing> Import-module AdmPwd.PS
PS C:\Users\testing> Update-AdmPwdADSchema
```

![schema](https://i.imgur.com/dngLY3c.png)

## Cambiar permisos del objeto Computer

During the password update process, the computer object itself should have permission to write values to ms-Mcs-AdmPwd and ms-Mcs-AdmPwdExpirationTime attributes. To do that we need to grant permissions to SELF built-in account.

To do that, Run command: 
```
PS C:\Users\testing> Set-AdmPwdComputerSelfPermission -OrgUnit Servers
```
![self](https://i.imgur.com/gGV5BH7.png)

# Asignar permisos al grupo para acceso de password

En este caso, se dara acceso para leer la contraseña al grupo llamado *ITStaff*. Para validar quien puede ver las contraseñas ejecutamos:

```
PS C:\Users\testing>   Find-AdmPwdExtendedRights -Identity servers
```
![seepass](https://i.imgur.com/9Pit8K2.png)

encontramos que los domain admins son los unicos que por defecto pueden ver las credenciales

Ahora, adicionamos los privilegios para que el grupo itstaff puedan ver la contraseña

```
PS C:\Users\testing> Set-AdmPwdReadPasswordPermission -Identity "Servers" -AllowedPrincipals "ITStaff"
```

![readpss](https://i.imgur.com/KLSoaPX.png)

# Crear GPO para LAPS

Abrir el *Group Policy Management*, crear una GPO para activar LAPS bajo la OU de Servers y dar en editar

![GPO](https://i.imgur.com/OGMsIk1.png)

ir a la ruta:  Computer Configuration | Administrative Templates | LAPS

![GPO2](https://i.imgur.com/sbEFLma.png)
![GPO3](https://i.imgur.com/e4hxUpL.png)
![GPO4](https://i.imgur.com/6RpULRC.png)
![GPO5](https://i.imgur.com/Vh1tZDD.png)


# Testear LAPS

para acelerar el proceso, se debe actualizar politicas en el servidor con:

```
C:\Users\testing>gpupdate /force
```

![update](https://i.imgur.com/M94PRQm.png)

En el controlador de dominio con cuenta domain admin validar sus permisos:

![laps-da](https://i.imgur.com/HChSbsq.png)

si, ensayamos las credenciales entramos con exito:

![test](https://i.imgur.com/bwyIiTY.png)
![succ](https://i.imgur.com/UrpfEaS.png)

Ahora, si intentamos visualizar el pass con otra cuenta que no tiene acceso (user en este caso) no se ve:

![no-pss](https://i.imgur.com/HpuFolA.png)

y si lo hacemos con una cuenta del grupo ITStaff, comprobamos que el pass se ve:

![pass](https://i.imgur.com/YzvuC9a.png)






