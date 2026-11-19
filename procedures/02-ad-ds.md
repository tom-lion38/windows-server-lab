# 2. Active Directory

## Promotion en contrôleur de domaine

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

Install-ADDSForest `
  -DomainName "ad.exemple.fr" `
  -DomainNetbiosName "LAB" `
  -InstallDns `
  -SafeModeAdministratorPassword (Read-Host -AsSecureString "Mot de passe DSRM")
```

Le serveur redémarre tout seul. Le mot de passe DSRM sert à réparer l'annuaire en
mode restauration : il faut le noter quelque part de sûr, pas dans un fichier du
dépôt.

Vérifications après le redémarrage :

```powershell
Get-ADDomain
dcdiag /q          # n'affiche rien si tout va bien
nslookup ad.exemple.fr
```

## Organisation

Je range tout sous une OU racine, pour ne jamais mélanger mes objets avec les
conteneurs par défaut (`Users`, `Computers`) :

```powershell
$racine = "DC=ad,DC=exemple,DC=fr"
New-ADOrganizationalUnit -Name "Lab" -Path $racine
foreach ($ou in "Utilisateurs", "Groupes", "Postes", "Serveurs") {
    New-ADOrganizationalUnit -Name $ou -Path "OU=Lab,$racine"
}

# Les nouveaux postes joints au domaine arrivent directement dans la bonne OU
redircmp "OU=Postes,OU=Lab,$racine"
```

## Groupes et utilisateurs

Un groupe par service, et les droits se donnent toujours aux groupes, jamais aux
personnes :

```powershell
$groupes = "OU=Groupes,OU=Lab,$racine"
New-ADGroup -Name "GG_Compta" -GroupScope Global -Path $groupes
New-ADGroup -Name "GG_Info"   -GroupScope Global -Path $groupes

New-ADUser -Name "Alice Martin" -SamAccountName "amartin" `
  -UserPrincipalName "amartin@ad.exemple.fr" `
  -Path "OU=Utilisateurs,OU=Lab,$racine" `
  -AccountPassword (Read-Host -AsSecureString "Mot de passe initial") `
  -ChangePasswordAtLogon $true -Enabled $true

Add-ADGroupMember -Identity "GG_Compta" -Members "amartin"
```

Pour créer beaucoup de comptes, le plus simple est un CSV lu par `Import-Csv` et
une boucle sur `New-ADUser`. Ne mets jamais de vrais mots de passe dans le CSV.

## Joindre un poste au domaine

Sur le poste, le DNS doit pointer vers le contrôleur de domaine, sinon il ne
trouve pas le domaine :

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10
Add-Computer -DomainName "ad.exemple.fr" -Restart
```
