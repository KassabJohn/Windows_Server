**Membre du groupe :**

**John Kassab -- Hugo Jacques -- Loick Odry**

**Vous trouverez en supplément la configuration du Windows Server en
tête de page**

**Configuration Windows Server**

Dans VMWare → Edit → Virtual Network Editor ⟶ passer Administrateur pour
gérer les cartes réseaux

Dans NAT, on décoche local DHCP car c'est nous qui allons le mettre en
place.

-   Subnet IP : 192.168.5.0 / 255.255.255.0

New Virtual Machine ⟶Custom ⟶install later ⟶Windows Server 2019 ⟶UEFI

1 & 4 processor config + 2048MB Ram

NAT → Store Virtual Disk as SF → 200GB

Delete printer ⟶ placer l'OS dans le CD/DVD

Datacenter Bureau

Personnaliser ⟶ On split 200 GB / 2

Install VMWare Tools ⟶ Typique

Ouvrir les paramètres réseaux ⟶ Protocole IPv4

-   Adresse IP : 192.168.5.11

-   Sous réseau : Auto

-   Passerelle : 192.168.5.2 (serveur DNS préféré)

On ping pour vérifier que l'on a bien Internet

On met à jour Windows Update

On renomme le poste : SRV-WIN2019

On update tout et on fait une snapshot après maj et VM éteinte.

**Rôle de serveur ADDS**

![Projet WDS \| Cardineaud
Steeve](mediafolder/f808047fcfd2e02e29ee55f2a27abdd395ab93eb.png)

Config de déploiement → Ajouter une nouvelle forêt → nom de domaine
racine : **contoso.adds**

Nouveau fctnl de la forêt → Win Serv 2016

Domaine → idem

**Ne pas cocher délégation DNS**

Mdp : Passw0rd

Nom de domaine Netbios : CONTOSO

Gestion disque → changer D en E dans le gestionnaire de disque et faire
un nouveau volume avec la partie non utilisé et mettre le volume D au
format NTFS.

Nom du volume : Data

Dossier de la BDD et des fichiers journaux : *D:\\ADDS\\NTDS*

Dossier SYSVOL : *D:\\ADDS\\SYSVOL*

On vérifie la config générale puis on installe

**Rôle DHCP**

Nouvelle étendu : 192.168.5.101-110

Plan ip → 101 à 110

Plage d'exclusion : 101 à 110

Passerelle : 192.168.5.2

DNS : 192.168.5.2 / contoso.adds

Réserver une IP par adresse MAC en 192.168.5.101 pour le poste client

Ajouter des rôles de serveur→ Serveur DHCP

-   Fonctionnalités : rien

-   Terminer config DHCP

Nom d'utilisateur : on laisse contoso admin puis on se rend dans :

Outils → DHCP → Clic droit sur IPv4 → Nouvelle étendu :

-   Nom : Etendu-192.168.5.101-110

-   192.168.5.101 / 192.168.5.110 en fin

-   Ajout d'exclusion : 101 à 110

-   Durée du bail : 8 jours

-   Passerelle : 192.168.5.2

-   Nom du domaine et serv DNS : 192.168.5.11

-   Serveur WinS : on oublie

-   Activer l'étendue

Reservation → Nouvelle étendue → PCclient1

-   Adresse IP : 192.168.5.101

-   Adresse MAC : 123456789AB

-   Ajouter.

**Installation et mise en place de WDS**

**Gestionnaire de serveur -\> Gérer -\> Ajouter des rôles et
fonctionnalités** puis cocher **Services de déploiement Windows** et on
clique sur suivant jusqu'à arriver au bouton d'installation

![](mediafolder/ef2c3834798d5fa0b2f3de491a6c266a734bcd21.png)

Ensuite, on se rend dans **Outils → Service de déploiement Windows**

Clic droit sur notre **serveur → Configurer le serveur**

![](mediafolder/dfc60258983242bc3b881ec38e0c167684153dc9.png)

