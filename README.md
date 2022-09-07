# Skripte zur Automatisierung der Einrichtung des VB365 Restore Portals

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

## Aktivieren des Wiederherstellungsportal von Veeam Backup für Microsoft 365 für einen anderen Tenant

Es gibt einige Dinge, die wir vorbereiten müssen, bevor wir das Wiederherstellungsportal auf verschiedenen Tenants aktivieren. 
Wir die Restore Portal Application ID kennen, die wir im Hauptmenü - Allgemeine Optionen - finden können:

![grafik](https://user-images.githubusercontent.com/106468699/188847429-c5cd1a57-53eb-411b-9ff0-20e784af0d9a.png)

Wechseln wir nun zur Registerkarte "Portal wiederherstellen". Dort finden wir die Anwendungs-ID, die wir kopieren, damit wir sie in den nächsten Schritten verwenden können:

![grafik](https://user-images.githubusercontent.com/106468699/188865081-45f543cb-9313-4a09-9a24-254667b11f0c.png)

Aktivieren des Wiederherstellungsportals für einen anderen Tenant

Bleiben wir bei der PowerShell und holen wir uns die Anmeldeinformationen eines Dienstkontos aus dem Tenant, das über genügend Rechte verfügt, um eine Unternehmensanwendung in Azure AD zu installieren, also holen wir uns wie gesagt die Anmeldeinformationen:

```
$Credential = Get-Credential
```

https://jorgedelacruz.uk/wp-content/uploads/2023/03/vbm365-multitenant-003.jpg

Mit dem nächsten Befehl wird unsere PowerShell mit AzureAD verbunden, wobei die zuvor eingeführten Anmeldeinformationen verwendet werden:

```
Connect-AzureAD -Credential $Credential
```

Wenn alles reibungslos funktioniert hat, sollten wir etwa Folgendes sehen:

![grafik](https://user-images.githubusercontent.com/106468699/188865492-accf5a49-6c75-4028-a529-e0f9608f1181.png)

Und der letzte Zaubertrick, ein einziger Befehl, der alles zusammenführt:

```
New-AzureADServicePrincipal -AppId "THEIDFROMYOURRESTOREPORTAL"
```

Wenn alles wie erwartet funktioniert hat, sollte die Ausgabe in etwa so aussehen:

![grafik](https://user-images.githubusercontent.com/106468699/188865688-32044109-6c2b-4667-9cf5-159c60e92fce.png)

Letzter Schritt: Erteilen von Berechtigungen für die neue Anwendung in Azure AD

![grafik](https://user-images.githubusercontent.com/106468699/188866241-800422b7-6d9c-4317-b861-9ea1eedc004a.png)

Wenn wir uns bereits in der Unternehmensanwendung befinden, können wir unter "Berechtigungen" auf "Admin-Zustimmung erteilen für" klicken.

![grafik](https://user-images.githubusercontent.com/106468699/188866298-2ab1cd9b-f4b6-4253-a99f-a3f3ac8dc0f0.png)
