# windows-server-lab

Mon lab Windows Server, monté dans une VM sur Proxmox. Le but est de comprendre
comment marche un domaine Windows en entreprise : l'annuaire, les stratégies de
groupe et le déploiement de postes par le réseau.

Tout ce qui est ici est anonymisé : le domaine, les noms et les adresses sont des
exemples à remplacer par les tiens.

## Ce que j'ai mis en place

- un contrôleur de domaine Active Directory, avec DNS intégré
- des OU, des groupes et des utilisateurs créés en PowerShell
- quelques GPO de base (sécurité, restrictions, environnement utilisateur)
- WDS pour installer des postes en PXE, sur un réseau isolé dans Proxmox pour ne
  pas gêner le reste du réseau

## Procédures

Dans l'ordre :

1. [La VM sur Proxmox](procedures/01-vm-proxmox.md) : réglages de la VM, pilotes VirtIO, agent invité.
2. [Active Directory](procedures/02-ad-ds.md) : promotion en contrôleur de domaine, OU, groupes, utilisateurs.
3. [GPO](procedures/03-gpo.md) : créer, lier et vérifier des stratégies.
4. [WDS et PXE](procedures/04-wds-pxe.md) : réseau isolé, DHCP, images, premier déploiement.

La [checklist](procedures/checklist.md) reprend tout ça en cases à cocher.

## Exemples utilisés

| Élément | Valeur d'exemple |
| :-- | :-- |
| Domaine | `ad.exemple.fr` (NetBIOS `LAB`) |
| Contrôleur de domaine | `DC01`, 192.168.10.10 |
| Réseau de déploiement isolé | 192.168.50.0/24, sans passerelle |

J'utilise un sous-domaine d'un vrai nom de domaine plutôt qu'un `.local`, c'est ce
que Microsoft recommande.