On choisit « **Intégré à Active Directory** » car ce serveur est membre
du domaine contoso.adds

![](mediafolder/efc42420675af5061789d7345ac17779ff0b85a2.png)

On modifie le chemin d'accès du dossier d'installation de
*C:\\RemoteInstall* à *D:\\RemoteInstall.*

On ne change rien au paramètre du serveur DHCP proxy

Dans les paramètres initiaux du serveur PXE on choisit répondre
uniquement aux ordinateurs clients connus

![](mediafolder/f5888ab6950556bf8ff70e5958bb1cdf2af547b0.png)
En cas d'erreur de démarrage du service, on démarre manuellement :

![](mediafolder/1f582cc048e070393e373a9f15c530a85bda35da.png)

On se rend dans **« Gestionnaire de serveur -→ Outils → Service de
déploiement Windows → Serveurs → *NotreServeur* -\> Image de démarrage →
Ajouter une image de démarrage ».**

On parcourt le disque et on y place le fichier **boot.wim** de l'ISO
Windows

Après ajout de l'image de démarrage on obtient ceci :

![](mediafolder/a06de4ec42668a4e4c50e5370e56f643656435f9.png)
Nous allons mettre en place pour le déploiement, l'image d'installation
(**install.wim**). Pour cela il faut se munir des DVDs de Windows qui
serviront pour le déploiement.

Pour se faire on se rend dans **« Gestionnaire de serveur → Outils →
Sélectionnez Services de déploiement Windows → Serveurs → *NotreServeur*
→** Clique droit sur **Image d'installation → Ajouter un groupe
d'image »**

![](mediafolder/167526f260f8432b9ebb7e871e48304018dc2d57.png)

![](mediafolder/2306d30ccdf960396d53d13d47391357c1399c99.png)

Clic droit sur le nom du groupe puis **« Ajouter une image
d'installation → Parcourir** puis on sélectionne le fichier
**« install.wim » → Suivant**

On choisit les images que l'on veut ajouter sur le serveur en les
cochant/décochant puis on termine l'opération.

Cliquez sur **Serveurs** → Faire un clic droit sur ***NotreServeur***. →
sélectionnez **Toutes les tâches** → sélectionnez **Démarrer**.

SI tout a correctement été exécuté, nous devrions obtenir ce message.

![](mediafolder/d8726f21fa5b3315e962c4129e130ee3a9fa755e.png)

De plus, une flèche verte se met à côté du nom du serveur.

Dans Service de Déploiement Windows, on ajoute un nouvel ordinateur avec
l\'adresse MAC généré par VMWare dans NAT puis *avdanced* et enfin
adresse MAC

On crée un nouveau pc client dans Service de Déploiement Windows dans
périphérique de pré installe Active Directory

On donne un nom et dans ID de périphérique on met ladresse MAC sans les
« **:** »

Dans DHCP, dans réservations on ajoute une réservation avec une adresse
ip et l\'adresse MAC

Puis dans Utilisateur et ordinateurs Active Directory, on ajoute un
nouvel ordinateur et on remplit de 0 jusqu'à suivant

Pour vérifier que tout fonctionne, on se rend dans la VM sans image afin
de tester notre service WDS.

Une fenêtre s'ouvre et nous propos quel système d'exploitation à
installer.

![](mediafolder/80368df96c69233c81f2f9cd51fcba07350fd4f1.png)

Une fois le système d'exploitation choisi, on continue l'installation de
Windows de manière classique.

*Fin de l'installation et déploiement de WDS.*

Pour créer un fichier de réponse pour l'automatisation de Windows, on
doit tout d'abord installer ADK Manager :
<https://go.microsoft.com/fwlink/?linkid=2120254>

Lors de l'installation nous n'avons besoin que des outils de
déploiement :

![](mediafolder/a86961c468f455ee50d5b3e4fcebc2eb524d2838.png)

On installe puis on ouvre l'outil

