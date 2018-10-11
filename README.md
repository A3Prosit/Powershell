



# Définition des termes

-UAC

-// Contrôleur de domaine

-Script

-// Windows

-Commutateur

-Batch

-Outil de scripting windows

-Message Box

-Lecture seule

-Instruction

# Analyse du contexte

-Quoi : Régler des paramètres sur plusieurs machines

-Comment : En faisant un script / En utilisant des UAC

-Pourquoi : Pour gagner du temps / Administrer le cybercafé

# Définition de la problématique

-Comment restreindre l'accès aux fichiers

-Comment déployer et exécuter un script sur des machines ayant une config différente

-Comment déployer le script

-Comment paramétrer de nombreux postes en utilisant un script unique

# Définition des contraintes

# Généralisation

Automatisation

# Hypothèses

-UAC permet de restreindre les accès

-Powershell -> Shell (lol) / Ligne de commande / .Net

-Ne fonctionnera pas sur Windows 98 (car dev en 2006)

-Script ne marche que sur la même version de Windows

# Plan d'action

# Études :

## Powershell

un script shell est un fichiers texte clair qu'un shell interprète comme suite de commandes. 
Ils peuvent cependant semmer le chaos


### Syntaxe

créé des variables : %variable% ou $variable

commandes utiles: 
``` 
Get-ChildItem dossier
```
Il est possible de créer ses propres alias grâce à la commande suivante : 
```
Set-Alias –Name nom_alias –Value commande_powershell
```

Get-ChildItem retourne les infos des fichiers dans le dossier donné et retourne les info des subkeys de registre mais pas les keys
Get-ItemProperty retourne tout  

commentaire # 1 ligne et <# #> x lignes

interpreteurs:

### Fonctionnement général

### Extension de fichier
.ps1
module .psn1
config .ps1xml
### Création

**signatures :** pour signer un scrpt on a besoin d'un certiicat normalisé X509 (rfc 2459)

aythenticodede microsoft met à dispo desoutils dédiés à la gestion des signatures numérique dans la cryptoAPI du SDK. ils sont classés en 2 catégories :
la signature de fichiers et contrôle de signatures:
- **MakeCat** construit un ficgier catalogue non signé. il est utilsé par la suite pour comparer le hachage du ficher catalogue aux hachages des différents dossiers pour verif si il y a eu des modifs
-   **SetReg**  Positionne les clefs de la base de registre qui commandent le procédé de vérification de certificat.
-   **SignTool**  Signe numériquement des fichiers et confirme leur signature numérique.

la création, visualisation et gestiondes certifs:
-   **MakeCert**  Crée un certificat X.509.
-   **Cert2SPC**  Crée un certificat d'éditeur de logiciel (SPC, Software Publisher's Certificate).
-   **MakeCTL**  Crée une liste de certificat de confiance (CTL, Certificate Trust List). MakeCTL est également accessible via un assistant GUI.
-   **CertMgr**  Gère les certificats, les CTL, et les listes de certificats révoqués (CRLs, Certificate Revocation Lists).

les certifs doivent être approuvés par une autorité de certif et ils ont une durée de vie limitée
autorités de certif: https://msdn.microsoft.com/en-us/library/ms995347.aspx

on peut faire sans une CA avec les services de certification PKI qui délivrent des certifs

pour auto-signer nos certificats avec makecert
```
&"C:\Program Files\Microsoft Visual Studio 8\SDK\v2.0\Bin\makecert.exe" `
-n "CN=Nous même autorité de certification racine locale" `
-a sha1 -eku 1.3.6.1.5.5.7.3.3 -r -sv LDPowerShell.pvk LDPowerShell.cer -ss Root -sr localMachine
```
note: sur powershell il faut commencer la commande par & si les chemins d'accès contiennent des espaces

Paramètre|Signification
|--|--|
-n|Nom commun, c'est un identificateur unique respectant la norme X500. Ex: /C=CountryCode/O=Organization/OU=OrganizationUnit/CN=CommonName.
-a|Indique l'algorithme de hachage pour l'empreinte.-ekuInsert dans le certificat un OID précisant comment le certificat sera utilisé par une application. Ici il s'agit de la [signature de code](https://docs.microsoft.com/fr-fr/windows/desktop/api/certenroll/nn-certenroll-ix509extensionmsapplicationpolicies)
-r|Crée un certificat 'auto-signé'.
-sv|Emplacement du fichier de clé privée .pvk.
-ss|Nom du  magasin de certificat où sera stocké le certificat créé.
-sr|Emplacement du magasin de certificat dans la registry, soit localMachine (HKLM) ou currentUser (HKCU).

on nous demande ensuite de saisir un mdp pour la clé privée puis on obtient notre fichier pvk
on nous redemande le mdp qu'on viens d'enrer et le fichier .cer est créé


maintenant que l'autorité de certif est créée on va créer un certif
```
&"C:\Program Files\Microsoft Visual Studio 8\SDK\v2.0\Bin\makecert.exe" -pe -n "CN=Nous même certificat pour PowerShell" `
 -ss My -a sha1 -eku 1.3.6.1.5.5.7.3.3 -iv LDPowerShell.pvk -ic LDPowerShell.cer
