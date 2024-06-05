# Posit Connect {#connect}



[Posit Connect](https://posit.co/products/enterprise/connect/) permet de partager des documents R Markdown ou Quarto, des applications Shiny et des tutoriels learnr facilement depuis un serveur centralisé. C'est pratique pour inclure ces items directement dans la matériel pédagogique en ligne et pour permettre aux étudiant d'effectuer des exercices à domicile sans qu'ils n'aient forcément accès à un système R + RStudio fonctionnel. Posit Connect est payant (plusieurs dizaines de milliers de dollars par an), **mais est gratuit pour une utilisation dans un cadre académique sous certaines conditions**. Il faut montrer au travail du matériel pédagogique associé que l'outil est utilisé dans le cours et que les étudiants apprennent à employer RStudio ou d'autres logiciels Posit. La Licence doit être renouvelée annuellement et n'est valable que pendant la durée du cours (mieux vaut donc avoir des cours qui couvrent toute l'année). À partir de 100 utilisateurs, il est aussi possible d'utiliser l'interface plumber API. Le logiciel doit être installé sur un serveur que l'on configure et administre nous-même.

## Exemple d'installation

TODO: section ancienne datant de 2021, et les explication ne sont plus à jour par rapport à la version actuelle. Conservé pour l'instant à titre d'illustration seulement !

Nous avons opté pour un serveur ayant les caractéristiques suivantes :

- CPU Intel Core i7-6900k 3,20GHz à 8 cœurs/16 threads,
- 64Go de mémoire RAM DDR4 2.666Mhz,
- 512Go de disque SSD NVMe Samsung Pro 960 + 2 fois 3To de HDD Toshiba P300 7200rpm SATA III,
- Xubuntu 18.04.5LTS,
- MongoDB 4.4.11, MongoDB Tools 100.5.1,
- RStudio Connect 1.8.8.2 + R 4.0.5 (licence 100 utilisateurs),
- Accessible par ssh seulement en interne (ou au travers d'un VPN).

- Installation de R 3.6.3 (équivalent à "svbox2020" qui est une installation type de R + RStudio + la série de packages R nécessaires pour les exercices du cours) :

```
sudo apt install curl
export R_VERSION=3.6.3
curl -O https://cdn.rstudio.com/r/ubuntu-1804/pkgs/r-${R_VERSION}_1_amd64.deb
sudo gdebi r-${R_VERSION}_1_amd64.deb
```

- Installation de R 3.5.3 et 3.4.4, respectivement équivalents à "svbox2019" et "svbox2018" pour pouvoir également exécuter les exercices des années précédentes :

```
curl -O https://cdn.rstudio.com/r/ubuntu-1804/pkgs/r-3.5.3_1_amd64.deb
sudo gdebi r-3.5.3_1_amd64.deb
curl -O https://cdn.rstudio.com/r/ubuntu-1804/pkgs/r-3.4.4_1_amd64.deb
sudo gdebi r-3.4.4_1_amd64.deb

# Test
/opt/R/${R_VERSION}/bin/R --version
```

- Configuration de Java pour R :

```
sudo /opt/R/3.6.3/bin/R CMD javareconf
```

- Installation de RStudio Connect 1.8.2.1-12 :

```
cd ~/Downloads
curl -LO https://rstudio-connect.s3.amazonaws.com/rsc-installer.sh
sudo bash ./rsc-installer.sh 1.8.2.1-12
# Clean up
rm rsc-installer.sh
sudo rm rstudio-connect_*.deb
```

- Installation des dépendances :

```
sudo apt install build-essential libcurl4-gnutls-dev openjdk-7-* libxml2-dev libssl-dev texlive-full libev-dev
# Pour les packages R
sudo apt install libgmp10-dev libgsl0-dev libnetcdf6 libnetcdf-dev netcdf-bin libdigest-hmac-perl libgmp-dev libgmp3-dev
sudo apt install libgl1-mesa-dev libglu1-mesa-dev libglpk-dev tdsodbc freetds-bin freetds-common freetds-dev
sudo apt install odbc-postgresql libtiff-dev libsndfile1 libsndfile1-dev libtiff-dev tk8.5 tk8.5-dev tcl8.5 tcl8.5-dev libgsl0-dev libv8-dev
```

- Configuration de RStudio Connect :

```
sudo nano /etc/rstudio-connect/rstudio-connect.gcfg
# URL temporaire du server... https://rstudio-connect.university.edu
# SenderEmail = user@university.edu
# Port :3939 (valeur par défaut)
```

- Introduction de la clé d'activation : 

```
sudo /opt/rstudio-connect/bin/license-manager deactivate
sudo /opt/rstudio-connect/bin/license-manager activate xxxx-xxxx-xxxx-xxxx-xxxx-xxxx-xxxx
```

- Redémarrage de RStudio Connect :

```
sudo systemctl restart rstudio-connect
```

- Création du premier compte (admin). Naviguer vers http://rstudio-connect.university.edu:3939. S'enregistrer et créer le nouveau compte. Configuration également de l'envoi de mail (serveur SMTP sur port 587 temporairement si nécessaire).

- Installation de Python :

```
[Python]
Enabled = true
; Python 2.7.12 installed with Ubuntu 16.04
Executable = /usr/bin/python
; Python 3.5.2 installed with Ubuntu 16.04 (and used by svbox2018)
Executable = /usr/bin/python3
```

- Packages Python supplémentaires (pip et pip3 déjà installés) :

```
sudo pip install setuptools
sudo pip install virtualenv
sudo pip3 install setuptools
sudo pip3 install virtualenv
```

- S'il faut une autre version de Python, ici 3.6.7 pour svbox2019, il faut compiler depuis les sources :

```
# Required dependencies
sudo apt install libffi-dev libgdm-dev libsqlite3-dev libssl-dev zlib1g-dev

# Download and extract source code
cd ~/Downloads
export VERSION=3.6.7 PYTHON_MAJOR=3
wget https://www.python.org/ftp/python/${VERSION}/Python-${VERSION}.tgz
tar -xzvf Python-${VERSION}.tgz
cd Python-${VERSION}

# Build Python from source
./configure \
    --prefix=/opt/Python/${VERSION} \
    --enable-shared \
    --enable-ipv6 \
    LDFLAGS=-Wl,-rpath=/opt/Python/${VERSION}/lib,--disable-new-dtags \
    --enable-optimizations
make

# Install Python
sudo make install

# Clean up
cd ..
sudo rm -rf Python-${VERSION}
rm Python-${VERSION}.tgz

# Check Python install
/opt/Python/${VERSION}/bin/python${PYTHON_MAJOR} --version

# Install pip, setuptools and virtualenv in this Python version
wget https://bootstrap.pypa.io/get-pip.py
sudo /opt/Python/${VERSION}/bin/python${PYTHON_MAJOR} get-pip.py
rm get-pip.py
sudo /opt/Python/${VERSION}/bin/pip install virtualenv
sudo /opt/Python/${VERSION}/bin/pip install setuptools
```

- Ajouter cette version de Python dans la configuration de RStudio Connect (ajouter `Executable = /opt/Python/3.6.7/bin/python3` dans la section "[Python]") :

```
sudo nano /etc/rstudio-connect/rstudio-connect.gcfg
```

- Redémarrer RStudio Connect :

```
sudo systemctl restart rstudio-connect
```

- Installation du certificat SSL. Copie des fichiers vers `/etc/ssl/private` et changement des droits d'accès :

```
sudo chmod o= /etc/ssl/private/university.cer|crt|csr|key|pfx
```

- Dans l'exemple d'installation, le fichier `.cer` ne fonctionne pas, mais on peut créer un `.pem` à partir du `.crt` au format PKCS#7 :

```
sudo openssl pkcs7 -print_certs -in /etc/ssl/private/sdd_umons_ac_be.crt -out /etc/ssl/private/sdd_umons_ac_be.pem -outform PEM
sudo chmod o-r /etc/ssl/private/sdd_umons_ac_be.pem
```

- Modification du fichier de configuration de RStudio Connect. Utilisation du port ` :443`, le `.key` comme clé privée et le `.pem` comme certificat, et TSL minimum 1.1 :

```
sudo nano /etc/rstudio-connect/rstudio-connect.gcfg
```

- Redémarrage du serveur

```
sudo systemctl restart rstudio-connect
```

- Vérification du bon démarrage dans le fichier log

```
sudo cat /var/log/rstudio-connect.log
```

- Dans notre exemple, nous choisissons de mettre en place une synchronisation de `/data1` avec `/data2` régulièrement :

```
sudo crontab -e
```

Édition du fichier en y ajoutant :

```
# Sync /data1/ with /data2/ every hour:45
45 * * * * /usr/bin/rsync -av --delete /data1/ /data2/ &> /data/datasync.log
```

- Déplacement des données de RStudio Connect vers `/data1` :

```
sudo systemctl stop rstudio-connect
sudo mkdir /data1/rstudio-connect
sudo rsync -av --delete /var/lib/rstudio-connect/ /data1/rstudio-connect/
```

Édition à nouveau le fichier de configuration de RStudio Connect (DataDir = /data1/rstudio-connect) :

```
sudo nano /etc/rstudio-connect/rstudio-connect.gcfg
```

Redémarrage de RStudio Connect :

```
sudo systemctl start rstudio-connect
```

- Backup du système en utilisant `rear`, voir https://tecmint.com/rear-backup-and-recover-a-linux-system/

```
sudo apt install rear extlinux
```

- Formatage d'une clé USB :

```
sudo rear format /dev/sdX # In case of error rear format -- --efi /dev/sdX (in the server, it is /dev/sdc)
```

- Changer `/etc/rear/local.conf` et y ajouter ceci :

```
OUTPUT=USB
BACKUP=NETFS
BACKUP_URL="usb:///dev/disk/by-label/REAR-000"
```

- Exécuter ceci pour voir la configuration actuelle :

```
sudo rear dump
```

- Créer un disque de récupération (unmount REAR-000, puis ...)

```
sudo rear -v mkrescue # Just a rescue disk
sudo rear -v mkbackup # Rescue disk AND backup the system
sudo rear -v mkbackuponly # Only backup the system
```

- Pour restaurer le système, on boote depuis la clé USB. Lorsque le login est demandé, entrer `root`, puis `rear recover`. Redémarrer une fois la restauration terminée. Voir aussi http://relax-and-recover.org/usage/

## Commandes utiles

Ci-dessous, quelques commandes utiles pour l'administration du serveur sur lequel vous déployez Posit Connect :

```
# Changer la résolution d'écran
xrandr -fb 1920x1080 (-display:0)

# Inspection du système
inxi -GCS # General system info
top (-i) # hit q to quit
htop
# Boot and BIOS info
sudo dmidecode -t system
sudo dmidecode -t bios
uname -a # Basic system info
# Detailed hardware info (including eno1 network (1Gb Intel)
sudo lshw (-short)
sudo lshw -html > lshw.html
lscpu # CPU info
watch -n 2 sensors # CPU temperature
lsblk # Storage info
df -h
sudo disk -l
lsusb (-v)  # Info about USB devices
# Reconfigure video driver
sudo update -initramfs -U
sudo dpkg reconfigure server-xorg
lsmod | grep nouveau
# Reconfigure locale
localectl US-utf8
# Diagnostic script for RStudio Connect
sudo /opt/rstudio-connect/scripts/run-diagnostics.sh /path/to/output/directory
```

Reconfiguration de la localisation vers US-utf8 :

````
localectl US-utf8
```

Ajouter de capteurs matériels (contrôle de la température) pour suivre à distance la condition du hardware :

```
sensors
sudo apt install nvme-cli
sudo nvme list
sudo nvme smart-log /dev/nvme0
sudo nvme smart-log /dev/nvme0 | grep temperature
```

Résoudre un problème lié à une mauvaise version de Pandoc :

lorsque l'on a le message `loadlocale.c:130: _nl_intern_locale_data: Assertion cnt < (sizeof (_nl_value_type_LC_TIME) / sizeof (_nl_value_type_LC_TIME[0]))' failed`

```
sudo mkdir -p /opt/scripts
sudo nano /opt/scripts/rstudio-connect-env.sh
# Add this:

#!/bin/bash
echo arguments: "$@" >&2
echo >&2
export LC_ALL=C
exec "$@"

# Then:
sudo chmod 755 /opt/scripts/rstudio-connect-env.sh

# Then, change RSconnect config:
sudo nano /etc/rstudio-connect/rstudio-connect.gcfg
# In this section, change Supervisor=

[Applications]
Supervisor = /opt/scripts/rstudio-connect-env.sh

# Then, restart RStudio connect
sudo systemctl restart rstudio-connect
# This does not work => reedit rstudio-connect.gcfg and comment out Supervisor= with ; and restart RStudio Connect

# Need to rebuild the locales
# In /etc/locale.gen, I have only one item: en_US.UTF-8 UTF-8
# Then:
sudo mv /usr/lib/locale/locale-archive /usr/lib/locale/locale-archive.save
sudo locale-gen --no-archive
sudo locale-gen --no-archive en_US.UTF8
```

Utilisation d'une version différente de Pandoc :

```
cd ~/Downloads
wget https://github.com/jgm/pandoc/releases/download/2.11.4/pandoc-2.11.4-1-amd64.deb
sudo dpkg -i pandoc-2.11.4-1-amd64.deb
pandoc --version # Still version 1.17.2
which pandoc # /usr/local/bin/pandoc
/usr/bin/pandoc --version # 2.11.4
/opt/rstudio-connect/ext/pandoc/2.11/pandoc --version # 2.11.2 with same options
/usr/lib/rstudio/bin/pandoc/pandoc --version # Same 1.17.2 than pandoc --version

#Sys.setenv(PATH = paste("/opt/rstudio-connect/ext/pandoc/2.11", Sys.getenv("PATH"), sep = ":"))
#Sys.setenv(PATH = paste("/usr/bin", Sys.getenv("PATH"), sep = ":"))
res <- system("which pandoc", intern = TRUE,ignore.stderr = FALSE, ignore.stdout = FALSE)
#stop(res)
#stop(Sys.getenv("RSTUDIO_PANDOC"))
#stop(rmarkdown::pandoc_version())
stop(rmarkdown::pandoc_available(error = TRUE))
(v2 <- ex$month[!duplicated(ex$month)])

# May be problem with pandoc-citeproc? But apparently not needed any more
which pandoc-citeproc # /usr/local/bin/pandoc-citeproc
pandoc-citeproc --version # 0.10.1

# pandoc-citeproc is missing from version 2.11... copy version from 2.9!
sudo cp /opt/rstudio-connect/ext/pandoc/2.9/pandoc-citeproc /opt/rstudio-connect/ext/pandoc/2.11/pandoc-citeproc

sudo cp /opt/rstudio-connect/ext/pandoc/2.11/pandoc /opt/rstudio-connect/ext/pandoc/2.11/pandoc-orig
```

Dans le cas d'un problème avec le driver vidéo (au cas improbable où votre serveur offre aussi une interface type GUI)... voici quoi faire pour le résoudre :

```
# Reconfigure the video driver
sudo update -intiramfs -U
sudo dpkg reconfigure server-xorg
lsmod | grep nouveau
# Completely reconfigure the video driver if it gets stuck with Nvidia driver:
# Start in recovery mode: type <shift> repeatedly while booting
(sudo) update-initramfs -u

# Solved temporarily => could boot in X11, then, remove Nvidia drivers completely. 
sudo apt-get purge nvidia-*
#sudo mv /etc/X11/xorg.conf /etc/X11/xorg.conf.bak
sudo apt-get install --reinstall xserver-xorg-video-intel libgl1-mesa-glx libgl1-mesa-dri xserver-xorg-core
sudo dpkg-reconfigure xserver-xorg

# May not have any alternatives configured (ok to skip)
sudo update-alternatives --remove gl_conf /usr/lib/nvidia-current/ld.so.conf
# Does not change... still no keyboard

# Inactivate graph ics drivers repositories then ...
sudo dpkg -P $(dpkg -l | grep nvidia-driver | awk '{print $2}')
sudo apt autoremove
sudo apt install xserver-xorg-video-nouveau
```

## Posit Connect et learnr

Vous pouvez distribuer des tutoriels learnr via Posit Connect. C'erst bien pratique, car vos utilisateurs n'oint pas besoin d'avoir R installé localement pour exécuter ces learnrs. Cependant, il faut garder à l'esprit que cela introduit une brèche de sécurité énorme dans votre serveur. En effet, le tutoriel learnr permet l'exécution de n'importe quel code R sur le serveur. Vous devez donc cloisonner et sécuriser suffisamment le processus R qui est utilisé pour éviter à l'utilisateur d'accéder à des données ou des composants sensibles sur votre serveur. Voic comment faire :

- Installer AppArmor :

```
sudo apt-get install -y libapparmor-dev apparmor-utils
sudo R
install.packages("RAppArmor")
```

- Configurer le profil AppArmor :

```
#cd /usr/local/lib/R/site-library/RAppArmor/
cd /opt/R/4.0.5/lib/R/library/RAppArmor/
sudo cp -Rf profiles/debian/* /etc/apparmor.d/

# Verify the profile
sudo nano /etc/apparmor.d/rapparmor.d/r-user
# Make sure there is /usr/lib/rstudio/bin/pandoc/* rix,

#Load the profiles into the kernel
sudo service apparmor restart

#To disable enforcing the global R profile
sudo aa-disable usr.bin.r

#To start enforcing the standard R policy:
sudo aa-enforce usr.bin.r

# Set options for learnr tutorials
sudo nano /opt/R/4.0.5/lib/R/etc/Rprofile.site
```

- Ajouter ceci à la configuration du process R lancé par les learnrs :

```
options(tutorial.exercise.evaluator.onstart = function(pid) {
  # Import RAppArmor
  require(RAppArmor, quietly = TRUE)
  # Set process group to pid (allows kill of entire subtree in cleanup)
  setpgid();
  # Set nice priority
  setpriority(10)
  # Set rlimits as appropriate
  rlimit_nproc(1000)
  rlimit_as(100*1024*1024)
  # Change to r-user profile
  aa_change_profile("r-user")
})

options(tutorial.exercise.evaluator.oncleanup = function(pid) {
  # Import RAppArmor
  require(RAppArmor, quietly = TRUE)
  # Kill entire process subtree. note that the second call works
  # because the call to setpgid above sets our pgid (process group id)
  # to our pid (process id)
  kill(pid, tools::SIGKILL)
  kill(-1 * pid, tools::SIGKILL)
})
```

- Le fichier `r-user` profile contient :

```
## Do not modify this file. Changes may be undone during upgrades. 
## To play around, create a copy with a different profile name and
## put it in: /etc/apparmor.d/rapparmor.d/

profile r-user {
        #include <abstractions/base>
        #include <abstractions/nameservice>

        capability kill,
        capability setgid,
        capability net_bind_service,
        capability sys_tty_config,

        @{HOME}/ r,
        @{HOME}/.Rprofile r,
        @{HOME}/R/ r,
        @{HOME}/R/** rw,
        @{HOME}/R/{i686,x86_64}-pc-linux-gnu-library/** mrwix,

        @{PROC}/[0-9]*/attr/current r,

        /bin/* rix,
        /dev/tty r,
        /etc/R/ r,
        /etc/R/* r,
        /etc/fonts/** mr,
        /etc/resolv.conf r,
        /etc/xml/* r,
        /tmp/** mrwix,
        /usr/bin/* rix,
        /usr/include/** r,
        /usr/lib/gcc/** rix,
        /usr/lib/R/bin/* rix,
        /usr/lib{,32,64}/** mr,
        /usr/lib{,32,64}/R/bin/exec/R rix,
        /usr/local/lib/R/** mr,
        /usr/local/lib/R/site-library/** mrwix,
        /usr/local/share/** mr,
        /usr/share/** mr,
        /usr/share/ca-certificates/** r,
        /opt/rstudio-connect/ext/pandoc/** mrix,
        /opt/rstudio-connect/ext/pandoc/2.9/* mrix,
        /opt/rstudio-connect/ext/pandoc/2.11/* mrix,
        /opt/rstudio-connect/** mrwix,
        /opt/R/4.0.5/lib/R/** mr,
        /opt/R/4.0.5/lib/R/library/** mrwix,
        /opt/rstudio-connect/mnt/** mrwix,
        /opt/rstudio-connect/mnt/app/packrat/lib/x86_64-pc-linux-gnu/4.0.5/** mrwix,
}
```

- Redémarrer AppArmor avec le profil modifié :

```
sudo nano /etc/apparamor.d/rapparmor.d/r-user
sudo service apparmor restart
```

\BeginKnitrBlock{warning}<div class="warning">
Testez toujours vos tutoriels learnrs sur le serveur et tentez d'exécuter des commandes potentiellement problématiques comme accéder à des dossiers ou fichiers sensibles, ou exécuter des commandes système avec `system(...)` dans un exercice où il faut rentrer du code R. Ne mettez vos tutoriels learnr en production que si vous êtes certain•e d'avoir correctement sécurisé votre serveur à ce niveau !</div>\EndKnitrBlock{warning}

Sur les ordinateurs où vous développez vos learnrs, vous devez aussi installer les packages R suivants :
`install.packages(c("RAppArmor", "unix"))`

## Tâches CRON et autres

- Récupération des données depuis MongoDB ATLAS et backup des données (**note : remplacer d'abord les items sensibles comme <PASSWORD> ou <SERVER> par le mot de passe et le nom du serveur MongoDB Atlas !!!**) :

```
echo $'#!/bin/bash
# Recuperation of MongoDB ATLAS data into local MongoDB database
# must be executed as root
# Copyright (c) 2021, Philippe Grosjean (phgrosjean@sciviews.org)

# First check that major versions of MongoDB servers (ATLAS vs local, x.y)
# match (otherwise, we may get corrupted data!)
mongo_local_version=`mongo --quiet --eval "db.version()" | awk -F "." \'{print $1 "." $2}\'`
mongo_atlas_version=`mongo --quiet --eval "db.version()" "mongodb://<USER>:<PASSWORD>@<SERVER>.mongodb.net:27017,<SERVER2>.mongodb.net:27017,<SERVER3>.mongodb.net:27017/sdd?ssl=true&replicaSet=<SHARD>&authSource=admin" | awk -F "." \'{print $1 "." $2}\'`
if [ "$mongo_local_version" != "$mongo_atlas_version" ]; then
  echo "MongoDB versions mismatch: local=$mongo_local_version, ATLAS=$mongo_atlas_version" > /data1/VERSIONS_MISMATCH
  exit 0 # Use a higher value to report the error
else
  rm -f /data1/VERSIONS_MISMATCH
fi

# Make sure /data1/dump directory exists
cd /data1 &&
mkdir -p /data1/dump &&
chown <USER>:<GROUP> /data1/dump &&
# Record current date (in UTC format)
date -u "+%Y-%m-%d %H:%M:%S" > cur_date &&
# Is it a last_date file?
if [ -f "last_date" ]; then
  # Last date file exists, use it and get only data from that date
  # Need to specify collection here => call 3x for h5p, shiny & learnr
  mongodump --quiet --uri="mongodb://<USER>:<PASSWORD>@<SERVER>.mongodb.net:27017,<SERVER2>-umnnw.mongodb.net:27017,<SERVER3>.mongodb.net:27017/sdd?ssl=true&replicaSet=<SHARD>&authSource=admin" --db="sdd" --collection="h5p" --query="{\\\"date\\\":{\\\"\$gte\\\":\\\"$(cat last_date)\\\"}}"
  mongodump --quiet --uri="mongodb://<USER>:<PASSWORD>@<SERVER>.mongodb.net:27017,<SERVER1>.mongodb.net:27017,<SERVER2>.mongodb.net:27017/sdd?ssl=true&replicaSet=<SHARD>&authSource=admin" --db="sdd" --collection="shiny" --query="{\\\"date\\\":{\\\"\$gte\\\":\\\"$(cat last_date)\\\"}}"
  mongodump --quiet --uri="mongodb://<USER>:<PASSWORD>@<SERVER>.mongodb.net:27017,<SERVER2>.mongodb.net:27017,<SERVER3>.mongodb.net:27017/sdd?ssl=true&replicaSet=<SHARD>&authSource=admin" --db="sdd" --collection="learnr" --query="{\\\"date\\\":{\\\"\$gte\\\":\\\"$(cat last_date)\\\"}}"
else
  # No last_date file, get everything from MongoDB ATLAS
  mongodump --quiet --uri="mongodb://<USER>:<PASSWORD>@<SERVER>.mongodb.net:<SERVER2>-umnnw.mongodb.net:27017,<SERVER3>.mongodb.net:27017/sdd?ssl=true&replicaSet=<SHARD>&authSource=admin" --db="sdd"
fi

# If something is collected, an sdd subdirectory is created
if [ -d "/data1/dump/sdd" ]; then
  # Backup last_restore
  if [ -f "last_restore" ]; then
    cp last_restore forelast_restore
  fi
  # Backup last_date too
  if [ -f "last_date" ]; then
    cp last_date forelast_date
  fi
  mongorestore &> last_restore &&
  cp cur_date last_date
fi

# Clean up /data1/dump directory and go back to intial dir
rm -rf /data1/dump/* &&
cd - > /dev/null
' | sudo tee /usr/local/bin/get_atlas_data > /dev/null &&
sudo chmod 755 /usr/local/bin/get_atlas_data &&
sudo chmod +x /usr/local/bin/get_atlas_data
```

- Réaliser un backup quotidien roulant, ainsi qu'un backup permanent chaque samedi :

```
# Regular daily backup 
echo '#!/bin/bash
# Daily, temporary MongoDB database backup (overwritten next same day)
DIR=`date +%a` &&
DEST=/data1/backup/mongodb/$DIR &&
rm -rf $DEST &&
mkdir -p $DEST &&
mongodump -o $DEST &&
date -u "+%Y-%m-%d %H:%M:%S" > $DEST/backup_date
' | sudo tee /usr/local/bin/backup_data_daily > /dev/null &&
sudo chmod 755 /usr/local/bin/backup_data_daily &&
sudo chmod +x /usr/local/bin/backup_data_daily &&

# Weekly, more permanent backup
echo '#!/bin/bash
# Weekly, permanent MongoDB database backup
DIR=`date +%Y%m%d` &&
DEST=/data1/backup/mongodb/$DIR &&
mkdir -p $DEST &&
mongodump -o $DEST
' | sudo tee /usr/local/bin/backup_data_weekly > /dev/null &&
sudo chmod 755 /usr/local/bin/backup_data_weekly &&
sudo chmod +x /usr/local/bin/backup_data_weekly
```

Vérifier ces scripts et ensuite, ajouter une entrée crontab :

```
sudo crontab -e
```

Éditer le fichier en y rajoutant :

```
# Get data from MongoDB ATLAS every 5 min
*/5 * * * * /usr/local/bin/get_atlas_data
# Sync /data1/ with /data2/ every hour:47
47 * * * * /usr/bin/rsync -av --delete /data1/ /data2/ &> /data/datasync.log
# Create a daily (except saterday) and a weekly backup of the MongoDB database
# on 03:41:00
41 3 * * 6 /usr/local/bin/backup_data_weekly
41 3 * * 0,1,2,3,4,5 /usr/local/bin/backup_data_daily
```

## Maintenance régulière

Après un an, il faut mettre à jour le certificat de sécurité :

TODO: adapter ce scrit.

```
# Place sdd_umons_ac_be.cer|key in /etc/ssl/private. It seems both keys do not match:
sudo openssl x509 -noout -modulus -in /etc/ssl/private/sdd_umons_ac_be.cer | openssl md5
sudo openssl rsa -noout -modulus -in /etc/ssl/private/sdd_umons_ac_be.key | openssl md5
# Maybe the PKCS#7 format is correct? So, the .crt with that format can be converted this way:
sudo openssl pkcs7 -print_certs -in /etc/ssl/private/sdd_umons_ac_be.crt -out /etc/ssl/private/sdd_umons_ac_be.pem -outform PEM
sudo chmod o-r /etc/ssl/private/sdd_umons_ac_be.pem
sudo openssl x509 -noout -modulus -in /etc/ssl/private/sdd_umons_ac_be.pem | openssl md5
# Now, this is correct!

# Update the certificate: I receive a .zip file with both sdd_umons_ac_be.key and sdd_umons_ac_be.cer.
# These files must be placed into /etc/ssl/private on the sdd server:
# From MacOS
scp ~/Desktop/sdd_certs_2021.zip econum@sdd.umons.ac.be:/data1/dump/cert.zip
# Create .pem file that invert the order of the 4 certificates from the .cer file, then...
scp ~/Desktop/sdd_umons_ac_be.pem econum@sdd.umons.ac.be:/data1/dump/sdd_umons_ac_be.pem
# From sdd
sudo ls -l /etc/ssl/private
sudo mv /etc/ssl/private/sdd_umons_ac_be.cer /etc/ssl/private/sdd_umons_ac_be.cer.old
sudo mv /etc/ssl/private/sdd_umons_ac_be.key /etc/ssl/private/sdd_umons_ac_be.key.old
unzip /data1/dump/cert.zip
rm /data1/dump/cert.zip
sudo mv sdd_umons_ac_be.* /etc/ssl/private
sudo chown root:root /etc/ssl/private/sdd_umons_ac_be.cer
sudo chown root:root /etc/ssl/private/sdd_umons_ac_be.key
sudo chmod 640 /etc/ssl/private/sdd_umons_ac_be.cer
sudo chmod 640 /etc/ssl/private/sdd_umons_ac_be.key
sudo mv /data1/dump/sdd_umons_ac_be.pem /etc/ssl/private/sdd_umons_ac_be.pem
sudo chown root:root /etc/ssl/private/sdd_umons_ac_be.pem
sudo chmod 640 /etc/ssl/private/sdd_umons_ac_be.pem
```

Mise à jour de R et RStudio Connect (exemple d'&ncien script à adapter et expliciter) :

```
# Current RStudio Connect is 1.8.2.1-12
ls /opt/R # See R versions already installed (3.4.4, 3.5.3, 3.6.3)
export R_VERSION=4.0.5
curl -O https://cdn.rstudio.com/r/ubuntu-1804/pkgs/r-${R_VERSION}_1_amd64.deb
sudo gdebi r-${R_VERSION}_1_amd64.deb
/opt/R/${R_VERSION}/bin/R --version # verification
sudo ln -s /opt/R/${R_VERSION}/bin/R /usr/local/bin/R
sudo ln -s /opt/R/${R_VERSION}/bin/Rscript /usr/local/bin/Rscript
# Install R package
sudo R
repos <- getOption("repos")
repos["CRAN"] <- "https://mran.microsoft.com/snapshot/2021-05-17"
options(repos = repos)
options(timeout = 300)

update.packages()

install.packages(c("tidyverse", "shiny", "bookdown", "data.table", "glue",
"here", "mongolite", "keras", "pastecs", "mlearning", "usethis", "testthat",
"covr", "sessioninfo", "reticulate", "remotes", "devtools", "knitr",
"latticeExtra", "inline", "Hmisc", "gridExtra", "gridGraphics", "ggsci",
"ggpubr", "GGally", "ggplotify", "ggrepel", "cowplot", "fs", "forcats", "purrr",
"R6", "RColorBrewer", "Rcpp", "anytime", "zoo", "assertthat", "bench", "hms",
"lubridate", "rsconnect", "RSQLite", "sos", "styler", "vctrs", "viridis",
"viridisLite", "withr", "xts", "igraph", "pryr", "proto", "renv", "tsibble",
"SciViews", "svMisc", "svGUI", "svDialogs", "extraDistr", "SuppDists", "lobstr",
"import", "miniUI", "vegan", "shinydashboard"))

install.packages("BiocManager")
BiocManager::install(c("graph", "ComplexHeatmap", "Rgraphviz",
  "RDRToolbox"), update = FALSE, ask = FALSE)

remotes::install_github("SciViews/mlearning@v1.0.6", force = TRUE)
# Fails! remotes::install_github("SciViews/tcltk2@v1.3.0", force = TRUE)
remotes::install_github("SciViews/svMisc@v1.2.0", force = TRUE)
remotes::install_github("SciViews/svGUI@v1.0.1", force = TRUE)
remotes::install_github("SciViews/svDialogs@v1.0.3", force = TRUE)
remotes::install_github("SciViews/svSweave@v1.0", force = TRUE)
remotes::install_github("SciViews/flow@v1.1.0", force = TRUE)
remotes::install_github("SciViews/data.io@v1.3.0", force = TRUE)
remotes::install_github("SciViews/chart@v1.3", force = TRUE)
remotes::install_github("SciViews/SciViews@v1.1.1", force = TRUE)

remotes::install_github("phgrosjean/pastecs@v1.4.0", force = TRUE)
remotes::install_github("phgrosjean/aurelhy@v1.0.8", force = TRUE)

remotes::install_github("rstudio/gradethis@ced5541")
remotes::install_github("r-lib/itdepends@f8d012b")

# Warning in svbox2021, we don't use learndown anymore, but learnitdown instead!
remotes::install_github("SciViews/learnitdown@v1.3.0")
#remotes::install_github("BioDataScience-Course/BioDataScience@...")
#remotes::install_github("BioDataScience-Course/BioDataScience1@...")
#remotes::install_github("BioDataScience-Course/BioDataScience2@...")
#remotes::install_github("BioDataScience-Course/BioDataScience3@...")

# Check and update license
sudo /opt/rstudio-connect/bin/license-manager status
# (re)activate license
sudo /opt/rstudio-connect/bin/license-manager deactivate 
sudo /opt/rstudio-connect/bin/license-manager activate YAG3-B2H5-MXQB-6HJS-E7WH-4G9Y-JITA
sudo systemctl restart rstudio-connect

# Upgrade RStudio Connect to version 
curl -Lo rsc-installer.sh https://cdn.rstudio.com/connect/installer/installer-v1.9.1.sh
sudo -E bash ./rsc-installer.sh 1.8.8.2
```

Installation d'une nouvelle version de Python :

```
sudo mkdir /opt/python
sudo curl -fsSL -o /opt/python/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh 
sudo chmod 755 /opt/python/miniconda.sh
sudo /opt/python/miniconda.sh -b -p /opt/python/miniconda

export PYTHON_VERSION="3.8.5"

sudo /opt/python/miniconda/bin/conda create --quiet --yes \
      --prefix /opt/python/"${PYTHON_VERSION}" \
      --channel conda-forge \
      python="${PYTHON_VERSION}"  "pip<20.1"

# Check
/opt/python/"${PYTHON_VERSION}"/bin/python --version
```

Installation de packages Python supplémentaires :

```
sudo /opt/python/"${PYTHON_VERSION}"/bin/pip install altair beautifulsoup4 cloudpickle \
  cython dask gensim keras matplotlib nltk numpy pandas pillow pyarrow \
  requests scipy scikit-image scikit-learn scrapy seaborn spacy sqlalchemy \
  statsmodels tensorflow xgboost

# Add python to the system path (optional)
# add PATH=/opt/python/"${PYTHON_VERSION}"/bin:$PATH in /etc/profile.d/python.sh

# Make Python available as Jupyter Kernel (optional)
#sudo /opt/python/${PYTHON_VERSION}/bin/pip install ipykernel
#sudo /opt/python/${PYTHON_VERSION}/bin/python -m ipykernel install --name py${PYTHON_VERSION} --display-name "Python ${PYTHON_VERSION}"

# Make Python available to reticulate
nano ~/.Rprofile # Add Sys.setenv(RETICULATE_PYTHON = "/opt/python/3.8.5/bin/python")
```
