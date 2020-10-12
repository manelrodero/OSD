# Office Deployment Toolkit

[Office Deployment Tool](https://docs.microsoft.com/deployoffice/overview-office-deployment-tool) (ODT) es una herramienta de línea de comandos que permite descargar y desplegar versiones _Click-to-Run_ de Office, como por ejemplo Microsoft 365 Apps for Enterprise, en los ordenadores de la empresa.

ODT proporciona más control sobre una instalación de Office: puede definir qué productos e idiomas se instalan, cómo deben actualizarse esos productos y si mostrar o no la experiencia de instalación a sus usuarios.

A fecha de hoy, 12 de octubre de 2020, se puede [descargar](https://www.microsoft.com/en-us/download/details.aspx?id=49117) la versión [16.0.12827.20268](https://docs.microsoft.com/en-us/officeupdates/odt-release-history):

```
Version: 16.0.12827.20268
File Name: officedeploymenttool_12827-20268.exe
Date Published: 6/9/2020
File Size: 2.9 MB
```

## Instalación de ODT

Descargar ODT y luego ejecutar el archivo ejecutable autoextraíble, que contiene el ejecutable de la Herramienta de implementación de Office (`setup.exe`) y varios archivos XML de configuración de muestra:

```
configuration-Office2019Enterprise.xml
configuration-Office365-x64.xml
configuration-Office365-x86.xml
setup.exe
```

## Archivos XML de configuración

Se puede utilizar [Office Customization Tool](https://config.office.com/) (OCT) desde **Microsoft 365 Apps Admin Center** para crear la configuración deseada en cada entorno y descargar el archivo XML correspondiente.

Por ejemplo, el siguiente archivo XML de configuración `office-proplus-2019-without-onedrive-sample.xml`:

* despliega **[Office Professional Plus 2019](https://docs.microsoft.com/en-us/deployoffice/office2019/deploy)** en **Spanish**
* incluye las herramientas de corrección en **Catalán**, **Inglés** y **Francés**
* no instala OneDrive ni Skype

```Xml
<Configuration ID="5a954385-0ee9-4669-b719-01977e82d080">
  <Info Description="- Office 2019 ProPlus x64 Spanish (+ corrección CAT, ENG, FRA)&#xA;- Volume License (MAK)&#xA;- Sin OneDrive&#xA;- Sin Skype" />
  <Add OfficeClientEdition="64" Channel="PerpetualVL2019" ForceUpgrade="TRUE">
    <Product ID="ProPlus2019Volume" PIDKEY="NMMKJ-6RK4F-KMJVX-8D9MJ-6MWKP">
      <Language ID="es-es" />
      <ExcludeApp ID="Groove" />
      <ExcludeApp ID="Lync" />
      <ExcludeApp ID="OneDrive" />
    </Product>
    <Product ID="ProofingTools">
      <Language ID="ca-es" />
      <Language ID="en-us" />
      <Language ID="fr-fr" />
    </Product>
  </Add>
  <Property Name="SharedComputerLicensing" Value="0" />
  <Property Name="PinIconsToTaskbar" Value="FALSE" />
  <Property Name="SCLCacheOverride" Value="0" />
  <Property Name="AUTOACTIVATE" Value="1" />
  <Property Name="FORCEAPPSHUTDOWN" Value="FALSE" />
  <Property Name="DeviceBasedLicensing" Value="0" />
  <Updates Enabled="TRUE" />
  <RemoveMSI />
  <AppSettings>
    <Setup Name="Company" Value="Your Company" />
  </AppSettings>
  <Display Level="Full" AcceptEULA="TRUE" />
</Configuration>
```

## Instalación de Office

Una vez se ha descargado ODT y se ha creado el archivo XML de configuración, se puede:

* Descargar Office localmente (en este caso la versión ProPlus 2019):

```
setup.exe /download office-proplus-2019-without-onedrive-sample.xml
```

* Instalarlo de forma desatendida:

```
setup.exe /configure office-proplus-2019-without-onedrive-sample.xml
```

## Activación de la licencia

**Microsoft 365 Apps** se conecta con Office Licensing Service y Activation and Validation Service durante la instalación para [obtener y activar una clave de producto desde Internet](https://docs.microsoft.com/en-us/deployoffice/overview-licensing-activation-microsoft-365-apps).

Si no se puede acceder a Internet en el momento de la instalación, lo más aconsejable es instalar **Office 2019** y [realizar la activación por volumen](https://docs.microsoft.com/en-us/deployoffice/vlactivation/plan-volume-activation-of-office) de la manera tradicional mediante:

* [KMS](https://docs.microsoft.com/es-es/deployoffice/vlactivation/activate-office-by-using-kms)
* [MAK](https://docs.microsoft.com/es-es/deployoffice/vlactivation/activate-office-by-using-mak)
* [Active Directory](https://docs.microsoft.com/en-us/deployoffice/vlactivation/plan-volume-activation-of-office#ad)

En el archivo XML de configuración anterior se ha utilizado la [GVLK de Office Professional Plus 2019](http://woshub.com/configure-kms-server-for-ms-office-2016-volume-activation/) (`NMMKJ-6RK4F-KMJVX-8D9MJ-6MWKP`) para evitar la publicación de nuestra MAK.

## Anexo

* [Office Cloud Policy Service](https://aka.ms/o365clientmgmt) para aplicar políticas de la misma forma que mediante GPO

