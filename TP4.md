# TP #4 - Debug d'une pipeline Gitlab-CI

> **Objectifs du TP**
> * Découvrir comment débugguer une pipeline pour réduire la boucle de feedback
> * Détecter facilement les erreurs de syntaxe avec le Linter intégré dans Gitlab

Vous avez probablement déjà constaté qu'il est assez difficile d'écrire un nouveau job du premier coup sans erreur.
Nous vous avons dit qu'il fallait toujours chercher à réduire la boucle de feedback: debug le pipeline en local est un moyen d'y parvenir ici.
Plusieurs solutions existent, nous vous en proposons 2 ici:

# Vérification et correction de la syntaxe de votre fichier gitlab-ci.yml

Toutes les instances de Gitlab (Gitlab.com ou votre Gitlab instancié) proposent un outil pour vérifier la syntaxe de vos fichiers `.gitlab-ci.yml`

Pour trouver l'outil de vérification de syntaxe Gitlab-CI:
- aller sur un projet qui contient une pipeline
- cliquer sur le menu gauche "CI/CD" puis sur "Pipelines"
- le bouton CI-Lint permet d'afficher le Linter

Maintenant que vous êtes sur la page de vérification de la syntaxe, utilisez-la pour corriger la syntaxe du fichier suivant :

```yaml
variables:
  PYTHON3_IMAGE: python:alpine

stages:
  - syntax

PyCodeStyle:
  image: $PYTHON3_IMAGE
  stage: syntaxe
  script:
    - pip install pycodestyle
    - pycodestyle app.py

Pylint:
  image: $PYTHON3_IMAGE
  stage: syntax
  script:
 - pip install pylint
 - pylint app.py
```

> **Tips**: il va vous falloir adapter au moins 3 lignes

# Reproduction d'une pipeline en local

Comme présenté avec les slides, les runners gitlab peuvent être de plusieurs types (docker, local, ...). Nous allons installer ici directement
le binaire `gitlab-runner` sur l'environnement local afin de pouvoir l'utiliser pour tester notre pipeline.

Exécutez la procédure suivante pour l'installer sur Ubuntu et vous trouverez ici pour les autres environnements (https://docs.gitlab.com/runner/install/) :

```bash
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
$ sudo apt-get install gitlab-runner=12.6.0
```

Testez la bonne installation du binaire avec la commande suivante:

```
$ gitlab-runner
```

Vous devriez avoir un résultat similaire à celui-ci:

```
$ gitlab-runner
Runtime platform                                    arch=amd64 os=linux pid=23738 revision=ac8e767a version=12.6.0
NAME:
   gitlab-runner - a GitLab Runner

USAGE:
   gitlab-runner [global options] command [command options] [arguments...]

VERSION:
   12.6.0 (ac8e767a)

AUTHOR:
   GitLab Inc. <support@gitlab.com>

COMMANDS:
     exec                  execute a build locally
     list                  List all configured runners
     run                   run multi runner service
     register              register a new runner
     install               install service
     uninstall             uninstall service
     start                 start service
     stop                  stop service
     restart               restart service
     status                get status of a service
     run-single            start single runner
     unregister            unregister specific runner
     verify                verify all registered runners
     artifacts-downloader  download and extract build artifacts (internal)
     artifacts-uploader    create and upload build artifacts (internal)
     cache-archiver        create and upload cache artifacts (internal)
     cache-extractor       download and extract cache artifacts (internal)
     cache-init            changed permissions for cache paths (internal)
     health-check          check health for a specific address
     help, h               Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --cpuprofile value           write cpu profile to file [$CPU_PROFILE]
   --debug                      debug mode [$DEBUG]
   --log-format value           Choose log format (options: runner, text, json) [$LOG_FORMAT]
   --log-level value, -l value  Log level (options: debug, info, warn, error, fatal, panic) [$LOG_LEVEL]
   --help, -h                   show help
   --version, -v                print the version
```

Créez un nouveau dossier **autre part** que dans sample-app, une fois dedans ajoutez un fichier `app.py` avec comme contenu le même que celui de sample-app. Plus qu'à utiliser la commande suivante `git init && git add . && git commit -m "Initial commit"`. Vous pouvez maintenant créer le fichier .gitlab-ci.yml suivant comme nous avons vu dernièrement : 

```yaml
variables:
  PYTHON3_IMAGE: python:alpine

stages:
  - syntax

PyCodeStyle:
  image: $PYTHON3_IMAGE
  stage: syntaxe
  script:
    - pip install pycodestyle
    - pycodestyle app.py

Pylint:
  image: $PYTHON3_IMAGE
  stage: syntax
  script:
    - pylint app.py
```

Exécutez maintenant le pipeline en local: 
où (exec docker fait référence à l'executor docker que nous avons vu en cours et PyCodeStyle étant le job à tester, cette valeur peut bien évidemment changer selon le test que vous souhaitez réaliser)

```bash
gitlab-runner exec docker PyCodeStyle
```

Vous devriez avoir un résultat similaire à celui-ci :
```bash
ubuntu@samuelantunes:~/test$ gitlab-runner exec docker PyCodeStyle
Runtime platform                                    arch=amd64 os=linux pid=22294 revision=ac8e767a version=12.6.0
WARNING: You most probably have uncommitted changes.
WARNING: These changes will not be tested.
Running with gitlab-runner 12.6.0 (ac8e767a)
Using Docker executor with image python:alpine ...
Pulling docker image python:alpine ...
Using docker image sha256:f7756628c1ee047c3b9246fe4eea25880386c26ed46c5bc5af11fddc90e91771 for python:alpine ...
Running on runner--project-0-concurrent-0 via samuelantunes...
Fetching changes...
Initialized empty Git repository in /builds/project-0/.git/
Created fresh repository.
From /home/ubuntu/test
 * [new branch]      master     -> origin/master
Checking out feccd64d as master...

Skipping Git submodules setup
$ pylint app.py

sh: 1: pylint not found 

ERROR: Job failed: exit code 1
FATAL: exit code 1]
```

A notre grand regret, nous constatons ici que le job n'est pas correct. Il semblerait que la dépendance "pytling" ne soit pas installée.

> Question
>
> Corriger maintenant le fichier .gitlab-ci.yml pour faire fonctionner le job

Après correction, vous devriez avoir un résultat similaire à celui-ci:
```bash
ubuntu@samuelantunes:~/test$ gitlab-runner exec docker PyCodeStyle
Runtime platform                                    arch=amd64 os=linux pid=22294 revision=ac8e767a version=12.6.0
WARNING: You most probably have uncommitted changes.
WARNING: These changes will not be tested.
Running with gitlab-runner 12.6.0 (ac8e767a)
Using Docker executor with image python:alpine ...
Pulling docker image python:alpine ...
Using docker image sha256:f7756628c1ee047c3b9246fe4eea25880386c26ed46c5bc5af11fddc90e91771 for python:alpine ...
Running on runner--project-0-concurrent-0 via samuelantunes...
Fetching changes...
Initialized empty Git repository in /builds/project-0/.git/
Created fresh repository.
From /home/ubuntu/test
 * [new branch]      master     -> origin/master
Checking out feccd64d as master...

Skipping Git submodules setup
$ pylint app.py

Your code has been rated at 10.00/10 (previous run: 5.00/10, +5.00)

Job succeeded
```

Et voilà comment faire vos tests sans spam vos pipelines (et du coup vos crédits de consommation de temps d'utilisation gratuits) directement sur votre poste.
