# TP #6 - Installation de gitlab-runner

> **Objectifs du TP**
> * Installer Gitlab-Runner via le dépôt officiel de la distribution
> * Enregistrer le runner


## Installer Gitlab-Runner via le dépôt officiel de la distribution

Si vous avez réalisé le TP4 sur le débug de pipelines en local vous devriez déjà avoir installé `gitlab-runner` .
Vous pouvez donc passer directement à la partie suivante.

Ajouter le repository gitlab-runner :

```bash
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
```

Installer le package gitlab-runner :
```bash
$ sudo apt-get install gitlab-runner
```

## Enregistrer un specific runner

Dans le projet “sample-app” que nous avons utilisé sur les premiers TPs, cliquez sur “Settings > CI/CD > Runners” pour afficher le token :

<img src="https://i.ibb.co/YkTqxmy/Screenshot-2021-01-13-at-10-47-34.png">

Exécuter :
```bash
$ sudo gitlab-runner register
```
Saisir l’url de gitlab :
```bash
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
https://gitlab.com/
```
Saisir le token récuperer dans l'interface gitlab :  
```bash
Please enter the gitlab-ci token for this runner:

```
Saisir “Specific runner of sample-app project” pour la description du runner :
```bash
Please enter the gitlab-ci description for this runner:
[ip-172-31-1-237]: Specific runner of sample-app project
```

Saisir “” pour les tags du runner pour ce TP mais je vous rappelle que les tags peuvent être utilisés pour définir sur quel runner doit être exécuté un job :
```bash
Please enter the gitlab-ci tags for this runner (comma separated):

```
Saisir **docker** pour l’exécuteur :

```bash
Please enter the executor: shell, virtualbox, docker-ssh+machine, custom, docker, docker-ssh, kubernetes, parallels, ssh, docker+machine:
docker
```

Saisir **centos7** pour l’image docker à utiliser par l’exécuteur :
```bash
Please enter the default Docker image (e.g. ruby:2.6):
centos7
```

Vérifier que le runner est enregistré :	
 
- Ce message doit s’afficher :
```bash
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```
- Dans le projet, cliquez sur “Settings > CI/CD > Runners”, le runner doit apparaître dans la liste des runners actifs du projet :

<img src="https://i.ibb.co/W3PtzJK/Screenshot-2021-01-13-at-10-47-42.png">



