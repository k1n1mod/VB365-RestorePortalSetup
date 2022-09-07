﻿# Skripte zur Automatisierung der Einrichtung des VB365 Restore Portals

## Funktion

Eine Sammlung von Skripten zur Automatisierung der Einrichtung und Konfiguration des Veeam Backup for Microsoft 365 Restore Portals. Diese Skripte können für VB365-Umgebungen mit einer einzigen Microsoft 365-Organisation und für solche mit mehreren Microsoft 365-Organisationen in einer mandantenfähigen Einrichtung (wie in einem Service Provider-Szenario) verwendet werden.

## Bekannte Probleme

* *Es gibt derzeit keine Methode zur Automatisierung der Konfiguration eines eigenständigen RESTful API-Servers.*

## Anforderungen

Alle Skripte sind so konzipiert, dass sie auf dem VB365-Server ausgeführt werden.

* Veeam Backup for Microsoft 365 v6
*     Für beide Skripte (Connect-VB365RestorePortal.ps1/New-VB365EnterpriseApplication.ps1) sind die folgenden PowerShell-Module erforderlich:
  * [Azure AD](https://www.powershellgallery.com/packages/AzureAD)
  * [Az.Accounts](https://www.powershellgallery.com/packages/Az.Accounts)

## Verwendung

* Installieren von Azure AD und Az.Accounts


```powershell
Install-Azure.ps1 
```

* Erstellen Sie die Azure Enterprise-Anwendung
  * ***Name:*** Name der zu erstellenden Unternehmensanwendung
  * ***URL:*** URL, die den Kunden für den Zugriff auf das Wiederherstellungsportal zur Verfügung gestellt werden soll

```powershell
New-VB365EnterpriseApplication.ps1 -Name "Veeam Restore Portal" -URL "https://veeam.domain:4443"
```

* Aktivieren des VB365 Wiederherstellungsportals
  * Befolgt die in der Veeam-Dokumentation beschriebenen Schritte [Veeam documentation](https://helpcenter.veeam.com/docs/vbo365/guide/ssp_configuration.html)
  * ***AppId:*** Enterprise Application Id (erstellt im vorherigen Schritt)
  * ***AppThumbprint:*** Zertifikats-Thumbprint, der bei der Erstellung der Unternehmensanwendung verwendet wurde
  * ***SaveCerts:*** (optional) Speichert alle Zertifikate im Skriptordner.
            Dies ist nützlich, wenn Sie die REST-API auf einem anderen Computer konfigurieren.

```powershell
Enable-VB365RestorePortal.ps1 -AppId 37a0f8e1-97bd-4804-ba69-bde1db293273 -AppThumbprint ccf2c168a2a4253532e27dba7e0093d6b6351f93 -SaveCerts
```

* Skript dem Mandanten zur Verfügung stellen, damit dieser eine einmalige Autorisierung für das Wiederherstellungsportal durchführen kann
  * *Dieser Schritt ist nicht erforderlich, wenn nur eine einzelne Microsoft 365-Organisation geschützt werden soll.*
  * Befolgt die in der Veeam-Dokumentation beschriebenen Schritte [Veeam documentation](https://helpcenter.veeam.com/docs/vbo365/guide/ssp_configuration.html)
  * Die Enterprise Application ID muss dem Skript hinzugefügt werden. Auf diese Weise muss der Mieter lediglich das Skript ausführen.
![grafik](https://user-images.githubusercontent.com/106468699/188834105-d5c47594-0ad9-4002-8df2-851d1ea6f5b2.png)

```powershell
Connect-VB365RestorePortal.ps1
```
