# Notes liées au packages d'ubuntu
## sudo apt-get update

### Erreur de mise a jour :
Au début le problème venait du logiciel de mise à jour.
Mais j'ai remarqué que la commande `sudo apt update` me retournait :
```sh
E: Impossible de récupérer http://ppa.launchpad.net/jonathonf/python-3.6/ubuntu/dists/bionic/InRelease  403  Forbidden [IP : 2001:67c:1560:8008::19 80]
E: Le dépôt http://ppa.launchpad.net/jonathonf/python-3.6/ubuntu bionic InRelease n'est plus signé
N: Les mises à jour depuis un tel dépôt ne peuvent s'effectuer de manière sécurisée, et sont donc désactivées par défaut.
N: Voir les pages de manuel d'apt-secure(8) pour la création des dépôts et les détails de configuration d'un utilisateur.
W: Une erreur s'est produite lors du contrôle de la signature. Le dépôt n'est pas mis à jour et les fichiers d'index précédents seront utilisés. Erreur de GPG : https://pkg.jenkins.io/debian-stable binary/ Release : Les signatures suivantes n'ont pas pu être vérifiées car la clé publique n'est pas disponible : NO_PUBKEY FCEF32E745F2C3D5
ubuntu@ubuntu-N13-N140ZU:~$ grep -r --include '*.list' '^deb ' /etc/apt/sources.list /etc/apt/sources.list.d/
```

Apparement une erreur de repository.  
J'ai fait un grep de tous les 'binary sources' actifs :
```sh
grep -r --include '*.list' '^deb ' /etc/apt/sources.list /etc/apt/sources.list.d/
```
Qui m'a retourné :
```
/etc/apt/sources.list:deb http://security.ubuntu.com/ubuntu bionic-security multiverse
/etc/apt/sources.list:deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable
/etc/apt/sources.list:deb https://dl.winehq.org/wine-builds/ubuntu/ bionic main
/etc/apt/sources.list.d/ondrej-ubuntu-php-bionic.list:deb http://ppa.launchpad.net/ondrej/php/ubuntu bionic main
/etc/apt/sources.list.d/jonathonf-ubuntu-python-3_6-bionic.list:deb http://ppa.launchpad.net/jonathonf/python-3.6/ubuntu bionic main
/etc/apt/sources.list.d/jenkins.list:deb https://pkg.jenkins.io/debian-stable binary/
/etc/apt/sources.list.d/deadsnakes-ubuntu-ppa-bionic.list:deb http://ppa.launchpad.net/deadsnakes/ppa/ubuntu bionic main
/etc/apt/sources.list.d/pgdg.list:deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main
/etc/apt/sources.list.d/mongodb-org-4.2.list:deb [arch=amd64] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse
/etc/apt/sources.list.d/google-chrome.list:deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main
/etc/apt/sources.list.d/nodesource.list:deb https://deb.nodesource.com/node_12.x bionic main
/etc/apt/sources.list.d/remmina-ppa-team-ubuntu-remmina-next-bionic.list:deb http://ppa.launchpad.net/remmina-ppa-team/remmina-next/ubuntu bionic main
/etc/apt/sources.list.d/microsoft-prod.list:deb [arch=amd64] https://packages.microsoft.com/ubuntu/18.04/prod bionic main
```

Comme pour moi l'erreur viennait de `http://ppa.launchpad.net/jonathonf/python-3.6/ubuntu` j'ai commenté le contenu de
```sh
sudo vim /etc/apt/sources.list.d/jonathonf-ubuntu-python-3_6-bionic.list
```
Ensuite `sudo apt update` a réglé magiquement le reste des problèmes.

>J'avais deux erreurs et cette technique a rélgé les deux en ajoutant simplement un `#` devant les lignes