On extrait l'ISO Windows pour y récupérer le fichier **install.wim**

On se rend ensuite dans **ADK** → **Image Windows**. Sélectionner
l'image Windows et on recherche le fichier **install.wim**

![](mediafolder/62ba020dff3c71a1cddbe7d100873bee4317c416.png)

ADK nous demande ensuite pour quelle version de Windows nous souhaitons
créer un fichier de réponse. Dans notre cas → Windows 10 Education N

**Ce dossier technique a pour but de créer 2 fichiers de réponses :**

1)  **Fichier de réponse boot → langue / disque / compte / choix ISO /
    login**

2)  **Fichier de réponse install → OOBE**

Fichier de réponse boot.xml

Tous les paramètres seront à ajoutés à la passe 1 WindowsPE :

![](mediafolder/7e7759785f3e2842be602f6a1b8847a7e317d5b1.png)

-   International Core nous permet de choisir la langue du système

![](mediafolder/58c2480f6ad203125d5499fccd2a7e1a1a082d4d.png)

![](mediafolder/7fd398be3351d5599f4b6fc0f79c7d3017330732.png)

![](mediafolder/620aea789bb26d3fcd13896d7b07ded0adc8de0d.png)

-   Windows Setup nous permet de gérer le partionnement des disques : on
    ajoute donc 4 *CreatePartitions* et 4 *ModifyPartitions*

![](mediafolder/147a0b60df3ab39264519f1bd604f07ed01715ec.png)

![](mediafolder/2373a1fa74bf84929864752173871b1ece830aca.png)

![](mediafolder/dff377a578a95fd3e2e9f7686640e57ad391661c.png)
![](mediafolder/cf877cd50849f2c51f88b7aaa934344fc7fbece8.png)

![](mediafolder/57aa988df27a3e8649fa1720befa7a29ef1f2638.png)
![](mediafolder/bff024a1d9f8cd66edde43d6f2f3eb775663cedd.png)

![](mediafolder/9acb3f75fd96de0d7ac8392047b951b69b0b87e6.png)
![](mediafolder/96ae993611b17299534edb338f28d7b0ae07315d.png)

![](mediafolder/d84bc8064052cc7b558e09866313498b3f99c283.png)
![](mediafolder/5822ca801768440bd532b5e67a62950d00077ef8.png)

On ajoute ***UserData*** avec en paramètre ***ProductKey*** à Setup
Neutral

![](mediafolder/4e95c825bb129087dacc5446be751061cd672ae5.png)

![](mediafolder/ce7ee5411d696745dfae684bf0a6626f7d290061.png)

![](mediafolder/78787cee4e988f49179eb3155cf746b67da85f94.png)

La clé de license a été généré avec Azure

-   On ajoute le service de déploiement Windows qui permet de définir
    quel ISO installer et sur quel disque l'installer

![](mediafolder/cba68a7124753292c56c4cfd6fc0681571b45de2.png)

![](mediafolder/198e2c295a6368895ae031ee535d1f0bdc4e91f0.png)

![](mediafolder/e50c03c20e5a016391e869e55355149d30de7563.png)

On installe donc l'ISO Windows 10 Education N sur le disque ID 0 (se
référer à l'ID du *DIskConfiguration*) et sur la partition n°4

-   Vu que nous appartenons au domaine contoso, nous allons aussi
    automatiser la partie d'identification

On ajoute *Login* à Windows Setup :

![](mediafolder/d0e993d8bf391e7dc646ffda8ca9bc2c361dd2ef.png)

![](mediafolder/67c5da342138727958045361c5e1e86e89167098.png)

Notre boot.xml est terminé, on le sauvegarde, on le placera plus tard en
même temps que le fichier de réponse install.xml

Fichier de réponses install.xml

Tous les paramètres présentés devront être ajoutés à la partie 7
*oobeSystem*

![](mediafolder/b41c462ebf351304ce70ff016d38aab684cc7e23.png)

