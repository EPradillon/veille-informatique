# Notes liées au packages d'ubuntu
## sudo apt-get update

### réagir face a des erreurs de package
```sh
grep -r --include '*.list' '^deb ' /etc/apt/sources.list /etc/apt/sources.list.d/
```
>par exemple j'avais eu un soucis avec jonatamachin python3.6 donc j'ai commenté la ligne dans le fichier de la réponse à la commande.
>
> -- <cite>[source](https://dev.to/deepika_banoth/how-i-solved-failed-to-fetch-http-ppa-launchpad-net-403-forbidden-2544)</cite>
