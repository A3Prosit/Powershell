
# Powershell



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

- Powershell

	- Syntaxe
	- Fonctionnement général
	- Extension de fichier
	- Création
	- Exécution
	- POO
	- Pipe
	- Crontab

# POWERSHELL
- Scripts shell
- Fichiers en texte clair interprété par un shell de commande
- Chaque ligne est exécutée dans l'ordre


Pour :
- Pratique
mais
- Peut faire des ravages si fait à la va vite

**10 Conseils**
- Préférer les variables (%) (ex : SystemRoot qui pointe vers le repertoire d'instalation)
- Utiliser 'set' pour voir la liste de variables environnement

**Paramètres de sécurité initiaux**
- N'exécutera pas les fichiers .PS1 si on double clique dessus (bloc note par défaut)
- Stratégie d’exécution est définit sur *"Restricted"* dès l'installation (interdit exécution, permet importer quelques XML signés par Microsoft, donc empêche nos propres scripts au démarrage)
	- Décrit les conditions dans lesquelles un script s’exécutera
	- Modifiable avec 'Set-ExecutionPolicy' ou fichier AMD (fichier microsoft)
	- Get-AuthenticodeSignature pour examiner des scripts
````
PS C:\Program Files\Microsoft Office\Office12>
Get-AuthenticodeSignature excel.exe | Format-List *
````

- Stratégie la plus basse "Unrestricted" (sans restrictions, fortement déconseillé)
- Stratégie la plus sécurisée "Allsigned" (exécute que avec signature avec certificat approuvé par microsoft)
	- Classe III, obtenu près de l'infrastructure de clé publique (PKI) de mon entreprise à acheter auprès des autorisations de certification commerciale (CA) comme CyberTrust / Thawte ou VeriSign
- Stratégie "RemoteSigned" (Fichiers locaux sans signature s'éxecutent, télechargés non [outlook..])
	- On peut quand même lancer un script manuellement téléchargé (que les infos' le savent)
	- /!\Existe une porte dérobée avec un script basé sur les profils

## UAC : 

- User Account Control : Fonctionnalité permettant de prévenir d'un changement de config sous Windows. "Voulez-vous vraiment exécuter ce programme windows" (popup')
- Modifiable depuis cmd : `C:\Windows\System32\UserAccountControlSettings.exe`
- Modifiable depuis regedit | registre :
``HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System``
- Modifiable depuis un GPO de domaine (ensemble machines d'un AD) :
	- depuis racine de domaine, GPO ⇒ Désactivation UAC

## Message Box Function :
- Fenêtre popup en PowerShell codable

## Get-ItemProperty Cmdlet
- Get-ChildItem c:\scripts ⇒ Retourne tous les fichiers dans le dossier scripts.
- Permet aussi d'énumérer toutes les clés de registres d'une application :
```
Get-ItemProperty "hklm:\software\microsoft\windows\currentversion\uninstall\windows media player"
```
```
DisplayName       : Windows Media Player 10
UninstallString   : "C:\Program Files\Windows Media Player\Setup_wm.exe" /Uninstall
DisplayIcon       : C:\Program Files\Windows Media Player\wmplayer.exe
ParentKeyName     : OperatingSystem
ParentDisplayName : Windows Updates
```

## Add-Type :

- Permet de définir des classes .NET Framework dans une session Windows Powershell
	- Instancier des objets avec 'New-Object' et les utiliser
	- Si on l'ajoute au profil Windows PowerShell, la classe sera disponible sur toutes les sessions
	- On peut indiquer un type, spécifier des méthodes...
	- Le type est définit par un "assembly" ou un code source (donc on le crée)
Paramètres :
````
- AssembyName (string) : Nom de l'assembly - Obligatoire
- CodeDomProvider : Compilateur - Optionnel
- CompilerParameters : Options du compilateur - Optionnel
- IgnoreWarnings : Ignorer les avertissements du compilateur (not an error) - Optionnel
- Language : C# par défaut, pour sélectionner le compilateur de code - Optionnel
- MemberDefinition (String) : Nouvelles propriétés ou méthodes pour la classe - Obligatoire
- Name (String) : Nom de la classe à créer - Obligatoire
- Namespace (String) : Espace de nom pour le type. (sinon dans espace de noms de M.PowerShell) - Opt
- OutputAssembly (String) : Génère un DLL avec le nom spécifié dans l'emplacement - Opt
- OutputType : Le type de sortie de l'assembly - Opt
- PassThru : Retourne un objet, qui représente les types ajoutés (informatif) - Opt
- Path (String) : Chemin vers code source ou DLL contenant les types - Obligatoire
- ReferencedAssemblies (String) : Par défaut, Assemblys référence à System.dll, on peut le changer - Opt
- TypeDefinition (String) : Code source qui contient les déf. de types - Obligatoire
 - UsingNamespace (String) : Spécifie les espaces de noms pour la classe (System par défaut) - Opt
 - CommonParameters : Prend en charge les paramètres courants comme ErrorAction... 
````
⇒ On ne peut pas mettre d'entrée objet vers Add-Type
⇒ On ne peut pas mettre de sorties, sauf System.RuntimeType


## Exécuter un script powershell :
- Exécutable depuis un .bat
- Exécutable depuis un terminal
- Exécutable depuis l’interpréteur PowerShell
- Exécutable depuis un .ps1

## Corbeille exercice : 
- *Get-Help* [nom-commande]
- *Get-Command :* Obtenir les commandes
- *Get-Item C:\** ⇒ Obtient l'élément à l'emplacement C:\
- *Get-item C:\ | Get-Member* : Propriétés et méthodes associés
- *(Get-Item C:\).GetFiles()* : fichiers contenus dans C
- *(Get-Item C:\).CreationTime* : Date de création 
- *Get-ChildItem -Path C:\** -Force : Afficher répertoires
- *$test = new-object -typename System.String -argumentlist "ceci est une chaîne de caractères"* : Créer l'objet 'test'
- *$test |Get-Member :* Permet de connaître les commandes liées à notre objet $test
- *$xls = New-Object -comObject Excel.Application : créer une instance d'excel (API COM)*
- *$xls | Get-Member :* Consulter toutes ses commandes
- *$xls.Visible = $true :*  L'ouvrir et l'afficher
- *$xls.WorkBooks.Add() :* Initialiser la grille
- *$xls.ActiveWorkBook.SaveAs("C:\essai.xls")* : Sauvegarder
- *Get-WMIObject –List* : Afficher la liste des objets VMI (systeme de gestion interne de windows - Surveillance / contrôle...)
- *Get-WMIObject -Class Win32_IP4RouteTable |Format-Table -Property Destination, Mask, NextHop* : Afficher la table de routage IPV4, dont "Format-Table" pour afficher sous forme de table.
- *cls ou clear-host* : 
- *Set-ExecutionPolicy RemoteSigned :* Modifier la configuration de PowerShell
- *Get-ExecutionPolicy :* Connaître la configuration de PowerShell
- *Get-Command | Out-File -FilePath C:\temp\fichier.txt :* Récupère la liste des commandes, et envoie la liste dans le fichier.txt
- *Get-Command | foreach {write-host "$_ est une commande de PowerShell"} :* Récupère la liste des commandes, et pour chaque write-host, en rajoutant la chaine juste après, l'affichant sur un terminal
- *Get-Command | Foreach {Get-Help $_ -detailed |Out-File -FilePath C:\temp\$_.txt –Encoding ASCII}* : Récupère la liste des commandes, pour chaque commande, on va obtenir l'aide détaillée, et on va créer un fichier du nom de la commande en .txt qui va contenir l'aide, avec un encodage ASCII)
- *Get-ChildItem C:\temp | ForEach-Object {$_.Get_extension().toLower()} |Sort-Object | Get-Unique| Out-File –FilePath C:\temp\extensions.txt -Encoding ASCII :* On récupère la liste des répertoires, pour chaque objet du répertoire, l'extension (en minuscule), et trié, et pas en double, on va créer un fichier extension.txt qui contient les extensions).
- 
