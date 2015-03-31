#Sécurité dans Docker
----------------------------------



##Préambule

Ce document n’est pas un tutoriel d’utilisation de Docker. Par conséquent, il n’est pas destiné à présenter au lecteur cette technologie de déploiement d’application, ni à l’aider à la prendre en main. De la documentation est largement disponible sur internet à cet effet. 
Le présent document a pour objectif d’amener un utilisateur ou potentiel utilisateur à prendre du recul sur la sécurité de cette technologie 
A l’image de technologie qu’il décrit, et surement par nécessité le document est collaboratif.

##Introduction
Docker est une technologie de gestionnaire, très léger et rapide, de conteneurs Linux. Docker permet de créer des images contenant l’application, et seulement elle, dont l’utilisateur aura besoin pour une action précise. Cette image est alors placée dans un serveur, le Docker Hub, afin d’être réutilisée sur n’importe quelle machine faisant tourner Docker.

On parle alors de virtualisation légère qui se doit d’être profondément différente de celle fournie par une machine virtuelle. 

Ce document ne cherche pas à plus développer ce qu’est Docker ou comment il fonctionne. Pour cela nous conseillons grandement de visiter [le site officiel](https://www.docker.com/) de cette jeune startup française au succès mondial. Vous pouvez aussi retrouver le code de Docker sur [ce lien](https://github.com/docker) (lien vers Github) et aider à améliorer le projet.

Nous cherchons ici à nous intéresser au sujet de la sécurité de Docker. Cette technologie étant très jeune et en pleine expansion nous pouvons observer une évolution permanente due à un besoin montant et à un aspect collaboratif de grande ampleur grâce au caractère open-source du code. 

Comme toutes technologies, une menace sécuritaire existe et est bien présente. Il existe beaucoup d’informations à ce sujet sur internet. L’ambition de cet article est de rassembler toute cette information et de la rendre disponible pour tous. 

Pour accompagner l’idée de Docker, il a semblé naturel de créer un document de référence open-source. Cela permettra à l’information contenue d’évoluer en même temps qu’évolue Docker, ainsi d’être le plus à jour et de rassembler l’ensemble des conseils disponibles sur la toile. 

Il a donc été choisi une approche en trois parties principales. Docker étant un condensé de technologies fournis par le noyau de Linux, il semble indispensable dans un premier temps de détailler ces technologies qui auront un rapport direct avec la problématique de sécurité. Il est ensuite important de mettre en place un modèle de menace de Docker et de répertorier l’ensemble des risques que peuvent rencontrer les utilisateurs et de leurs donner les solutions pour palier à ces problèmes. Finalement, pour séparer la menace de la solution, il faut développer une dernière partie sur la mise en place dans Docker même de ces nouvelles technologies pour résoudre des problèmes ou en tout cas les prévenir.

Nous espérons que ce document deviendra une référence incontournable pour chaque utilisateur de Docker en fonction de ses besoins.


##I.	Les technologies du conteneur Docker
###A.	Les technologies du noyau

Docker simplifie l’utilisation d’outils existants et c’est sa principale force. S’il est possible d’isoler des conteneurs : processus, interfaces réseau, points de montage, c’est grâce à l’utilisation des namespaces Linux. Un processus qui s’exécute dans un espace de nom correspondant à un conteneur ne sera visible que par ce conteneur et le système hôte exécutant ce conteneur, point. La virtualisation par conteneur est plus rapide que la virtualisation par hyperviseur car les conteneurs ont directement accès aux ressources matérielles de l’hôte. La consommation de ces ressources pouvant être gérées par l’hôte en question grâce aux control groups (cgroups) du noyau Linux.

Tout ceci fait donc de Docker un chroot dopé aux stéroïdes, mécanisme qui sera présenté dans la suite du document.

À ses débuts Docker était une couche d’abstraction à LXC car il permettait de s’affranchir de la complexité de ce dernier en gérant pour nous un certain nombre d’opérations, plus facilement. Aujourd’hui Docker va beaucoup plus loin que LXC puisqu’il dispose de sa propre plate-forme : Libcontainer (bien que LXC soit toujours proposé sous forme de plugin).

Et ce qui démarque Docker des autres, c’est la rapidité des flux de travail que l’utilisateur va pouvoir mettre en place : tout est fait pour réduire le temps entre le développement, les tests, le déploiement et l’utilisation de l’application en production. Et pour cela, on trouve les images.

Une image est le composant de base de Docker. C’est un instantané d’un système d’exploitation (par exemple Debian). Ces images sont disponibles sur un dépôt public, géré par Docker, appelé le Docker Hub. 

###B.	Les groupes de contrôle

####1.	Définition
Les groupes de contrôle ou cgroups permettent d’allouer des ressources – comme du processeur, de la mémoire système ou de la bande-passante –en groupes de processus hiérarchiquement organisés.

Les cgroups fournissent un moyen de hiérarchiser des groupes et de labéliser des processus et d’y appliquer des limites de ressources. Normalement, tous les processus devraient recevoir le même niveau de ressources systèmes. Avec cette approche, les applications qui induisent un nombre important de processus obtiennent plus de ressources que celles avec peu de processus, indépendamment de l’importance de ces applications.

####2.	Fonctions
On peut séparer les fonctions fournis par les cgroups :
 
* **Limitation des ressources** : des groupes peuvent être mis en place afin de ne pas dépasser une limite de mémoire — cela inclut aussi le cache du système de fichier.

* **Priorisation** : certains groupes peuvent obtenir une plus grande part de ressources processeur ou de bande passante d'entrée-sortie.
* **Comptabilité** : permet de mesurer la quantité de ressources consommées par certains systèmes en vue de leur facturation par exemple.
* **Isolation** : séparation par espace de nommage pour les groupes, afin qu'ils ne puissent pas voir les processus des autres, leurs connections réseaux ou leurs fichiers.
* **Contrôle** : figer les groupes ou créer un point de sauvegarde et redémarrer.

####3.	Utilisation des cgroups
L’utilisation de ces groupes se fait de plusieurs façons :

* En accédant au système de fichier virtuel cgroup manuellement.
* En créant et gérant des groupes à la volée en utilisant des outils tels que cgcreate, cgexec, cgclassify (de libcgroup)
* Le « démon de moteur de règles » qui peut automatiquement déplacer les processus de certains utilisateurs, groupes ou commandes vers un cgroup en respectant ce qui est specifié dans la configuratin.
* Indirectement à travers d’autres logiciels qui utilisent cgroups, tel que la virtualisation LXC, libvirt, systemd. 


####4.	Utilisation dans Docker
Ces cgroups permettent Docker de partager les ressources des hardwares aux conteneurs et, si besoin, de mettre en place une limite de temps et des contraintes. Par exemple : limiter la mémoire disponible pour un conteneur spécifique.

###C.	Les « namespaces » 

Dans Docker, les namespaces, technologie d

Un namespace est un espace isolé du reste du système en termes de ressources. Un processus s’exécutant dans un namespace n’a aucune visibilité sur l’utilisation ressources qu’ont les processus  extérieurs à ce namespace. Ceci est réalisé par la technologie « namespace » du noyau Linux qui abstrait le système de ressources pour en afficher une vue restreinte aux processus s’exécutant dans un namespace.

Le noyau Linux propose à ce jour 6 types namespaces. Il est donc possible d’isoler les processus entre eux vis-à-vis de 6 types de ressources différentes :

Nom du namespace ---> Type d’isolation réalisée  
IPC	---> System V IPC,    POSIX message queues  
Network	---> Network devices, stacks, ports, etc.  
Mount ---> Mount points  
PID --->	Process IDs  
User --->	User and group IDs  
UTS --->	Hostname and NIS domain name  


A chaque exécution d’un processus conteneurisé (Docker run) le daemon docker commence par créer un ensemble de namespaces et execute ensuite le processus dans ces namespaces. Ce processus conteneurisé n’a donc aucune visibilité sur l’extérieur du conteneur et encore moins la possibilité d’affecter l’environnement extérieur du conteneur.

###D.	Le mécanisme « chroot »

####1.	Définition
Un chroot dans les Unix est une commande permettant de changer le répertoire racine d’un processus de la machine hôte.
####2.	Objectif
Cette commande permet d’éviter une compromission complète d’un système lors de l’exploitation d’une faille. Par exemple si un pirate utilise une faille présente sur une application chrootée, il ne pourra avoir accès qu’à l’environnement isolé et pas à l’ensemble du système d’exploitation. Chroot est donc là pour limiter des dégâts. 

###E.	Les capabilities

Communément, nous distinguons deux types de processus : les processus privilégiés (EUID est 0) et les non-privilégiés (EUID autre que 0). Les capabilities sont un raffinement de cette attribution binaire des droits. Elles sont des unités de privilèges du système. Elles permettent donc d’attribuer à un processus uniquement les droits qu’il nécessite et non pas l’intégralité des droits sur le système. Prenons l’exemple d’un serveur web qui a besoin d’ouvrir un port inférieur à 1024. En lui attribuant la capability CAP_NET_BIND_SERVICE, nous lui offrons cette possibilité sans lui accorder tous les droits sur le système. 
  
Dans Docker, les conteneurs peuvent également se voir attribuer des capabilities. L’attribution de capabilities à un conteneur se fait au lancement du conteneur à l’exécution de la commande run. Par défaut, aucune capability n’est accordé au conteneur sauf quelques-unes.

Liste des capabilities autorisées par défaut :

CHOWN, DAC_OVERRIDE, FSETID, FOWNER, MKNOD, NET_RAW, SETGID, SETUID, SETFCAP, SETPCAP, NET_BIND_SERVICE, SYS_CHROOT, KILL, AUDIT_WRITE
 
Si nécessaire l’utilisateur peut accorder au conteneur d’autres capabilities en passant en paramètre de la commande run la liste de celles-ci.

###F.	Seccomp

#####1.	Définition
Seccomp qui est le diminutif de Secure Computing mode, est un mécanisme de sécurité qui fournit une sandbox dans le noyau de Linux. Seccomp permet un processus de faire une transition à sens unique vers un état sécurisé (« secure ») où aucun appel système n’est possible excepté `exit()`, `sigreturn()`, `read()`, et `write()`. S’il venait à essayer un autre appel, le noyau détruirait le processes avec un `SIGKILL`. Cela permet une isolation des ressources systèmes.

Ce mode seccomp est disponible via l’appel système `prctl()` en utilisant `PR_SET_SECCOMP` comme argument.

#####2.	Seccomp-bpf
Plus récemment, il a été ajouté une extension à seccomp. Seccomp-bpf permet un filtrage des appels systèmes utilisant un politique de sécurité implémentée. Il utilise des règles des Berkeley Packet Filters. 

###G.	La Libcontainer

La Libcontainer est une interface standard qui permet de créer des conteneurs et même de modifier un conteneur après sa création. Plus généralement, cette interface permet à Docker de manipuler directement les namespaces, les cgroups, les capabilities, les systèmes de contrôle d’accès AppArmor et SELinux, les interfaces réseaux (netlink) et les règles de firewall.

###H.	Le mécanisme de conteneurisation

Docker automatise la création de conteneur. Définir à quel moment et de quelle manière sont configurés les conteneurs. 


##II.	Modèle de menace
Docker en tant que programme complexe et tout à fait récent peut entrainer des problèmes de sécurité. Plusieurs questions peuvent venir à l’esprit lors de l’utilisation de Docker. Une des principales est de se demander si les conteneurs sont sécurisés?

La réponse est bien sur “ni oui ni non”. Comme il a déjà été dit, la conteneurisation permet une isolation, donc un apport en sécurité. Mais en fait on parle de pseudo-isolation.

###A.	Pseudo-isolation du conteneur

Les namespaces proposent une solution qui fonctionne sur le modèle de la mémoire virtuelle sur des machines multi-tenantes. Cela crée l’illusion qu’une application pense avoir l’ordinateur pour elle seule. En d’autres termes, via une “isolation”. Quand un processus ou un groupe de processus sont isolés grâce à ces namespaces, on dira qu’ils sont contenus. On crée donc un lien entre la virtualisation et la conteneurisation. Néanmoins, la conteneurisation isole d’une autre manière, il ne faut donc pas confondre les deux notions. Il faut bien comprendre cette différence pour utiliser des conteneurs en toute sécurité. La virtualisation reste une isolation totale des programmes au point que l’on peut y utiliser Linux. Les conteneurs ne sont pas aussi bien isolés.


En effet, le conteneur ne contient pas plusieurs choses:

1.	Tous les conteneurs partagent le même noyau. Par exemple si une application conteneurisée est détournée avec une vulnérabilité due à une montée de privilèges, tous les conteneurs et aussi l’utilisateur deviennent vulnérables. Ce problème spécifique sera développé dans la prochaine partie. 
2.	Certaines ressources ne sont pas isolées via des namespaces. Les keyring de noyaux sont un exemple de telles ressources. 
3.	Par défaut, les conteneurs héritent de beaucoup de capabilities du noyau. Bien que Docker fournisse la possibilité de restreindre les kernel capabilities, il faut comprendre en profondeur des besoins d’une application pour être lancée dans un conteneur plutôt que de le lancer dans une machine virtuelle. Le conteneur et l’application seront dépendants des capabilities du noyau dans lesquels ils résident.
4.	Il ne faut pas croire qu’un conteneur est du type “écriture unique et lancement quand on veut”. Puisqu’ils utilisent le noyau de la machine hôte, les applications doivent être compatibles avec ce noyau.

###B.	Généralement

Pour ce qui est des modèles de menaces de Docker, il en existe plusieurs. Ce document va essayer d’en développer certains.
Nous pouvons trouver :

* Les applications régulières
 * Servers web, bases de données…
* Les services systèmes (haut niveau)
 * logging, accès distance...
* Les services systèmes (bas niveau)
 * organisation des composants, networking...
* Le noyau
 * Politique de sécurité, drivers…
* Le cas des immutable infrastructures (en anglais)

La problématique de beaucoup d’informaticiens et de ce document est de réduire cette surface d’attaque, car plus la surface d’attaque sera grande plus il sera difficile de se défendre. 

Tout comme il se peut se poser la question d’adapter les conteneurs au besoin. Car chaque conteneur peut remplir un besoin, donc pour des systèmes plus complexes il faut développer des systèmes de différents conteneurs, entrainant forcément des failles de sécurité. 

Il revient souvent un problème récurrent comme conséquence qui est celui des privilèges. Certaines personnes pensent qu’avoir les privilèges root sur des conteneurs c’est avoir les privilèges sur l’ensemble de la “boite”. Cette affirmation est plutôt vrai mais la question à se poser est “doit-on absolument donner les privilèges roots aux conteneurs?”

####1.	Les applications régulières
Certaines applications régulières telles qu’un serveur Apache, MysSQL, Redis, ne nécessitent jamais les privilèges root (sauf pour installer les packages). Il ne faut donc jamais les lancer en mode root.

On peut faire une liste de risques et de conséquences dans ce cas-là.

#####a)	Risque 1: lancer du code arbitraire

Le risque serait que ces applications lancent du code arbitraire. Souvent ce problème se trouve dans le code lui-même ou peuvent trouver leurs origines via des brèches dans la sécurité qui entrainent des exécutions de code malicieux. Et il est impossible d’y remédier car c’est l’utilisateur qui autorise. Cela peut avoir de lourdes conséquences en termes de brèches de sécurité.

#####b)	Risque 2: passage de non-root à root

Ce risque qui trouve son origine dans les droits SUID (Set User ID). SUID est un droit qui s’applique aux fichiers exécutables. Lorsque le droit SUID est appliqué à un exécutable et qu’un utilisateur quelconque l’exécute, le programme détiendra alors les droits du propriétaire du fichier durant son exécution. On peut trouver des vulnérabilités dans les codes binaires SUID.

Pour corriger ce risque il faut donc réduire la menace des codes binaires SUID. Pour cela on peut soit les retirer tout simplement, sot retirer les bit SUID, soit élever le  système de fichiers avec un noSUID.

Dans le cas de Docker, enlever les codes binaires SUID est très simple  A COMPLETER. Docker ne supporte pas l’élévation noSUID.

#####c)	Risque 3: exécution arbitraire de code noyau

Il existe une possibilité lorsqu’il y a un bug dans les appels systèmes (exemple vmsplice* en 2008), d’exécuter du code arbitraire dans le noyau, ce qui peut entrainer de graves conséquences. 

Il y a deux principales méthodes pour y remédier. Premièrement de limiter les appels systèmes disponibles en utilisant un filtre seccomp-bpf par exemple et en créant ainsi une black liste et une white liste pour les appels systèmes. On peut deuxièmement faire tourner des noyaux plus solides. GRSEX est une bonne idée car stable depuis quelque temps. Il faut souvent mettre à jour son noyau.

Pour ce qui concerne Docker, il faut tout simplement faire plus d’expérimentations. 

#####d)	Risque 4: fuite vers un autre conteneur

Il peut de temps en temps avoir des bugs dans le code d’un namespace, entrainant ainsi une fuite dans les systèmes de fichiers.

Pour fixer ce problème il faut dans un premier temps fixer les utilisateurs namespaces. Il faut mapper les UID dans les conteneurs à un différent UID à l’extérieur. Pour Docker c’est en cours de révision. On peut aussi améliorer les modules de sécurité (SELinux par exemple). Ce sont des mécanismes pour isoler des processus. On peut noter en plus AppArmor qui sera utilisé avec SELinux dans Docker.

####2.	Les services systèmes (haut niveau)
Nous pouvons répertorier plusieurs types de services systèmes dits de haut niveau. On y retrouve les SSH, les cron ou les syslog par exemple. Ce sont des services nécessaires que l’on utilise tout le temps. 

Le problème principale est qu’ils sont “souvent” lancés en tant que root or ils n’ont pas toujours réellement besoin d’être lancés en tant que root. De plus c’est difficile de les lancer en tant que non-root mais ils restent des codes non arbitraires. 


#####a)	Risque 1: lancer du code arbitraire en root

Un premier risque dans ces cas-là est celui de lancer un code arbitraire en tant que root puisque l’utilisateur lance souvent ces services en tant que root. 

On retrouve ces codes arbitraires à cause de données mal formées. Notons que le risque est beaucoup plus élevé dans le cas du SSH que pour les syslog ou cron.

Pour régler ce problème il faut avant tout isoler les services dit sensibles. Par exemple on peut lancer un SSH dans un hôte bastion ou dans une machine virtuelle.


#####b)	Risque 2: désorganiser /dev

Il arrive que des codes malicieux désorganisent le dossier /dev.

Pour empêcher de tels problèmes, il est conseillé de mettre en place des groupes de controle pour les “devices”. Ainsi on crée des blacklists/whitelists.   Cette granularité permet de choisir les possibilités d’actions (lecture, écriture, aucun des deux ou les deux) et permet de donner des importances graduées aux composants.

Dans le cas de Docker il a été développé un support pour cette granularité. 

#####c)	Risque 3: utilisation d’appels root

Encore une fois via des codes malicieux il est possible de faire des appels root (mount, chmod, iptables…)

La solution à cette faille reste les capabilities. Cela casse le “root” pour beaucoup de permissions. 

Docker propose les capabilities comme solutions. 


#####d)	Risque 4: désorganiser /proc, /sys

Les codes malicieux peuvent aussi s’en prendre aux dossiers /proc ou /sys, ce qui peut avoir des conséquences dramatiques.

La solution se trouve donc dans la prévention pour des contrôles d’accès non autorisés. On pensera donc au MAC via AppArmor ou SELinux. 

Il est aussi possible d’implémenter de manière plus large les namespaces. Il existe des zones de procfs/sysfs qui sont “namespace-aware” mais d’autres ne le sont pas mais peuvent être modifiés en écrivant dans le code du noyau. 

Docker verrouille /proc et /sys pour éviter ces problèmes. 


#####e)	Risque 5: Fuites avec UID 0

Les codes malicieux peuvent entrainer des fuites dans UID 0.

Pour régler cela il faut une fois de plus employer des namespaces utilisateurs. Ainsi UID 0 qui se trouve dans le conteneur sera mappé à un UID aléatoire à l’extérieur. Si on fait une sortie, l’utilisateur ne sera pas root. Si on tente pour lancer des appels systèmes étranges, ils seront faits comme des UID non privilégiés.

Pour ce problème-là, Docker est en cours d’évolution et travaille pour trouver une solution. 


####3.	Les services systèmes (bas niveau)
Après avoir fait les appels systèmes de haut niveau il semble logique d’aborder ceux de bas niveau. Ici on parle de l’ensemble de l’organisation des composants (clavier, souris, écran), du réseau et de ses configurations du firewall et du système de fichiers. 

Comme ceux de haut niveau, ils sont tout le temps utilisés et sont nécessaires. Mais ils ne sont pas du tout utile dans des conteneurs car les composants “physiques” sont traités par la machine hôte et pour ce qui est de la configuration réseau et du système de fichiers, ils sont aussi traités par la machine hôte.

Il existe néanmoins des exceptions telles que custom mounts (FUSE) ou les network appliances. 

Il existe deux principaux risques. 

#####a)	Risque 1: lancer du code arbitraire en tant que root

Ce problème récurrent dans les appels de hauts niveaux se retrouve dans ceux de bas niveaux.
 
La solution reste toujours d’isoler les fonctions sensibles.
 
Docker propose une granularité pour palier à ce problème. On utilise par exemple la commande `docker run --net container:.. `pour les network namespace. nsenter sera utilisé pour d’autres opérations.

#####b)	Risque 2: lancer du code arbitraire avec tous les privilèges. 

Cette faille sera traitée comme un problème lié au noyau que l’on développera dans la suite de ce sujet.


####4.	Le Noyau
On sépare en trois parties. On y retrouve les drivers qui communiquent avec le hardware, les piles de réseau et les polices de sécurité qui contrôlent le reste.


“Si vous lancez quelque chose qui par définition demande tout le contrôle sur le hardware ou le noyau, les conteneurs ne vont pas rendre l’opération sécurisée. S’il vous plait n’essayez pas de vous tirer une balle dans le pied de manière sécurisée”

-- <cite>Jérôme Petazzoni, Membre de Docker.</cite>


A cela Jérôme Petazzoni répond à l’unique problème de sécurité lié au noyau, dans le cas où l’utilisateur lance du code avec TOUS les privilèges. Il faut dont lui donner son propre noyau et hardware virtuel c’est à dire de le lancer dans une machine virtuelle. On peut lancer cette machine virtuelle dans un conteneur ou elle-même peut contenir un conteneur.

On peut donc lancer un conteneur avec tous les privilèges, dans Docker, lui-même lancé dans une machine virtuelle qui est, elle, lancée dans un conteneur dans Docker. 

Tout cela est expliqué dans le Github personnel de [Jérôme Petazzoni](https://github.com/jpetazzo/docker2docker).

####5.	Immutable infrastructures

Ici on impose une nouvelle règle, on impose que le conteneur ne soit qu’en lecture possible. Il arrive très souvent que l’on doit écrire dans le conteneur. Pour cela il est mis en place un compromis pour lequel si on doit écrire, cela doit se faire dans une zone sans exécution (noexec area). La notion de scabilité est grâce à cela devenue très facile d’utilisation et cela rend la tâche beaucoup plus compliquée pour les utilisateurs malveillants.

###C.	Conteneur en fonction du besoin

###D.	Le problème root


##III.	Bonnes pratiques
###A.	Cas d’utilisation de Docker

####1.	Bonnes pratiques pour la création de conteneurs
 **Objectifs** : Sécurité du système hôte par confinement d’un processus. 

Définir les bonnes pratiques lors de la création d'un conteneur. Comment réduire la surface d'attaque et mettre en place plusieurs "barrières" de sécurité ? Cela passe entre autre par une construction réfléchie de son système de conteneurs et une bonne rédaction du DockerFile. Dans cette partie des exemples sont absolument nécessaires.
 
####2.	Bonnes pratiques pour l’utilisation de conteneurs 

Définir les bonnes pratiques lors de l'utilisation d'un conteneur. Notez ici que le conteneur peut soit avoir été forgé par son utilisateur qui en est donc également l'auteur, soit provenir du DockerHub qui peut/doit être considérer comme non-fiable. Q : Doit-on mettre en œuvre les mêmes mesures de sécurités dans les deux cas ?

###B.	DockerFile

Le DockerFile est un fichier qui contient les instructions de création d’une image Docker. La qualité de rédaction d’un DockerFile impact directement celle de l’image qui en résultera.

####1.	Règle n°1 : Inclure le minimum de ressources dans le conteneur.
De manière générale, dans un conteneur, il ne faut inclure que le minimum de package, de fichier, de processus nécessaire. Il faut avoir à l’esprit que tout package, fichier ou processus inutile au fonctionnement du programme conteneurisé sont davantage de failles, d’informations, voir de potentiel vecteurs d’attaques laissés à un attaquant. En effet, l’attaquant peut chercher à prendre le contrôle du conteneur. Dans ce cas le surplus de ressources lui offre davantage de vecteur d’attaques. Il peut également avoir déjà pris le contrôle sur le conteneur par exemple, en exploitant une faille du serveur web qu’il héberge, et dans ce cas les ressources inutiles peuvent lui fournir des informations sur le système hôte ou lui permettre d’étendre son contrôle sur le système hôte. 

Typiquement, 

####2.	Règle n°2 : Le conteneur ne doit contenir qu’un seul processus. 
Fondamentalement, Docker existe pour isoler les processus entres eux et du système hôte. Ainsi, logiquement, lancer plus d’un processus par conteneur n’est pas conseillé. Premièrement pour des raisons pratiques. Par exemple, si seul le code d’un serveur web lié à un service de base de données doit être mis à jour, opération de maintenance qui nécessite de recréer le conteneur du serveur, avoir séparé en terme de conteneur le service de base de données du serveur web, permet de ne recréer que le conteneur du serveur web. Cet exemple nous amène aux notions de système de conteneurs et de liaison entre les conteneurs. 

####3.	Règle n°3 : Utiliser un fichier .dockerignore
Il est fortement conseiller d’utiliser le fichier .dockerignore. Lorsque qu’il est présent dans le répertoire à l’origine de la création du conteneur, le fichier .dockerignore contenant la liste des dossiers et fichiers à ne pas inclure dans le conteneur. Tel qu’il a été dit plus haut le conteneur doit contenir le moins de ressources possible. Premièrement, car la suppression des fichiers et dossiers inutiles réduira la taille du conteneur et donc son temps de création, son temps de transfert sur le réseau et bien évidement sa taille sur la machine hôte.
 
Il n’est pas conseiller d’exposer un port « public » de la machine, c’est-à-dire de fixer le port de la machine hôte qui sera affecté au conteneur.  Principalement, par soucis de flexibilité car l’utilisateur du conteneur ne sera plus en mesure de choisir le port « public » sur lequel le conteneur est exposé. Ainsi, une seule instance de ce conteneur ne pourra être exécutée à la fois. 

**Exemple** : Dans le DockerFile « EXPOSE 80:8080 » fixe le port privé (interne) à 80 et le port public (externe) à 8080. « EXPOSE 80 » ne fixe que le port interne à 8080 et laisse à l’utilisateur la possibilité de fixer le port externe avec l’option -p lors du lancement de l’image du conteneur.

####4.	Règle n°4 : Bien rédiger son DockerFile.
Ceci n’est pas à proprement parler une règle de sécurité pour le DockerFile mais un conseil informatique général qui peut paraitre évident. Nous l’avons vu plus haut, un conteneur se doit d’être éphémère. Cela implique forcement que le créateur du conteneur, ou autre selon la situation, va devoir revenir sur le code conteneurisé et le DockerFile. Ainsi, il est conseillé de correctement rédiger son DockerFile. Pour rappel, la totalité des failles informatiques sont d’origine humaine. La machine de faisant uniquement ce que l’humain lui demande de faire.

Pour cela, il est conseillé de :

-	Ne mettre qu’un seul argument par ligne. Par exemple, pour la commande RUN.
Vous trouverez quelques exemples de DockerFile ci-dessous :

[lien 1](https://github.com/dockerfile/java/blob/master/openjdk-7-jre/Dockerfile)  
[lien 2](https://github.com/dockerfile/ubuntu/blob/master/Dockerfile)  
[lien 3](https://github.com/dockerfile/nginx/blob/master/Dockerfile)  

###C.	La digital signature verification

Depuis peu Docker propose une nouvelle fonctionnalité pour améliorer la sécurité des images fournies. Il propose de mettre en place une signature que le Docker engine vérifiera pour certifier du caractère officiel et optimisé d’une image. Ce sont des clés cryptographiques. Elles seront appelées les Official Repos. Elles sont optimisées par la communauté pour être les meilleures parties pour créer des applications distribuées. Cette signature ajoute donc un niveau de confiance.

Il est important de rappeler l’importance d’une telle signature puisque un téléchargement sur cinq depuis le Docker Hub Registry représente l’une de ces Official Repos. 

###D.	Combinaisons de SELinux/AppArmor/TOMOYO


#####1.	Principe
Comme il a été vu précédemment, il est devenu pratiquement indispensable de mettre en place  du contrôle d’accès pour accroitre la sécurité des conteneurs. AppArmor et SELinux (nous ne parlerons pas de TOMOYO qui reste une troisième alternative) en sont des solutions très performantes. 

On sait que Ubuntu propose des templates AppArmor très faciles d’utilisation pour les LXC. Ceci permet de savoir ce que fait notre logiciel. Pour mettre en place la bonne politique, il faut être sûr de surveiller l’application depuis son début. 

Dans le conteneur la configuration de AppArmor se définit dans le `lxc.aa_profile`

####2.	Security Options
Docker a mis en place une nouvelle commande pour la command line, --security-opt, qui autorise l’utilisateur à mettre en place des SELinux et AppArmor customisés. 

###E.	Seccomp pour limiter les syscalls

L’utilisation de Seccomp qui est disponible pour CentOS, Debian, Fedora, Gentoo, Oracle, Plamo et Ubuntu, peut être mise en place pour le conteneur en modifiant ses configurations et ainsi pour définir les rêgles seccomp. 

On agira donc dans `lxc.seccomp = /usr/share/lxc/config/common.seccomp.`

Pour Docker, cette fonctionnalité peut être activée en utilisant la commande `–lxc-conf` dans les paramètres `docker run`.

###F.	User mapping

Nous avons vu que pour empêcher par exemple une fuite vers un autre conteneur que les LXC permettent de remapper l’utilisateur et des groupes d’IDs. Le fichier de configuration devra alors avoir la forme : 

`lxc.id_map = u 0 100000 65536`  
`lxc.id_map = g 0 100000 65536`  


Dans cet exemple, cela mappe les premiers 65536 utilisateurs et groupes d’IDs dans les conteneurs 100000 à 165536 de la machine hôte. Les fichiers concernés sont donc **/etc/subuid** et **/etc/subgid.**

Pour le cas de Docker nous cherchons donc à ajouter des paramêtres `–lxc-conf` au docker run :

`docker run -lxc-conf=”lxc.id_map = u 0 100000 65536″ -lxc-conf=”lxc.id_map = g 0 100000 65536″`

###G.	Jouer avec les capabilities

Nous avons vu précédemment l’intérêt d’utiliser des capabilities pour améliorer la sécurité de Docker.  Grâce à eux on peut donner à Docker des rôles spécifiques qui doivent être données à des conteneurs. Il est intéressant de savoir que l’on peut utiliser les options add et drop ensembles. Dans un premier temps on choisit d’autoriser toutes les capabilities et enfin d’en dropper certains, cela permet de limiter les capabilities.
 
`docker run –cap-drop=CHOWN`    
`docker run –cap-add=ALL –cap-drop=MKNOD`


On peut fournir une liste de capabilities à dropper en toute sécurité :

* audit_control
* audit_write
* mac_admin
* mac_override
* mknod
* setfcap setpcap
* sys_admin
* sys_boot
* sys_module
* sys_nice
* sys_pacct
* sys_rawio
* sys_resource
* sys_time
* sys_tty_config

Ces options sont à mettre en place dans les fichiers de configurations `lxc.cap.drop` et `lxc.cap.keep`

###H.	Utilisation du réseau
Pour ce qui est de la configuration réseau qui peut entrainer de nombreux problèmes. Il est conseillé de s’intéresser à [https://docs.docker.com/articles/networking/](l’article) fourni par Docker où l’ensemble des configurations sont très bien indiquées. 


###I.	Lien entre conteneur
 
 




















##Conclusion
La sécurité est un problème qui touche tout le monde. Même un utilisateur isolé peut se retrouver face à des failles. 

C’est pour cela qu’il est important que chacun soit averti de ces risques. Un utilisateur prévenu en vaut deux. Mais il ne faut jamais oublier que la sécurité en tant qu’idéal n’existe pas. C’est pour cela qu’il faut souligner l’importance de mettre à jour un tel document.







 
##Remarques sur le document

Une version anglaise est attendue très prochainement. Nous avons choisi d’écrire ce document en français pour une première approche et un premier appel aux modifications.  


##Bibliographie

1.	Site officiel de Docker 
2.	 Présentation de Jérôme Petazzoni 
3.	