![](mediafolder/562a09171e728d5acd45a57ecee7693ca4dde3e4.png)

![](mediafolder/48c3adbd0f2f29cb56e4b300485ac7929fb10c1a.png)

![](mediafolder/57508c339aa9b27d53ef8220193bf921654fcb71.png)

![](mediafolder/bb9c6655928ba95f9694e42636f578615c3b5627.png)

On définit un mot de passe de son choix dans *Password* puis *Value*

![](mediafolder/c1257e408770cb86fb9284ef6e991fe197a6052a.png)

![](mediafolder/b2559a9e08f160fad0c0ac214032872ffbcb93b8.png)

![](mediafolder/a02add0606d96e5e05bcf7c7616f7a71b00925c8.png)

On définit un mot de passe de son choix dans *Password* puis *Value*

Nous avons désormais terminé nos 2 fichiers de réponses.

On déplace nos fichiers de réponses xml dans le dossier
**R*emoteInstall/WdsClientUnattend*** de notre serveur.

Pour les déployer, on se rend sur notre serveur Windows → **Outils** →
**Service de déploiement Windows**

Clic droit sur notre serveur → **Propriétés** puis on se rend dans
l'onglet **Client**.

Dans Architecture 64x UEFI on sélectionne notre fichier de réponse
**boot.xml**

![](mediafolder/a7a7d7c9146a1e8360a0474239f7d90779533135.png)

Pour le fichier de réponses install.xml, on se rend dans le dossier
image d'installation, on double clique sur notre groupe d'image puis on
sélectionne l'ISO sur lequel on veut appliquer le fichier de réponse

Clic droit sur l'ISO puis enfin Propriétés, on coche « autoriser l'image
à s'installer en mode sans assistance » puis on y place le fichier de
réponse **install.xml**

![](mediafolder/09b8906795f7b86cb02511ec9169aefb6768bf36.png)

Les fichiers de réponses sont terminés et déployés, la prochaine
installation de Windows se fera de manière automatisée sans la moindre
interaction demandée de la part de l'utilisateur

**Capture d'une image Windows**

Nous allons voir comment capturer Windows avec une image de capture sur
le service WDS.

Avant d'effectuer la capture du poste, le système d'exploitation doit
avoir été préparé à l'aide de l'utilitaire sysprep.

L'image de capture doit être activée sur le serveur WDS.

Avant toute chose, nous devons installer le rôle « **service de
déploiement Windows **»

On doit tout d'abord configurer notre serveur :

![](mediafolder/76e989c62655f894ed715cf567a9d2ba2e2e5ac1.png)

![](mediafolder/f73f26f7bb4cfb92089a8efc1e1ab263e36f06b5.png)

![](mediafolder/4c403b593f7dc53b0b372ec31d6ffe49906dbf4f.png)

Le service PXE doit répondre à tous les clients mêmes inconnus. Si l'on
souhaite approuver et même nommer des clients inconnus, il faut cocher
la case exiger l'approbation de l'administrateur

Le service WDS est configuré, il reste à le démarrer manuellement

![](mediafolder/4a26458408e6444b8c1e3e55fb95949ed539cc2c.png)

-   CREATION DES IMAGES DE DEMARRAGE ET CAPTURE

 Image de démarrage → Ajouter une image de démarrage

On fournit comme image de démarrage le fichier **boot.wim** contenu dans
l'iso du DVD de Windows 10 Education

On renomme pour indiquer clairement qu'il s'agit d'une image de boot, et
de quelle version de Windows elle est issue.

L'image sera stockée dans D:\\RemoteInstall\\Boot\\x64\\images.Une fois
l'image de boot créée, on passe à l'image de capture, que l'on crée à
partir de cette image de démarrage

![](mediafolder/165263a13371de2f3aac2e6f39111497337fdbc7.png)

On définit un nom de l'image et un fichier temporaire ou l'image sera
créée, avant qu'elle ne soit copiée dans le dossier WDS.

