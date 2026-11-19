# Checklist du lab Windows Server

## VM
- [ ] ISO Windows Server et ISO VirtIO dans Proxmox
- [ ] VM en q35 + UEFI, disque VirtIO SCSI, carte VirtIO, agent QEMU
- [ ] Pilotes VirtIO et agent installés (`virtio-win-guest-tools.exe`)
- [ ] Nom de machine, IP fixe, mises à jour
- [ ] Instantané avant la promotion

## Active Directory
- [ ] Rôle AD DS installé, forêt créée
- [ ] `dcdiag` sans erreur, DNS du domaine qui répond
- [ ] OU racine du lab et sous-OU
- [ ] `redircmp` vers l'OU des postes
- [ ] Groupes par service, utilisateurs, appartenances

## GPO
- [ ] GPO utilisateurs et ordinateurs créées et liées aux bonnes OU
- [ ] Politique de mots de passe réglée au niveau du domaine
- [ ] `gpresult` vérifié sur un poste

## WDS
- [ ] Pont isolé `vmbr1` sans IP ni passerelle
- [ ] Deuxième carte du serveur sur `vmbr1`, IP fixe
- [ ] DHCP autorisé, étendue liée seulement au réseau isolé
- [ ] WDS initialisé, ports DHCP réglés
- [ ] Image de démarrage et image d'installation ajoutées
- [ ] Une VM installée de bout en bout par le réseau

## Fin
- [ ] Poste de test joint au domaine
- [ ] GPO appliquées sur le poste
- [ ] Doc mise à jour
