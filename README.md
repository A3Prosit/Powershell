
# Powershell



# Définition des termes

-UAC : User Account Control

-// Contrôleur de domaine

-Script : Programme en langage interprété, qui permet de manipuler les fonctionnalités d'un système informatique.

-// Windows

-Commutateur : Switch

-Batch : Traitement Batch ⇒ Enchaînement automatique d'une suite de commande (processus) sur un ordi sans intervention d'un opérateur. Un .BAT est l'extension d'un fichier de commande MS-DOS, permettant d'exécuter du .EXE ou du .COM.

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

- Successeur de Windows NT (1993) (cmd.exe)
- Successeur de MS-DOS de Windows 98 (command.com)
- Scripts shell (Interpréteur de commande)
- Série commandes écrites dans un fichier
- Chaque ligne est exécutée dans l'ordre
- Va s'appuyer sur le framework *Microsoft .Net*


## Structure & commandes 

- Structure des commandes :
![](https://www.it-connect.fr/wp-content-itc/uploads/2014/12/cmdlet-550x181.png)

- Variables :
	- Emplacement de stockage provisoire
	- Variables utilisent $
	- Variables d'environnement : $env:[NomVariableEnvironnement]
	- Repose sur le principe de la POO (Classe / Instance / Collection / Propriétés / Méthodes...)
		- Utiliser un point pour accéder à l'attribut d'un objet, ou une de ses méthodes.
	- **Conseils variables :**
		- Préférer les variables (%) (ex : SystemRoot qui pointe vers le repertoire d'instalation)
		- Utiliser 'set' pour voir la liste de variables environnement

- Les | : Permet d'utiliser la sortie d'une commande, pour la passer en entrée d'une autre commande.
![](https://www.it-connect.fr/wp-content-itc/uploads/2014/12/pipelinebis-550x300.png)


## Commandes de base :
- Get-Help : Obtenir de l'information sur une commande
- Get-Command : Obtenir les commandes disponibles
- Get-Member : Récupérer les propriétés et méthodes d'un objet. 
- Get-ItemProperty : Obtenir les propriétés d'un item (dernier accès... & utiliser registre)
- Get-ChildItem : Retourne les items et ses enfants sur un Path
	- Retourne tous les fichier (childs) et fichiers sous-jacents depuis un Path
- **Message Box function** : Fenêtre popup en PowerShell programmable et configurable
- **User Account Control** : Fonctionnalité permettant de prévenir d'un changement de config sous Windows. "Voulez-vous vraiment exécuter ce programme windows" (popup')
	- Modifiable depuis cmd : `C:\Windows\System32\UserAccountControlSettings.exe`
	- Modifiable depuis regedit | registre :
``HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System``
	- Modifiable depuis un GPO de domaine (ensemble machines d'un AD) :
- Add-Type : Ajouter une classe Microsoft.NET à la session Powershell
	- Instancier des objets avec 'New-Object' et les utiliser
	- Si on l'ajoute au profil Windows PowerShell, la classe sera disponible sur toutes les sessions
	- On peut indiquer un type, spécifier des méthodes...

## Exécuter un script powershell :
- Exécutable depuis un .bat
- Exécutable depuis un .ps1
- Exécutable depuis un terminal
- Exécutable depuis l’interpréteur PowerShell
Principe du dot-sourcings :
- .  .\exemple.ps1⇒ Exécuter un script et afficher le résultat console
	- Permet de réutiliser les variables qui auraient été détruites

## Les stratégies d'exécutions :

- **Restricted** :
	- Stratégie par défaut lors de l'installation Windows
	- N'exécutera pas les fichiers .PS1 si on double clique dessus (bloc note par défaut)
	- interdit exécution, permet importer quelques XML signés par Microsoft, donc empêche nos propres scripts au démarrage
- **Unrestricted**
	-  Stratégie sans restrictions, fortement déconseillé
- **Allsigned**
	- Stratégie la plus sécurisée
	- Exécute que signature avec certificats approuvés par microsoft
	- Obtenu près de l'Infra de la clé publique (PKI) de l'entreprise
	- A acheter auprès des autorisations de certification commerciale (CA) {CyberTrust / Thawte ...}
- **RemoteSigned**
	- Fichiers Locaux sans signature s'exécutent
	- Les scripts télécharges depuis navigateur / outlook... : Non
	- MAIS :
		- On peut quand même lancer un script manuellement téléchargé (Domaine de l'info)
		- /!\ Comporte des failles, dont une porte dérobée avec script basé sur les profils

**Pour les modifier :**  
	- Modifiable avec 'Set-ExecutionPolicy' ou fichier AMD (fichier microsoft)
	- Get-AuthenticodeSignature pour examiner des scripts
````
PS C:\Program Files\Microsoft Office\Office12>
Get-AuthenticodeSignature excel.exe | Format-List *
````
## Pour & Contre : 

- Pratique
- Permet de gérer des chaînes de caractères avec des unités de 16 bits
mais
- Peut faire des ravages si fait à la va vite

## Équivalences Linux & Mac : 
- Linux :
![](https://user.oc-static.com/files/160001_161000/160272.png)
	- SH = ancêtre de tous les shell (.sh)
	- Bash : Shell par défaut mais aussi sous MacOS X.

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

## Workshop : 
````POWERSHELL
$rep = New-Object -TypeName System.String -ArgumentList "***************** FICHIER DE LOG *****************"


$dateDuJour = Get-Date -UFormat "%Y-%m-%d"
$log = "C:\temp\"+$dateDuJour+"-result.log"


if(Test-Path $log){
    Write-Host("Création d'une copie .bak")
    Copy-Item $log $log'.bak'
} else {
    $rep | Out-File -FilePath $log
}

$table = Get-ChildItem -Path (Get-Item env:\PUBLIC).value -Recurse -File

$taille_max = 512 #512Ko
$cpt = 0 
foreach ($data  in $table){
    if($data.Length/1000 -ge $taille_max){
        $var = $data.FullName+"   trouvé : "+$data.Length/1000+" Ko"
        Add-Content -Path $log -Value $var
        $cpt++
    }
}
$cptAff = "Total de fichier trouvés > "+$taille_max+" Ko : "+$cpt
Add-Content -Path $log -Value $cptAff


#New-EventLog -LogName Application -Source Test ##Création du Log - A n'exécuter qu'une seule fois

#Get-EventLog -List #==> Consulter la liste des EventLog
Write-EventLog -LogName Application -Source "Test" -Message $cptAff -EventId 1000 -EntryType information
#Get-EventLog -list | ? log -eq Application #Voir le nombre d'entrées dans "Application"
#Get-EventLog -LogName Application -Source "Test" #voir tous les enregistrements
#eventvwr #afficher journal windows

#Write-Host((Get-winevent -listlog Application).providernames) #Renvoi la liste des sources (ceux qui écrivent)
if ((Get-winevent -listlog Application).providernames -contains "Test") { write-Host("Journal bien exécuté") }

#New-Item -Path HKLM:\HKEY_LOCAL_MACHINE\SOFTWARE\ -Name Test #Créer la clé de registre
$item = Get-ItemProperty -Path HKLM:\HKEY_LOCAL_MACHINE\SOFTWARE\Test -Name "Date"
if (((get-date $dateDuJour) - (get-date $item.Date)) -gt 5){
    $message = "Vous n'avez pas lancé le script depuis le "+$item.Date+" Faites attention !!!"
    Write-EventLog -LogName Application -Source "Test" -Message $message -EventId 1000 -EntryType warning
}

#Set-ItemProperty -Path HKLM:\HKEY_LOCAL_MACHINE\SOFTWARE\Test -Value $dateDuJour -Name "Date"
Set-ItemProperty -Path HKLM:\HKEY_LOCAL_MACHINE\SOFTWARE\Test -Value $cpt -Name "Compteur fichiers > 512 Mo"

Get-EventLog -LogName Application -Source "Test" #voir tous les enregistrements
````


