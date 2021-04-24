# Autopilot

[Autopilot](https://docs.microsoft.com/en-us/mem/autopilot/windows-autopilot) es una colección de tecnologías que sirven para:

* Configurar y preconfigurar nuevos dispositivos y prepararlos para su uso
* Restablecer, reasignar y recuperar dispositivos ya en uso

## Requerimientos

Para poder realizar este proceso se necesita una [versión soportada de Windows 10 SAC](https://docs.microsoft.com/en-us/windows/release-information/) o un Windows 10 Enterprise LTSC 2019.

Además, se tienen que cumplir ciertos requisitos a nivel de:

* [Red](https://docs.microsoft.com/en-us/mem/autopilot/networking-requirements)
  * Resolución de nombres DNS de Internet
  * Acceso al puerto 80 (HTTP), 443 (HTTPS) y 123 (UDP/NTP)

> **Nota**: en el documento anterior se detallan los diferentes servicios a los que necesita acceder el equipo durante este proceso (DNS, Windows Update, NTP, Delivery Optimization, etc.).

* [Configuración](https://docs.microsoft.com/en-us/mem/autopilot/configuration-requirements)
  * Habilitar Automatic enrollment en Azure AD
  * [Personalizar la imagen del Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/LoginTenantBranding
) (_custom branding_)
  * El primer usuario que inicie sesión en el equipo necesita permisos para unir equipos al Azure AD
  * (Opcional) Habilitar activación por subscripción para subir de Pro a Education

* [Licencias](https://docs.microsoft.com/en-us/mem/autopilot/licensing-requirements)
  * Azure AD Premium
  * Intune

## 1. Añadir dispositivos a Windows Autopilot

> **Nota**: Normalmente será el fabricante (Dell, HP, Lenovo, etc.) quien registre los dispositivos en Windows Autopilot cuando los compramos, pero se pueden [añadir también de forma manual](https://docs.microsoft.com/en-us/mem/autopilot/add-devices) para probar y evaluar el escenario.

Este proceso manual consiste en:

* Recoger la identidad hardware de los dispositivos (_hardware hashes_)
* Subir esta información a Intune usando un fichero CSV

Para poder usar Windows Autopilot se requiere:

* Tener una subscripción a [Intune](https://docs.microsoft.com/en-us/mem/intune/fundamentals/licenses)
* Habilitar [Windows 10 Automatic enrollment](https://docs.microsoft.com/en-us/mem/intune/enrollment/windows-enroll#enable-windows-10-automatic-enrollment)
* Tener una subscripción a [Azure AD Premium](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-get-started-premium)

El registro (_enrollment_) lo puede hacer un usuario que tenga alguno de estos roles:

* Intune Administrator
* Policy and Profile Manager
* Un rol personalizado creadao usando [RBAC](https://docs.microsoft.com/en-us/mem/intune/fundamentals/role-based-access-control)

> **Nota**: Si se está haciendo _co-management_, la recolección de los _hardware hashes_ necesarios para subir a MEM se puede hacer con [MEMCM](https://docs.microsoft.com/en-us/configmgr/comanage/how-to-prepare-win10#windows-autopilot).

### Script Get-WindowsAutoPilotInfo.ps1

[Get-WindowsAutoPilotInfo.ps1](https://www.powershellgallery.com/packages/Get-WindowsAutoPilotInfo) es un script que se puede descargar desde la PowerShell Gallery para obtener la información hardware de un equipo desde Windows 10 (ya sea desde el propio SO en funcionamiento o desde la pantalla de OOBE).

La ejecución habitual consiste en ejecutar el siguiente código para obtener un fichero CSV con el identificador hardware del equipo:

```PowerShell
New-Item -Type Directory -Path "C:\HWID"
Set-Location -Path "C:\HWID"
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Unrestricted
Install-Script -Name Get-WindowsAutoPilotInfo
Get-WindowsAutoPilotInfo.ps1 -OutputFile AutoPilotHWID.csv
```

Este fichero CSV se tiene **importar** desde el **MEM admin center**: `Devices > Windows > Windows enrollment > Devices (under Windows Autopilot Deployment Program > Import`.

De todos modos, lo más cómodo es importarlo directamente desde el propio equipo usando la opción `-Online`:

```PowerShell
Get-WindowsAutoPilotInfo.ps1 -Online
```

> **Nota**: Esta opción requiere autenticarse y aprobar el [**consentimiento**](https://github.com/microsoftgraph/powershell-intune-samples/blob/master/AdminConsent/readme.md) de la aplicación [**Microsoft Intune PowerShell**](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps/menuId/) para el usuario o la organización.

![Get-WindowsAutoPilotInfo.ps1](images/Get-WindowsAutoPilotInfo.png?raw=true "Get-WindowsAutoPilotInfo.ps1")

Una vez importados, desde el [blade de Windows Autopilot devices](https://endpoint.microsoft.com/?ref=AdminCenter#blade/Microsoft_Intune_DeviceSettings/DevicesEnrollmentMenu/windowsEnrollment) se pueden cambiar ciertos atributos de los equipos:

* Device Name
* Group Tag
* User Friendly Name (si se ha asignado el equipo a un usuario)

> **Nota**: si se asigna un equipo a un usuario, su nombre aparecerá en la pantalla de inicio de sesión personalizada.

## 2. Crear un grupo para Autopilot

La creación de un [grupo para Autopilot](https://docs.microsoft.com/en-us/mem/autopilot/enrollment-autopilot) se realiza desde **MEM admin center**: `Groups > New group`:

* Group type: Security
* Group name: `ADM Intune Autopilot`
* Group description
* Membership type: Assigned o Dynamic Device
* Members: añadir el dispositivo importado previamente

![Add device to group](images/AddDeviceGroup.png?raw=true "Add device to group")

También se puede usar una _dynamic query_ para asignar automáticamente los dispositivos de tipo AutoPilot que se han importado mediante el fichero CSV:

```
(device.devicePhysicalIDs -any (_ -contains "[ZTDId]"))
```

## 3. Crear un perfil de Autopilot

La creación de un [perfil de Autopilot](https://docs.microsoft.com/en-us/mem/autopilot/profiles) se realiza desde **MEM admin center**: `Devices > Windows > Windows enrollment > Deployment Profiles > Create Profile > Windows PC`:

* Nombre y descripción
* Definir la experiencia OOBE del usuario

![Autopilot Profile](images/AutopilotProfile.png?raw=true "Autopilot Profile")

* Grupo al que se asigna el perfil: `ADM Intune Autopilot`

> **Nota**: Intune comprueba periódicamente si hay nuevos dispositivos en los grupos asignados y les asigna el perfil. Antes de desplegar un dispositivo **hay que esperar a que se haya asignado el perfil*: `Devices > Windows > Windows enrollment > Devices (under Windows Autopilot Deployment Program`.

![Profile Assigned](images/ProfileAssigned.png?raw=true "Profile Assigned")

Los dispositivos Autopilot que no tienen perfiles asignados se pueden ver desde `Devices > Overview > Enrollment alerts > Unassigned devices`.

El informe del despliegue mediante Autopilot se guarda durante 30 días en `Devices > Monitor > Autopilot deployments`.

## 4. OOBE

Una vez el equipo está registrado en Windows Autopilot y tiene un perfil registrado, se puede reiniciar y realizar el proceso de **OOBE**:

```
shutdown.exe -r -t 0
```

Si todo funciona correctamente, aparecerá el _branding_ que se había configurado. Al introducir un usuario y una contraseña correcta, el equipo se unirá al Azure AD y se configurará para ser gestionado mediante Intune:

![Device Joined](images/DeviceJoined.png?raw=true "Device Joined")

## 5. A continuación...

A partir de aquí, los pasos siguientes son la aplicación de políticas, instalación de aplicaciones, etc. Hay todo un mundo por explorar ;-)

![Application Install](images/IntuneApplications.png?raw=true "Application Install")

## Borrar dispositivos Autopilot

Durante la vida útil de un dispositivo, es probable que se tenga que borrar (por pruebas, por cesión del equipo, etc.)

Si no están registrados en Intune:

* Borrarlos de Windows Autopilot: `Devices > Windows > Windows enrollment > Devices`

Para borrarlos completamente del _tenant_:

* Borrarlos de Windows Autopilot
* [Borrarlos de Intune](https://docs.microsoft.com/en-us/mem/intune/remote-actions/devices-wipe#delete-devices-from-the-intune-portal)
* Borrarlos del Azure AD: `Devices > Azure AD devices`

## Referencias

* [Demonstrate Autopilot deployment](https://docs.microsoft.com/en-us/windows/deployment/windows-autopilot/demonstrate-deployment-on-vm
)
* [Replacement Dell Hardware Bound to Windows Autopilot](https://www.dell.com/support/kbdoc/es-es/000132036/replacement-hardware-bound-to-windows-autopilot)
* [Step by step Autopilot scenarios](https://blog.mindcore.dk/2020/08/step-by-step-autopilot-scenarios.html) (Mattias Melkersen)
* [Azure AD won’t let you delete device objects associated with Windows Autopilot](https://oofhours.com/2020/07/30/azure-ad-wont-let-you-delete-device-objects-associated-with-windows-autopilot/) (Michael Niehaus)
* [Cleanup Windows Autopilot registrations](https://oliverkieselbach.com/2020/01/21/cleanup-windows-autopilot-registrations/) (Oliver Kieselbach): para poder dar de baja los equipos y evitar que se vuelvan a registrar en nuestro _tenant_ si se han cedido a terceros
* [Connect to Microsoft Graph for Intune with Powershell ISE Add-ons](https://www.imab.dk/connect-to-microsoft-graph-for-intune-with-powershell-ise-add-ons-with-a-single-click/) (Martin Bengtsson)
* [Automatically assign Windows AutoPilot deployment profile to Windows AutoPilot devices](https://www.petervanderwoude.nl/post/automatically-assign-windows-autopilot-deployment-profile-to-windows-autopilot-devices/) (Peter van der Woude): para poder ver todos los equipos de tipo Autopilot, los que son de una compra determinada, etc.
