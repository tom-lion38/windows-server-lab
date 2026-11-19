# 3. GPO

Je crée les GPO en PowerShell quand c'est possible, pour pouvoir les refaire à
l'identique. La console `gpmc.msc` reste plus pratique pour les réglages qui n'ont
pas de clé de registre simple.

## Créer et lier une GPO

```powershell
Import-Module GroupPolicy
$ou = "OU=Utilisateurs,OU=Lab,DC=ad,DC=exemple,DC=fr"

New-GPO -Name "U_Restrictions" -Comment "Restrictions de base des utilisateurs"
New-GPLink -Name "U_Restrictions" -Target $ou
```

Je préfixe le nom par `U_` pour les GPO qui touchent les utilisateurs et `C_` pour
celles qui touchent les ordinateurs. Ça évite de se demander où la lier.

## Exemples de réglages

```powershell
# Bloquer le panneau de configuration
Set-GPRegistryValue -Name "U_Restrictions" `
  -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoControlPanel" -Type DWord -Value 1

# Verrouiller la session après 10 minutes d'inactivité
New-GPO -Name "C_Securite" | New-GPLink -Target "OU=Postes,OU=Lab,DC=ad,DC=exemple,DC=fr"
Set-GPRegistryValue -Name "C_Securite" `
  -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System" `
  -ValueName "InactivityTimeoutSecs" -Type DWord -Value 600
```

La politique de mots de passe du domaine (longueur, complexité, verrouillage) ne
se règle pas dans une GPO liée à une OU : elle se règle dans la « Default Domain
Policy », ou avec des stratégies de mot de passe affinées
(`New-ADFineGrainedPasswordPolicy`).

## Vérifier qu'une GPO s'applique

Sur un poste du domaine :

```powershell
gpupdate /force
gpresult /r                       # résumé : quelles GPO sont appliquées ou filtrées
gpresult /h C:\rapport-gpo.html   # rapport complet lisible dans un navigateur
```

Si une GPO n'apparaît pas, je vérifie dans l'ordre : l'objet est-il dans la bonne
OU, la GPO est-elle liée et activée, le filtrage de sécurité contient-il bien
« Utilisateurs authentifiés », et le poste résout-il bien le domaine en DNS.