```
Paramètre |Signification |
|--|--|
|-pe|Marque la clé privée générée comme étant exportable. Cela permet l'inclusion de la clé privée dans le certificat.|
|-n|Nom commun, c'est un identificateur unique respectant la norme X500. Ex:/C=CountryCode/O=Organization/OU=OrganizationUnit/CN=CommonName.|
|-ss|Nom du magasin de certificat où sera stocké le certificat créé.|
|-a|Indique l'algorithme de hachage pour l'empreinte.|
|-eku|Insère dans le certificat un OID précisant comment le certificat sera utilisé par une application. Ici il s'agit de la [signature de code](https://docs.microsoft.com/fr-fr/windows/desktop/api/certenroll/nn-certenroll-ix509extensionmsapplicationpolicies) |
|-iv|Spécifie le fichier de clé privée .pvk de l'émetteur.|
|-ic|Spécifie le fichier de certificat de l'émetteur.|

on peut visualiser le magasin de certif dans:
-certmgr.exe
-IE outil->options internet-> contenu-> certificats
-mmc.exe
get-psprovider

### Exécution

on peut lancer un script powershell à partir d'un .bat, une invite de commande, l'interpréteur powershell, un service une tâchee planifiée ou en double clickant dessus

on doit créer un .ps1

modifier le UAC avec Set-ExecutionPolicy
pour nos propres script il faut passer en remoteSigned ou Unrestricted

on peut ensuite le lancer en ligne de commade ou dans un .bat en entrant
```
powershell scriptPath
```

dans powershell:
```
.\monScript.ps1
```
ou en lançant le .bat via un service ou une tâche planifiée


faire . ./script.ps1 garde les variables
### POO

powershell est basé sur le framework .net et offre une interface entièrement orrienté objet. Le resultat d'une commande n'est pas du texte mais un objet ou collection d'objets avec ses propriétés et méthodes

powershell permet 'lutilisation d'API COM (Component Object Model). ex, pour ouvrir excel : 
```
$xls = New-Object -comObject Excel.Application
``` 

### Pipe

you know it

### Crontab

sch task

### Risques

par défaut powrshell n'exécute pas les fichiers PS1, par défaut ils sont associés au bloc-note 
par défaut la statégie d'exécution est en restricted interdisant l'exécution de tous scripts. On peut modifier la stratégie d'exécution avec l'appelet de commande Set-ExecutionPolicy ou via un modèle d'administration de stratégie de groupe (fichier ADM) fourni par Microsoft

niveaux de restriction: 

- restricted : 
	- ne lance pas les scripts
	- mode interactif seulement
- AllSigned:
	- lance les scripts
	- ils doivent être signés par un parti de confiance
- RemoteSigned:
	- lance les scripts
	- les fichiers téléchargés doivent être signés par un parti de confiance
	- les locaux peuvent être lancés
- Unrestricted:
	- lance les scripts, pas besoin de signature
 
Get-AuthenticodeSignature : examine un script signé numériquement et donne des informations sur la signature (si il est signé, si la signature est intacte, quel certif a été utilisé)

```
Get-AuthenticodeSignature excel.exe | Format-List *
```
en AllSigned il faut des certif numériques de classe III aka un certif avec un code Authenticode de Microsoft. On peut les obtenir auprès d l'infrastructure des clé publiques (PKI) interne de notre entreprise si on en a une ou si on en a acheté une auprès des autorités de certification commerciales (CA), comme CyberTrust, Thawte et VeriSign.

```
Get-ChildItem CERT: -recurse –codeSigningCert
```
montre si des certif sont installés sur notre machine

RemoteSigned offrent une backdoor si les logiciels téléchargés n'ont pas d'indicateur que met nrmalement outlook ou explorer. Elle est cependant limitée car il faudrait téléharger le script, lancer PowerShell et exécuter le script manuellement



## Message Box

## UAC

User Account Control fonctionnalité permettant de prévenir d'un changmeent de config du système Windows (c'est le "voulez vous exécuter ..." )
Comment le désactiver: 

avec le GUI:

C:\Windows\System32\UserAccountControlSettings.exe

et on baisse le curseur 

![](https://syskb.com/wp-content/uploads/2009/12/syskb00029.png)
1
avec les registres:

regedit

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System
mettre Enable UAC à 0

avec les GPO: 

- lancer la Group Policy Management Console (GPMC) avec gpmc.msc
- aller à la racine et créé une GPO "Desactivation UAC" 
- aller dans Computer Configuration > windows settings > security settings > Local Policies > Security Options
- Paramétrez les stratégies suivantes comme indiqué:

	1.  User Account Control: Behavior of the elevation prompt for administrators in Admin Approval Mode **– Elevate without prompting**.
	2.  User Account Control: Detect application installations and prompt for elevation –  **Disabled**.
	3.  User Account Control: Only elevate UIAccess applications that are installed in secure locations –  **Disabled**.
	4.  User Account Control: Run all administrators in Admin Approval Mode –  **Disabled**.
## Langage scripting Windows et Linux (liste)

Bourne: sh de 1979
a donnée :
- ash Almquist shell
- bash bourne again écrit pour GNU, shell par défaut sur la plupart des linux
- dash Debian Almiquist shell remplacement moderne d'ash sur debian et ubuntu
- ksh Korn shell
- pdksh public domain korn shell
- mksh MirBSD Korn shell basé sur ksh et pdksh
- zsh Z shell moderne backward compatible avec bash
- Busybox

 csh C shell 1988 sur OS2 et 1992 windows

powershell
wscript
awk
