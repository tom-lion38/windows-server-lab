# 4. WDS et PXE sur un réseau isolé

Le PXE a besoin d'un serveur DHCP qui répond aux postes qui démarrent. Pour ne pas
mettre un deuxième DHCP sur mon vrai réseau, je fais tout sur un réseau isolé dans
Proxmox : seuls le serveur WDS et les VM à installer y sont branchés.

## Le réseau isolé dans Proxmox

Sur l'hôte, un pont sans adresse IP et sans passerelle, rattaché à une carte
réseau libre (ou à aucune carte si tout se passe entre VM). Dans
`/etc/network/interfaces` :

```
auto vmbr1
iface vmbr1 inet manual
        bridge-ports none
        bridge-stp off
        bridge-fd 0
#Réseau PXE isolé
```

Puis `ifreload -a`. Ensuite :

- le serveur WDS reçoit une deuxième carte réseau sur `vmbr1`
- les VM à installer n'ont qu'une carte, sur `vmbr1`, et démarrent sur le réseau
  en premier (ordre de démarrage dans Options)

## DHCP sur le réseau isolé

Sur le serveur, la deuxième carte prend une IP fixe sans passerelle, par exemple
192.168.50.1/24. Puis :

```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DhcpServerInDC -DnsName "dc01.ad.exemple.fr" -IPAddress 192.168.50.1
Add-DhcpServerv4Scope -Name "PXE" -StartRange 192.168.50.100 -EndRange 192.168.50.150 -SubnetMask 255.255.255.0
Set-DhcpServerv4OptionValue -ScopeId 192.168.50.0 -DnsServer 192.168.50.1
```

Vérifie que l'étendue n'est liée qu'à la carte du réseau isolé
(`Get-DhcpServerv4Binding`), jamais à celle du réseau principal.

## WDS

```powershell
Install-WindowsFeature WDS -IncludeManagementTools

# Dossier des images, sur un disque de données de préférence
wdsutil /Initialize-Server /RemInst:"D:\RemoteInstall"

# DHCP et WDS sur la même machine : WDS ne doit pas écouter le port 67
wdsutil /Set-Server /UseDHCPPorts:No /DHCPOption60:Yes

# Répondre à tous les postes (pratique en lab, à restreindre ailleurs)
wdsutil /Set-Server /AnswerClients:All
```

## Les images

L'ISO de Windows est montée en `E:` dans cet exemple.

```powershell
wdsutil /Add-Image /ImageFile:"E:\sources\boot.wim" /ImageType:Boot
wdsutil /Add-ImageGroup /ImageGroup:"Windows 11"
wdsutil /Add-Image /ImageFile:"E:\sources\install.wim" /ImageType:Install /ImageGroup:"Windows 11"
```

Si l'ISO contient un `install.esd` au lieu d'un `install.wim`, il faut d'abord
extraire l'édition voulue :

```powershell
dism /Get-WimInfo /WimFile:E:\sources\install.esd
dism /Export-Image /SourceImageFile:E:\sources\install.esd /SourceIndex:6 `
     /DestinationImageFile:D:\install.wim /Compress:max /CheckIntegrity
```

Attention : Microsoft a déprécié le démarrage WDS avec le `boot.wim` des ISO
récentes de Windows 11. Si l'image de démarrage est refusée, il faut passer par
une image WinPE faite avec l'ADK de Windows.

## Premier déploiement

Démarre une VM vide sur `vmbr1`. Elle doit obtenir une IP en 192.168.50.x, charger
l'image de démarrage, puis proposer les images d'installation. Si rien ne se passe :

- la VM démarre-t-elle bien en UEFI sur le réseau (et pas sur le disque vide) ?
- le DHCP a-t-il distribué un bail (`Get-DhcpServerv4Lease -ScopeId 192.168.50.0`) ?
- le service WDS tourne-t-il (`Get-Service WDSServer`) ?
- en UEFI avec Secure Boot, l'image de démarrage doit être signée : désactive le
  Secure Boot de la VM le temps du test si besoin.
