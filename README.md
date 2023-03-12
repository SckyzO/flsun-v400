# FLSun V400

Vous trouverez ici toutes mes informations pour améliorer votre FLsun V400.

Je vais décrire toutes les étapes que j'ai réalisé pour mettre à jour ma machine avec la derniere version de klipper / moonraker / ubuntu ... etc. 

## Les liens utiles

### Documentation officielle

- Klipper
- Moonraker
- Github de Guilouz pour tout ce qui tourne autour de la FLSun
- Forum lesimprimantes3D.com section FLSun


### Documentation spécifique

- Calibrer votre extrudeur : https://www.klipper3d.org/Rotation_Distance.html
- Régler le Pressure Advance : https://www.klipper3d.org/Pressure_Advance.html
- Ajuster manuellement la compensation de résonance (Resonance Compensation): https://www.klipper3d.org/Resonance_Compensation.html
- Mesurer les résonances avec ADXL : https://www.klipper3d.org/Measuring_Resonances.html
- Pour utiliser la fonction Exclude Objects: https://docs.mainsail.xyz/features/exclude_objects
- Pour utiliser la fonction Timelapse: https://github.com/mainsail-crew/moonraker-timelapse/blob/main/docs/configuration.md
- Pour afficher les vignettes à l'écran: https://klipperscreen.readthedocs.io/en/latest/Thumbnails/
- Thème de Mainsail pour le Speeder Pad: 

## Première étape, hack de la speeder pad 

Le Speeder Pad livré avec la FLSun ne permet pas de se connecter en SSH par défaut avec l'utlisateur pi. Guilouz propose un tuto pour réinstaller la tablette, mais je trouve cette procédure dommage quand on sait que notre tablette est déjà installée et fonctionnelle. Celle ci tourne sous Ubuntu 20.04 LTS. J'ai trouvé le repo de amoutiers la procédure pour forcer la mise à jour du mot de passe sans réinstaller le système. 

Pour cela rien de plus simple, vous allez sur son [repo github](https://github.com/amoutiers/FLSun-Speeder-Pad-root-password) , vous téléchargez le fichier update.bin, vous le mettez à la racine d'une clé USB formatée en FAT32. Vous redémmarer le speeder pad. Un message indiquera qu'il y a une mise à jour en cours et le speeder pad redémarrera. Une fois fait, vous pouvez vous connecter en SSH à votre speed pad (user: pi, password: flsun). Vous pouvez le changer par la suite cela n'a pas d'importance sur le système. Retirer la clé USB et maintenant nous allons passer au reste de la procédure. 

## Configurer Ubuntu

### Update d'ubuntu

Maintenant que nous pouvons nous connecter à notre système en SSH (si vous n'êtes pas familier avec le SSH et la ligne de commande, je vous recommande avant de suivre la procédure d'apprendre un peu, car moi j'en bouffe à longueur de journée mais pour les néophytes c'est pas pareil :smile: )

```bash
sudo apt update
sudo apt upgrade -y
sudo apt autoclean
```

### Installation des packages nécessaires

```bash
sudo apt install vim
```

### Configuration de l'heure

Etonnament, il n'est pas possible de changer l'heure sur la tablette, ce qui est complètement débile je trouve. Du coup il va falloir changer la timezone pour que l'heure de Paris soit pris en charge. 

```bash
sudo timedatectl set-timezone Europe/Paris
```

### Changer la zone du Wifi pour celle de la France

```bash
iw reg set FR
```

Activer ce changement une fois pour toute, il faut ajouter l'entrée dans le fichier `/etc/rc.local`. Taper la commande suivante et rebooter: 

```bash
sudo sed -i 's/^exit 0/iw reg set FR\nexit 0/g' /etc/rc.local
```

## Configurer Klipper

Dans un premier temps on va sauvegarder toutes nos données "au cas ou". 

```bash
cd ~
tar czf backup-klipper-flsun-v400.tar.gz moonraker* mainsail/ klipp* .moonraker_database/ KlipperScreen/ .KlipperScreen-env/ mjpg-streamer/ savedVariables.cfg
```

Vous devriez avoir un fichier `backup-klipper-flsun-v400-original.tar.gz`. Garder le précieusement "au cas ou" :) 

## Nettoyer les anciennes versions

Pour cela nous allons utiliser le script kiauh

```bash
cd ~
git clone https://github.com/th33xitus/kiauh.git
./kiauh/kiauh.sh
```

Puis nous allons taper les directives suivantes 

```
3) [Remove]
5) [KlipperScreen]
3) [Mainsail]
2) [Moonraker]
1) [Klipper]
B) « Back
Q) Quit
```

Puis nous allons finir de nettoyer manuellement les restes 
```
rm -Rf .moonraker_database/ klipper_* mjpg-streamer/ moonraker-timelapse/ savedVariables.cfg
```

## Installer Klipper et tout l'environnement

Toujours avec l'outil kiauh

```bash
./kiauh/kiauh.sh
1) [Install]
1) [Klipper]
1) [Python 3.x]  (recommended)
Number of Klipper instances to set up: 1
2) [Moonraker]
3) [Mainsail] (ne pas installer les macros)
B) « Back
Q) Quit
```

Puis on installe KlipperScreen depuis le repo de Guilouz

```
git clone https://github.com/Guilouz/KlipperScreen-Flsun-Speeder-Pad.git
sudo mv /home/pi/KlipperScreen-Flsun-Speeder-Pad /home/pi/KlipperScreen
cd ~/KlipperScreen
./scripts/KlipperScreen-install.sh
```