## Création de deux partitions avec fdisk:

fdisk /dev/sd?

n (pour nouvelle partition)
p (pour primaire)
Entrée (pour partition 1)
Entrée (pour début)
+2G (pour taille)

n (pour nouvelle partition)
p (pour primaire)
Entrée (pour partition 1)
Entrée (pour début)
+2G (pour taille)

w (pour écrire)

## Maintenant les deux partitions sont créées

Le point de montage du FileSystem applicatif sera /application, le FileSystem devra faire 1 Go et son volume devra s'appeler lvData.
Le point de montage du FileSystem de sauvegarde sera /sauvegarde, le FileSystem devra faire 256 Mo et son volume devra s'appeler lvSauvegarde.

## Création des répertoires

mkdir /application
mkdir /sauvegarde

## pvcreate ne marche pas car pas installé par défaut (peut-être la meme chose sur la machine)

apt-get update
apt-get upgrade -y
apt-get install lvm2 -y

## Il faut créer les Physical Volumes (PV) sur les partitions créées précédemment.

pvcreate /dev/sd?1
pvcreate /dev/sd?2

## Il faut créer les Volume Groups (VG) sur les Physical Volumes créés précédemment.

vgcreate vgData /dev/sd?1
vgcreate vgSauvegarde /dev/sd?2

## Il faut créer les Logical Volumes (LV) sur les Volume Groups créés précédemment.

lvcreate -L 1G -n lvData vgData
lvcreate -L 256M -n lvSauvegarde vgSauvegarde

## Il faut formater les Logical Volumes créés précédemment.

mkfs.ext4 /dev/vgData/lvData
mkfs.ext4 /dev/vgSauvegarde/lvSauvegarde

## Il faut monter les Logical Volumes créés précédemment.

mount /dev/vgData/lvData /application
mount /dev/vgSauvegarde/lvSauvegarde /sauvegarde

## Les FileSystems /application et /sauvegarde devront être montés automatiquement au démarrage de la machine

/dev/mapper/vgData-lvData /sauvegarde ext4 defaults 0 2
/dev/mapper/vgSauvegarde-lvSauvegarde /sauvegarde ext4 defaults 0 2

## Pour la partie où il faut créer un utilisateur

sudo adduser admin

## Mettre l'admin en tant que sudoer

sudo usermod -aG sudo admin

## Générer la clé SSH

ssh-keygen

## Aller dans /etc/sudoers/admin et ajouter :

admin ALL=(ALL) NOPASSWD: /usr/bin/last -n 10
admin ALL=(ALL) NOPASSWD: /usr/bin/tail -n 15 /var/log/syslog