A partir de ce moment, on peut démarrer une station sur le boot PXE
(F12) pour vérifier que le DHCP attribue une adresse, et que le serveur
WDS prend le relais pour proposer de démarrer un déploiement ou une
capture

![](mediafolder/d1fba107d4c27ae264ece5514196b052443eaf3c.png)

Pour déployer une image, on crée d'abord un groupe d'image, avant
d'ajouter l'image elle-même.

Dans un 1^er^ temps, on va ajouter une image issue du DVD original de
Windows 10, à savoir le fichier **install.wim**

Clic droit sur notre image d'installation → Ajouter une image
d'installation

![](mediafolder/b10e3885b9fb96ad94b2d09bbad2bb0e82ef0dae.png)

On choisit l'image souhaitée parmi celles contenues dans le fichier .wim
(il peut y en avoir plusieurs\...)

![](mediafolder/5371567d212e20affc83fb7d33a12d25d73dfd26.png)
L'image apparaitra après avoir cliqué sur le bouton Terminer. Cette
image permet de déployer un système d'exploitation vierge de toute
modification.

Nous allons à présent préparer et capturer une image en vue d'un
déploiement.

On se rend donc dans notre VM Client qui appartient au serveur puis on
exécute le programme **sysprep**

![](mediafolder/d41634519948301663535da90ea2788f41e1db61.png)

On entre en mode OOBE et on fait attention à arrêter le système

![](mediafolder/5be0a837e64fc07a09cb58895bd2c46ea6131833.png)

Une fois arrêté, la station est prête à la capture cependant il ne faut
pas la redémarrer tant que le serveur WDS n'est pas prêt à installer.

Dans les services de déploiement Windows, clic droit pour créer un
groupe d'images

![](mediafolder/3fa10c60a67f0cb6d16b46180dffeb584a4258a4.png)
On peut alors démarrer la station avec un F12 de manière à faire un boot
PXE.

On choisit comme image de boot non pas celle qui permet de déployer,
mais celle qui permet de capturer.

![](mediafolder/05843e03ec06bc0a43a21915123d8d8afb1d12b9.png)

![Capturer une image d\'un
Windows](0961ce54c6516a6ce1006b6407422cff11c0fa28.jpg)

Une fois booté, on arrive sur un assistant qui va nous permettre de
capturer une image d'installation. On choisit le volume, le nom de
l'image ainsi qu'une description.

![](mediafolder/ac5c41aebc1ee1af47d02900121003973d67b4c3.png)

L'image va d'abord être enregistrée localement dans un dossier que l'on
peut préciser avec *Parcourir*\...

![](mediafolder/b15afe5408ed80b3986ffbd686299b6ddc59f66e.png)

On coche impérativement « Télécharger l'image sur un serveur » puis on y
renseigne le nom complet (SRV-WIN2019) ou son adresse IP puis on clique
sur Connexion.

Une fois la liaison établie, on peut choisir le groupe d'image

![](mediafolder/738cd95e2586e16012c16d6c4f09b6ae4ad4f5ea.png)

La capture se lance et prend généralement entre 15 et 30 minutes.

L'image est alors disponible dans WDS, dans le groupe indiqué lors de la
capture.

Image d'installation → W10_Captured → w10

L'image se présent sous la forme de 2 fichiers dans le dossier
RemoteInstall.

A partir de ce point, notre serveur WDS est donc capable de faire booter
nos PC Client en mode PXE (grâce au rôle DHCP qui est installé sur le
même serveur que le WDS), capturer une image d'un système
« **sysprepé** » et de déployer cette image sur d'autres machines.

Problèmes rencontrés :

Après avoir utilisé 3x sysprep sur une même machine (étant donné que
nous n'avions jamais utilisé cet outil, nous étions obligé de faire
plusieurs tests), sysprep devient indisponible

Ce tutoriel nous a permis à corriger ce problème :
<https://www.tutos.eu/7090>
