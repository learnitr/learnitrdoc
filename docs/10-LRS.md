# (PART) LearnIt::R LRS {.unnumbered}

# Learning Record Store & MongoDB {#lrs}



Ce chapitre ouvre la section consacrée à **LearnIt::R LRS**, la version complète de la plateforme. Autant dans la version GitHub, nous évitons l'utilisation de serveurs spécialisés, autant ici, nous les mettons en œuvre pour offrir plus de fonctionnalités aux enseignants et aux étudiants. Par contre, l'installation, la configuration et la maintenance de ces serveurs exige des compétences techniques en adminitration système sous Linux. Beaucoup d'enseignants et de formateurs n'ayant pas ces compétences, il est fortement conseillé de travailler en collaboration avec votre service ou gestionnaire informatique pour déployer ces serveurs.

Un "[Learning Record Store](https://xapi.com/learning-record-store/)" (LRS en abrégé) est une base de données un peu particulière configurée pour stocker toutes les données relatives aux traces d'apprentissage. Il existe un protocole standard pour l'échange entre les plateformes d'apprentissage et les LRS : le [xAPI](https://xapi.com). Ce dernier format permet d'interconnecter différents systèmes qui produisent ou qui consomment des données entre eux. Il existe de nombreux outils payants [Learning Locker](https://learningpool.com/solutions/learning-locker-community-overview/), [Watershed LRS](https://www.watershedlrs.com), ou gratuits tel [SCORM Cloud LRS](https://rusticisoftware.com/products/scorm-cloud/lrs/?_ga=2.47789487.923837694.1641462138-1391329406.1641462138) ou [TRAX LRS](https://traxlrs.com) pour gérer ce type de base de données. Ces outils [assurent la compatibilité nécessaire](https://adopters.adlnet.gov/products/all/0).

\BeginKnitrBlock{note}<div class="note">
Il existe aussi un autre (ancien) standard nommé SCORM. Une initiative cmi5 vise à fusionner les deux et à fournir une interface standardisée pour interopérer des outils différents au sein d'un "Learning Management System" (LMS) hétérogène. En gros, cmi5 = xAPI + LMS, [voir P.27 ici](https://adlnet.gov/assets/uploads/cmi5%20Best%20Practices%20Guide%20-%20From%20Conception%20to%20Conformance.pdf).
</div>\EndKnitrBlock{note}

## Notre "LRS"

Les évènements que nous enregistrons ne sont pas tous formatés en xAPI pour le moment. Ainsi, les données provenant de H5P le sont, mais pas celles provenant de Learnr, Shiny ou Wooclap. Dans LearnIt::R, l'approvisionnement des données se fait directement dans un format propre aux documents que sont stockés dans la base de données. Ainsi, le travail côté serveur est simplifié et il est possible d'utiliser un service tel que [MongoDB Atlas](https://www.mongodb.com/atlas). Ce service est gratuit pour une base de données de 512Mo, ce qui est largement suffisant pour tester l'outil sans se ruiner. La version payante qui autorise plus d'espace de stockage est également pécuniairement très intéressante là ou des véritables LRS (donc des outils plus spécialisés) sont beaucoup plus chers.

Le choix de MongoDB a été motivé par plusieurs facteurs :

- La possibilité d'utiliser gratuitement MongoDB ATLAS, un serveur sécurisé et fiabilisé contre les pannes sur le cloud pour y stocker (temporairement) jusqu'à 512Mo de données.

- La facilité d'installer un serveur MongoDB local uniquement (machine personnelle autant que serveur dédié).

- La simplicité de backup/restauration de données pour un même serveur ou entre serveurs avec `mongodump`/`mongorestore`.

- La facilité d'exportation des données avec `mongoexport`, ainsi que les drivers pour PHP (Wordpress/H5PxAPIKatchu) et pour R avec le package {mongolite}.

- La rapidité de requêtes relativement complexes, et la possibilité de mettre au point de telles requêtes avec [MongoDB Compass](https://www.mongodb.com/products/tools/compass).

Par contre, au niveau des points négatifs, nous avons :

- Utilisation d'un port particulier 27017 qui doit être ouvert. Ce port pourrait très bien être fermé dans un réseau universitaire à destination d'étudiants (Wifi étudiant) car il est contenu dans une plage de port traditionnellement utilisée par des jeux. Or, il est parfois nécessaire de limiter l'accès aux jeux sur le réseau d'un campus car sinon, cela saturerait la bande passante.

- Pas de possibilité d'utiliser un outil comme {dbplyr} pour définir les requêtes. Langage de requêtage spécifique (à noter que que nous développons aussi le package [{mongoplyr}](https://github.com/SciViews/mongoplyr) pour offrir une fonctionnalité similaire).

- Plus complexe à déployer et utiliser en local que, par exemple, une base de données SQLite.

- Bien moins couramment utilisé que MySQL pour des solutions sur le Net.

- Il faut utiliser un format de données différent pour l'échange et la publication des données (Open Data).

Nous ne sommes donc pas définitivement fixés sur ce choix comme solution unique de base de données pour la plateforme LearnIt::R LRS. En tous cas, MongoDB a le mérite de fonctionner correctement pour l'utilisation que nous en faisons jusqu'ici.


## Organisation

TODO: cette organisation plus complexe utilisant deux serveurs MongoDB ne convient pas à tous. Remainer en présentant une organisation classique autour d'un seul serveur, puis présenter de manière optionnelle la version avec deux serveurs, plus sécuritaire.

Ce type de base de données permet de stocker des données semi-structurées qui correspond bien à des évènements xAPI. Par sécurité, une base de données MongoDB *locale* est créée sur sur le serveur RStudio Connect. Elle n'est pas disponible depuis l'extérieur. Donc, seules les applications exécutées à partir de RStudio Connect y ont accès. Par contre, cela ne permet pas d'enregistrer les données générées ailleurs (H5P, applications learnr ou shiny exécutées localement ...). Ainsi, nous déployons une seconde base MongoDB sur le cloud via MongoDB ATLAS. Cette dernière sert uniquement à collecter *transitoirement* les données externes au serveur. Ensuite, un script sur RStudio Connect est exécuté à intervalle régulier pour rapatrier ces données externes dans la base de données interne.

La solution à deux serveurs offre les avantages suivants :

- Sécurité maximale des données une fois rapatriées dans la base sur le serveur dédié (parce qu'elle n'est pas accessible de l'extérieur).
- Duplication des points d'accès (et MongoDB ATLAS utilise trois serveurs différents sur le cloud) afin de garantir que les données puissent toujours être récoltées, même si le serveur dédié ne répond plus (résilience en cas de panne).

### Base interne

TODO: expliquer l'installation et le fonctionnement de la base MongoDB interne.

Pour tester l'accès à un port afin de vérifier si la base MongoDB est accessible, on peut utiliser la commande `nc` (netcat) ou `netstat`. Par exemple, pour tester le port 27017, on peut faire ceci :

```
# Accès en local (il faut succeeded!)
nc -zv 127.0.0.1 27017
# Accès distant (remplacer xxx.xxx.xxx.xxx par l'adresse ip du serveur)
nc -zv xxx.xxx.xxx.xxx 27017
# En local pour déterminer l'état du port 27017
netstat -pano | grep 27017
```

On peut aussi directement tester l'accès à la base MongoDB avec les outils en ligne de commande installés depuis un ordinateur distant comme ceci :

```
mongo "mongodb://xxx.xxx.xxx.xxx:27017"
db
quit()
```

### Base externe

TODO: expliquer l'installation et le fonctionnement de la base MongoDB externe. Deux comptes, par exemple, **input** avec des droits limité uniquement en écriture pour l'introduction des données et **teacher** avec des droits en lecture et écriture.

### Format des données

Si vous utilisez les fonctionnalités {learnitdown} dans votre bookdown, vous pouvez diversifier l'offre d'exercices (H5P, Shiny, learnr). Il vous faudra bien entendu pouvoir enregistrer l'activité dans ces différents exercices. Afin d'éviter la multiplication des formats de données, la plateforme LearnIt::R LRS homogénise la présentation des événements entre les différentes applications. Ces données sont toutefois enregistrées dans des collections différentes (**h5p**, **learnr** et **shiny**) dans la base MongoDB. Chaque entrée contient les champs suivants :

- **_id:** C'est l'identifiant unique du document MongoDB attribué automatiquement lors de son insertion

- **session:** La session à partir de laquelle l'évènement a été généré. Cette information est surtout utile pour les applications Shiny. Pour H5P, c'est un identifiant complémentaire de l'utilisateur enregistré qui est indiqué. Le plus souvent du type `email: mailto:user@site.com`, mais pas forcément. Les tutoriels learnr n'introduisent rien dans ce champ.

- **date:** La date au format GMT à laquelle l'évènement a été généré. Cette date est enregistrée à la microseconde près, mais la résolution réelle est moins bonne (probablement entre 50 à 150µsec, sauf pour les applications Shiny où le temps est comptabilisé à la milliseconde près).

- **id:** Uniquement pour H5P, l'identifiant du widget, le numéro à rentrer dans le shortcode, par exemple, `[h5p id="3"]` pour l'item 3.

- **app:** L'identifiant textuel de l'application, voir convention pour le nom des applications ci-dessous.

- **version:** Le numéro de version pour les apps Shiny et learnr. Les applications H5P n'ont malheureusement pas de numéro de version. Ce champ contient donc `null` dans ce cas, sauf s'il s'agit d'un "sous-contenu" (voir convention de noms d'apps ci-dessous), dans ce cas, nous aurons l'identifiant du sous-contenu.

- **user:** Utilisateur actuel sur la machine où l'application s'exécute.

- **login:** Login GitHub/Wordpress de l'utilisateur (peut être le même ou différent de **user**).

- **email:** Adresse mail institutionnelle (valeur passée par Moodle dans `iemail`). Pour les applications H5P, l'email Wordpress est renseigné généralement dans session, s'il a été défini. Pour les applications Shiny, l"identification complète de l'utilisateur, y compris son email Wordpress se retrouve dans le champ **data** pour l'évènement **started**.

- **course:** Le cours que suit l'étudiant, sous forme de l'identifiant interne, Moodle ou autre.

- **institution:** Nom de l'institution où l'étudiant est enrôlé.

- **verb:** Le verbe xAPI correspondant à l'évènement (voir ci-dessous verbes xAPI).

- **correct:** `"TRUE"` ou `"FALSE"` uniquement pour les évènement correspondant à des réponses à une question, donc les verbes **answered** (pour les questions) ou **submitted** (pour du code R). Dans les autres cas, sa valeur est `""`, sauf si une application Shiny a été lancée, mais sans que l'utilisateur ne clique jamais sur le bouton `Submit`. Dans ce cas, la valeur vaut `"NA"`.

- **score:** Nombre de points obtenus pour la question. Dépend de l'application. Pour les H5P, il s'agit du nombre d'items corrects moins le nombe d'items erronés. Pour Shiny, c'est le nombre de widgets correctement positionnés par rapport au nombre testé dans la solution. Enfin pour learnr, c'est `0` ou `1` selon que la question est complètement correcte ou pas. Pas pertinent en dehors de réponses et donc ce champ vaut `null` dans ce cas.

- **max:** Score maximum pouvant être obtenu pour la question. Dépend du contexte. En dehors de réponses aux question, ce champ vaut `0`.

- **grade:** La fraction de bonne réponse, ou autrement dit, une note sur 1 pour la question. Si on a `1` la réponse est totalement correcte, mais si on a `0.5`, elle ne l'est qu'à moitié. En dehors de réponse à une question, ce champ n'a pas de sens et sa valeur est `null`.

- **label:** Le libellé de l'item générant l'évènement. Pour Shiny, c'est le nom du widget. Pour H5P, c'est le texte de la question, etc.

- **value:** La valeur sélectionnée pour le widget Shiny, ou la réponse donnée pour H5P, par exemple. Dépendant du contexte.

- **data:** Des données complémentaires sous forme d'une chaîne de caractères contenant un objet JSON. Pour H5P, il s'agit de l'évènement xAPI généré. Pour learnr, il s'agit d'une partie du contenu (moins `correct` et `label` qui sont extraits dans les champs correspondant). Pour Shiny il s'agit d'informations complémentaires sur les widgets.

### Verbes xAPI

Les évènements xAPI de H5P sont déjà décrits par des verbes. Par contre, les évènements Shiny ou learnr sont décrits par des dénominations propres. Afin d'homogénéiser le tout dans la plateforme LearnIt::R, tous les évènements Shiny et learnr sont traduits en verbes xAPI selon le tableau suivant :

| Verbe xAPI  | learnr                    | shiny         | H5P        | Remarque   |
|:------------|:--------------------------|:--------------|:-----------|:-----------|
| started     | session_start             | start         |            | L'utilisateur a démarré activement une application |
| attempted   |                           |               | attempted  | L'utilisateur arrive sur un H5P (différent de started) |
| exited      |                           | inputs/quit   |            | L'utilisateur quitte volontairement (différent de stopped) |
| stopped     | session_stop              | stop          |            | L'application s'est arrêtée |
| displayed   | section_viewed            |               |            | L'utilisateur visualise une section |
| progressed  | section_skipped           |               | progressed | L'utilisateur avance dans les exercices (différent de displayed) |
| seeked      | video_progress            |               |            | Progression dans une vidéo |
| interacted  |                           | inputs        | interacted | Dans Shiny: sauf boutons 'submit' & 'quit' |
| answered    | question_submission       |               | answered   | Réponse à une question (hors code R) |
| reset       | reset_question_submission |               |            | Nouvel essai après une réponse erronée |
| executed    | exercise_submission       |               |            | Bouton 'Run Code' dans learnr |
| evaluated   | exercise_result           |               |            | Résultat de l'évaluation du code R, si correct == ""  |
| submitted   | exercise_result           | inputs/result |            | Idem, mais si correct != "" dans learnr, Bouton Shiny 'submit'  |
| computed    |                           | outputs       |            | Par défaut, les outputs ne sont pas enregistrés dans Shiny |
| debugged    |                           | errors        |            | Erreur détectée dans une app Shiny  |
| assisted    | exercice_hint             |               |            | Sauf si dernier hint = réponse => revealed |
| revealed    | exercice_hint             |               |            | Pour les boutons 'solution' dans learnr & dernier 'hint' |
| completed   |                           |               | completed  | Dans H5P, un bilan général est calculé à la fin |


## Installation

TODO: détailler les procédures de configuration de MongoDB Atlas, et d'installation sur un serveur dédié ou en local.

- Création d'un compte MongoDB ATLAS, une organisation et un serveur M0 (gratuit). Deux bases de données sont créées : "test" et "cours". La première ne contient rien d'important et sert... à des tests comme son nom l'indique. La seconde contient nos données avec les collections "h5p", "learnr" et "shiny". Naturellement, vous pouvez donner le nom que vous souhaitez à ces deux base de données.

- Installation et configuration de MongoDB sur un serveur dédié avec uniquement la base de données "cours", et des tables "users", "h5p", "learnr", "shiny", ... Également, mise en place de scripts cron (voir plus loin) pour récupérer les données venant de MongoDB ATLAS toutes les 5 min et pour faire des backups journaliers de la base de données.

- Installation d'un serveur MongoDB local sur une machine sous Windows ou macOS pour travailler hors connexion avec un snapshot des données obtenu via `mongodump`/`mongorestore`.

Ces trois systèmes sont détaillés ci-dessous.

### MongoDB ATLAS

TODO: expliquer l'organisation de MongoDB ATLAS...

### Serveur MongoDB local associé à Posit Connect

TODO: développer l'innstallation du serveur MongoDB, voir https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu. Ci-dessous un script exemple qui date et qui doit être mis à jour :

```
sudo apt-get install gnupg
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
sudo apt-get update
# Either latest version
sudo apt-get install -y mongodb-org
# ... or specific version
#sudo apt-get install -y mongodb-org=4.4.0 mongodb-org-server=4.4.0 mongodb-org-shell=4.4.0 mongodb-org-mongos=4.4.0 mongodb-org-tools=4.4.0
# Optionally freeze version
#echo "mongodb-org hold" | sudo dpkg --set-selections
#echo "mongodb-org-server hold" | sudo dpkg --set-selections
#echo "mongodb-org-shell hold" | sudo dpkg --set-selections
#echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
#echo "mongodb-org-tools hold" | sudo dpkg --set-selections
# Determine if it is systemd or init that is used
ps --no-headers -o comm 1 # systemd here
# Start mongodb
sudo systemctl start mongod

# By default, MongoDB only accept clients from same machine (what we want)
# Data directory is /var/lib/mongodb, but we want /data1/mongodb
sudo mkdir /data1/mongodb
sudo chown -R mongodb:mongodb /data1/mongodb
# Edit /etc/mongod.conf and adjust dbPath to /data1/mogodb
sudo nano /etc/mongod.conf

# Start the MongoDB server and test it from R
sudo systemctl start mongod
# If an error, first try sudo systemctl daemon-reload
# Status (can also use start/stop/restart
sudo systemctl status mongod
R
library(mongolite)
con <- mongo(collection = "shiny", db = "sdd")
con$insert(trees)
trees2 <- con$find()
all.equal(trees, trees2)
con$drop()
rm(con)
gc()
q("no")

# Quand c'est bon, activer le service définitivement avec
sudo systemctl enable mongod

# Create a program supervisor for common environment variables (not done yet)
mkdir -p /opt/scripts
touch /opt/scripts/connect-env.sh
nano /opt/scripts/connect-env.sh

# Add this in the file:

#!/bin/bash

echo arguments: “$@“ >&2
echo >&2
export MONGO_URL=“mongodb://sdd:sdd@sdd-umons-shard-00-00-umnnw.mongodb.net:27017,sdd-umons-shard-00-01-umnnw.mongodb.net:27017,sdd-umons-shard-00-02-umnnw.mongodb.net:27017/test?ssl=true&replicaSet=sdd-umons-shard-0&authSource=admin”
export MONGO_BASE=“sdd”
exec “$@“

Then:
chmod 755 /opts/scripts/connect-env.sh

Add the following in /etc/rstudio-connect/rstudio-connect.gcfg
[Applications]
Supervisor = /opt/scripts/connect-env.sh
```

### Serveur local MacOS ou Windows

Le plus simple est de regrouper dans un dossier sur un disque SSD externe tous les programmes et toutes les données relatives à un snapshot donné de notre LRS afin de pouvoir manipuler tout cela de manière indépendante et sécurisée (accès local uniquement). Le dossier de base contient deux sous-dossiers : `bin` et `db`.

Le dossier `bin` contient tous les exécutables macOS et Windows nécessaires, soit `mongod` et `mongo` de la distribution de MongoDB, et `bsondump`, `mongodump`, `mongoexport`, `mongofiles`, `mongoimport`, `mongorestore`, `mongostat` et `mongotop` des MongoDB Data Tools de version compatible. Ces versions sont choisies identiques à celles déployées sur votre serveur dédié et/ou sur MongoDB Atlas^[Aligner les versions majeurs de MongoDB avec MongoDB Atlas nécessite de jongler un peu avec les mises à jour car nous n'avons aucuns contrôles des versions sur Atlas dans un compte gratuit. Vous êtes prévenu à l'avance lorsqu'une migration vers une nouvelle version de MongoDB sur Atlas va se faire et vous devez vous aligner !]. Ces différents exécutables sont récupérés depuis les installeurs (format .tgz ou .zip) macOS et Windows depuis [MongoDB on-premises](https://www.mongodb.com/try/download/community) et [MongoDB tools](https://www.mongodb.com/try/download/database-tools). Ses exécutables macOS et Windows occupent un peu plus de 1Go dans le dossier `bin`.

Le dossier `db` contient le snapshot de la base de données. Il est créé en réalisant un `mongorestore` à partir des données obtenues par `mongodump` depuis votre serveur externe comme ceci (exemple sous MacOS) :

```
# On your serveur, navigate to a emty folder, say '/data/dump/sdd', then:
mongodump # Possibly restrict with --collection=<col_name> and/or --query='{"x": {"$gt":1}}'
# Compress this folder
cd ..
tar -czvf sdd.tar.gz sdd
rm -rf sdd
# Go back to your local machine, then:
cd <empty_temporary_dir>
scp unser@server.edu:/data/dump/sdd.tar.gz sdd.tar.gz # Enter password...
tar -xzvf sdd.tar.gz
rm sdd.tar.gz
cd ..
<path_to>/mongorestore # Of course, the local MongoDB server must be running here
```

En plus de ces deux sous-dossier, il y a également deux scripts qui permettent de lancer le serveur MongoDB sous macOS ou sous Windows. Ces scripts sont :

- `sdd_server_macos.sh` :

```
#!/bin/bash
cd "$(dirname "$0")"
port=27017
./bin/mongod --port $port --dbpath db
```

- `sdd_server_win.bat` :

```
@echo off
set port=27017
start cmd /c bin\mongod --port %port% --dbpath db
```

Le script correspondant à votre système doit être exécuté soit dans un terminal macOS, soit dans une fenêtre Powershell Windows après avoir changé le répertoire par défaut vers le dossier racine :

```
cd <dossier_racine>
./sdd_server_macos.sh ou ./sdd_server_win.bat
```

TODO: Un script qui remplit le sous-dossier `bin` en fonction du système, et qui traite aussi le cas Linux Ubuntu.

Un démon MongoDB serveur local est lancé sur le port 127.0.0.1:27017 et sert les données présentes dans le sous-dossier `db`. Les messages du serveur sont directement affichés dans la fenêtre terminal. À partir de ce moment, on peut travailler tranquillement sur les données, les analyser et/ou prototyper des outils à déployer ensuite sur le serveur sans craintes de casser la base de données principale. Ce serveur ne permet qu'un accès local. Donc, il est parfaitement sécurisé.

\BeginKnitrBlock{note}<div class="note">
Les versions de MongoDB sur le serveur et sur votre disque SSD doivent toujours être les mêmes, ainsi que celles sur MongoDB Atlas. En cas de changement de version, il faut donc bien penser à remplacer les exécutables du dossier `bin` par les nouvelles versions avant de continuer à travailler !
</div>\EndKnitrBlock{note}

### Mise-à-jour de MongoDB

La version de MongoDB serveur est dictée par celle installé dans MongoDB Atlas. Voici les étapes à réaliser pour une mise à jour :

- Vérification de la compatibilité du driver PHP dans Wordpress et upgrade éventuel (voir le chapitre \@ref(wordpress)).

- Vérification de la compatibilité de {mongolite} dans R, à la fois sur l'ordinateur des enseignants et sur les machines des étudiants :

```
packageVersion("mongolite")
# If needed:
#remotes::install_github("jeroen/mongolite@vX.X.X")
```

- Vérification de l'option "featureCompatibilityVersion" qui doit être mise à une valeur correcte (selon que vous voulez pour l'instant rester compatible avec l'ancien sans utiliser les fonctionnalités de la nouvelle version de MongoDB ou pas), donc dans R :

```
library(mongolite)
# Start the local server, then... check featureCompatibilityVersion
admin <- mongo(url = "mongodb://127.0.0.1:27017/admin")
admin$run('{ "getParameter": 1, "featureCompatibilityVersion": 1 }')
# If it is lower than 4.4, check first upgrade recommandations, then
#admin$run('{ "setFeatureCompatibilityVersion": "4.4" }')
```

- Mettre à jour les versions de MongoDB Tools. Faire cet upgrade sur les versions locales aussi.

- Tester la compatibilité de la nouvelle version de MongoDB sur une instance locale de la base de données et y exécuter les différents requêtes. Si tout va bien, mettre à jour les serveurs.


## RGPD et droit d'auteur

Vous ne pouvez pas installer de base de données pour collecter des données à caractère personnel en Europe sans vous conforter au règlement général sur la protection des données. Les questions relatives aux données d'un point de vue plus légal sont traitées ici afin que vous puissiez réaliser les démarches nécessaires pour vous mettre en conformité. Si votre cours fait partie d'un cursus dans une institution universtaire ou autre, voyez avec votre département juridique pour établir le document ad hoc. Tant que vous y êtes, vérifiez aussi si vous respectez le droit d'auteur et les diverses licences éventuelles pour votre matériel en ligne.

### RGPD

Le **Règlement Général sur la Protection des Données** ou RGPD (n° 2016/679) adopté par l'Union Européenne en 2016 régit la façon dont les données à caractère personnel peuvent être collectées et utilisées. Ce règlement est en faveur de l'utilisateur (ici l'étudiant). Il faut notamment son accord pour collecter et utiliser ses données personnelles. Or de telles données sont indispensables pour suivre la progression des étudiants, les noter, etc.

Par exemple à l'UMONS, l'étudiant signe le document adéquat lors de son inscription ([charte de la vie privée](https://web.umons.ac.be/fr/mentions-legales-et-protection-de-la-vie-privee/charte-vie-privee-umons-20190605/), complétée de documents disponibles sur l'Intranet UMONS). Ce document mets **Moodle** en conformité par rapport au RGPD, mais pas les outils externes, tels que ceux de LearnIt::R. Nous devons donc préciser exactement ce que nous devons faire pour être en conformité RGPD par l'intermédiaire d'un document complémentaire affiché sur le site (entrée [Protection de la vie privée](https://wp.sciviews.org/politique-de-confidentialite/) dans le menu à gauche) qui informe et précise la façon dans les données à caractère personnel des étudiants sont utilisées. Assurez-vous d'avoir l'accord express de vos étudiants par rapport à un tel document.

L'utilisateur doit avoir la possibilité d'effacer intégralement ses données personnelles s'il le souhaite lorsqu'il efface son compte d'un site. C'est indiqué explicitement dans le RGPD. Cependant, des restrictions à ceci sont indispensables pour la bonne gestion du suivi des étudiants à l'Université. C'est pour cela que l'étudiant a du signer un accord lors de son inscription. Afin d'être en conformité avec cette directive, nous avons rajouté un bouton `Effacer mes données personnelles` dans la première page des cours bookdown. S'il clique sur ce bouton, l'utilisateur peut ensuite lire le contenu de manière anonyme et les activités learnr/Shiny ne sont **pas** enregistrées. Par contre, H5P est toujours enregistré et les données historiques antérieures sont toujours dans la base de données.

L'anonymisation ou la pseudonymisation des données est de mise lorsque ces données servent à une étude générale (par exemple, évolution des performances des cohortes d'étudiants avec les outils progressivement mis en place, études scientifiques, ...). Cela passe par l'effacement des données de toute information à caractère personnel. Le nom, numéro de matricule ou adresse email de l'étudiant sont remplacés par un identifiant générique, par exemple, "étudiant 1", "étudiant 2", etc. Il faut mettre cette pratique en œuvre pour toutes les études rétrospectives générales visant à estimer l'impact pédagogique des outils mis en place (voir appendice \@ref(anonymisation)).

##### Lien utiles {-}

- [RGPD pour les développeurs](https://lincnil.github.io/Guide-RGPD-du-developpeur/) par la [CNIL](https://www.cnil.fr/professionnel) en France,

- [Techniques d'anonymisation](https://www.cnil.fr/fr/le-g29-publie-un-avis-sur-les-techniques-danonymisation)

- [Recommandation relative aux mots de passe](https://www.legifrance.gouv.fr/cnil/id/CNILTEXT000033929210/)

- [Présentation de l'information RGPD aux utilisateurs](https://design.cnil.fr/concepts/information/)

### Droit d'auteur et licence

TODO: discuter le choix de la licence pour le contenu en ligne.

TODO: mise en œuvre d'un outil anti-plagiat.

TODO: aspects relatif à l'IA et aux LLM.

