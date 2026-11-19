# 1. La VM Windows Server sur Proxmox

## Avant de créer la VM

Il faut deux ISO dans le stockage d'ISO de Proxmox :

- l'ISO de Windows Server (version d'évaluation ou licence)
- l'ISO des pilotes VirtIO (`virtio-win.iso`, publiée par le projet Fedora)

## Réglages de la VM

| Onglet | Réglage | Pourquoi |
| :-- | :-- | :-- |
| Système | Machine `q35`, BIOS `OVMF (UEFI)`, disque EFI, TPM 2.0 | Démarrage UEFI comme sur un vrai serveur récent |
| Système | Agent QEMU coché | Arrêt propre et IP visible dans Proxmox |
| Disques | Bus `SCSI`, contrôleur `VirtIO SCSI single`, cache par défaut, `discard` coché | Meilleures performances, TRIM vers le stockage |
| Processeur | Type `host` | Toutes les instructions du processeur |
| Mémoire | 4 Go minimum, 8 Go pour être tranquille | AD DS et WDS sont légers |
| Réseau | Modèle `VirtIO (paravirtualized)` | Débit correct, pilote dans l'ISO VirtIO |

Ajoute un deuxième lecteur CD avec l'ISO VirtIO.

## Pendant l'installation

L'installeur ne voit pas le disque au début, c'est normal : il n'a pas le pilote
VirtIO SCSI. Clique sur « Charger un pilote » et va chercher dans l'ISO VirtIO le
dossier `vioscsi\<version de Windows>\amd64`.

Choisis l'édition « Expérience de bureau » si tu veux l'interface graphique, sinon
l'édition Core se gère entièrement en PowerShell.

## Après l'installation

Sur l'ISO VirtIO, lance `virtio-win-guest-tools.exe`. Il installe d'un coup les
pilotes restants (réseau, ballooning, etc.) et l'agent QEMU.

Ensuite, en PowerShell administrateur :

```powershell
# Nom de la machine
Rename-Computer -NewName "DC01" -Restart

# IP fixe (adapter le nom de la carte, voir Get-NetAdapter)
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.10 -PrefixLength 24 -DefaultGateway 192.168.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1

# Mises à jour
Install-Module PSWindowsUpdate -Force
Get-WindowsUpdate -Install -AcceptAll -AutoReboot
```

Le DNS pointe sur la machine elle-même parce qu'elle va devenir le serveur DNS du
domaine. Fais un instantané de la VM dans Proxmox à ce moment-là : c'est un bon
point de retour avant la promotion en contrôleur de domaine.
