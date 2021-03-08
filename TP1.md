# TP #1 - Premier pipeline

> **Objectifs du TP**
> * Ecrire un premier pipeline permettant de valider la conformité du code.
> * Apprendre et manipuler `.gitlab-ci.yml`
>
> **Prérequis**
> Avant de commencer ce TP, vous devez avoir satisfait les prérequis suivants
> * Vous avez validé votre accès à Gitlab
> * Vous avez ajouté votre clé SSH a votre compte
>

## Fork du projet

Rendez-vous à l'URL suivante : `https://gitlab.com/samuelantunes/sample-app` où vous trouverez un projet contenant une application d'exemple que nous allons utiliser pendant les TPs.

Cliquez sur le bouton "Fork" en haut de la page du projet afin de créer une copie du projet sur votre compte.
Vous devriez arriver sur la page de la copie du projet.

Sur la page d'accueil du projet et cliquez sur le bouton `Clone` en haut de la page et copiez l'URL
`Clone with SSH`.
Collez ensuite l'URL dans la console en ajoutant `git clone` devant l'URL :

```bash
$ git clone git@gitlab.com:<votre_login>/sample-app.git
```

Dans tous les TPs qui suivent vous travaillerez sur ce Fork du projet qui vous permettra de travailler en isolation.

## Création d'un pipeline avec un job

Nous allons créer notre premier pipeline. Pour cela rien de plus simple, il suffit de créer un fichier `.gitlab-ci.yml` à la racine du projet Git que vous venez de cloner sur votre poste.
Le pipeline ne contiendra qu'un seul job, qui consiste à analyser la conformité du code avec les règles PEP8 (le standard en Python).

Créons le fichier `.gitlab-ci.yml` suivant:

```yaml
variables:
  PYTHON3_IMAGE: python:alpine

stages:
  - syntax

PyCodeStyle:
  image: $PYTHON3_IMAGE
  stage: syntax
  script:
    - pycodestyle app.py
```

Ce pipeline défini une étape `syntax` qui execute un job `PyCodeStyle` exécutant la commande `pycodestyle`.
Ajoutez ce fichier à vos sources en créant une branche et en effectuant un commit :

```bash
$ git add .gitlab-ci.yml
$ git checkout -b add_pipeline_conformity
$ git commit -m "Ajout d'un pipeline testant la conformité pep8"
```

Envoyez votre code en revue en poussant votre branche sur gitlab:

```bash
$ git push origin add_pipeline_conformity
```
La commande `push` vous renvoie une URL permettant la création automatique d'une Merge Request, exemple :

```
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 1.17 KiB | 1.17 MiB/s, done.
Total 4 (delta 0), reused 0 (delta 0)
remote:
remote: To create a merge request for add_pipeline_conformity, visit:
remote:   [...]/merge_requests/new?merge_request%5Bsource_branch%5D=add_pipeline_conformity
remote:
To [...]/test-gitlab-tp.git
 * [new branch]      add_pipeline_conformity -> add_pipeline_conformity
```

Cliquez sur le lien qui vous amène sur un formulaire de création de Merge Request sur Gitlab et validez la création.

**Attention !** Lors de la création de la Merge Request, Gitlab vous proposera automatiquement de faire la MR sur
le projet d'origine et non pas votre Fork, il faudra donc corriger la branche de destination pour utiliser votre fork !

> Question 1.1
>
> - Pourquoi le pipeline a t-il échoué ?

Corrigez les erreurs remontées dans la console du job `PyCodeStyle` et effectuez un nouveau changement :

```bash
$ git add <lefichier>
$ git commit -m "Corrections pep8"
$ git push origin add_pipeline_conformity
```

## Création d'un second job parallele

Il existe d'autres linter que `pycodestyle` pour Python, `pylint` est un autre linter populaire.
Nous pourrions utiliser les 2 outils et exécuter les deux jobs en parallele, pour cela éditons notre fichier `.gitlab-ci.yml` :

```yaml
variables:
  PYTHON3_IMAGE: python:alpine

stages:
  - syntax

PyCodeStyle:
  image: $PYTHON3_IMAGE
  stage: syntax
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

Envoyez votre code en revue en poussant votre code sur une branche :

```bash
$ git add .gitlab-ci.yml
$ git commit -m "Ajout d'un job testant la conformité pylint"
$ git push origin add_pipeline_conformity
```

> Question 1.2
>
> - Pourquoi le pipeline a t-il échoué ?
>

Le dépot que nous avons cloné contient un fichier `requirements.txt` qui contient la liste des dépendances
requises par notre application. Les erreurs renvoyées par Pylint nous informent que ces dépendances ne sont pas présentent
lors de l'execution de notre test, il nous faut donc installer ces dépendances.

En Python l'installation de librairies est possible avec la commande `pip` que nous avons déjà utilisée dans notre pipeline.
Il est également possible d'utiliser `pip` en lui donnant un fichier contenant la liste des dépendances :

```bash
$ pip3 install -r requirements.txt
```

Modifiez le pipeline pour utiliser cette commande à la place de nos appels pip précédents :

```yaml
variables:
  PYTHON3_IMAGE: python:alpine

stages:
  - syntax

PyCodeStyle:
  image: $PYTHON3_IMAGE
  stage: syntax
  script:
    - pip install -r requirements.txt     # < ici
    - pycodestyle app.py

Pylint:
  image: $PYTHON3_IMAGE
  stage: syntax
  script:
    - pip install -r requirements.txt     # < ici
    - pylint app.py
```

et effectuez un nouveau changement :

```bash
$ git add <lefichier>
$ git commit -m "Correction pip"
$ git push origin add_pipeline_conformity
```

Le pipeline ne doit maintenant plus renvoyer d'erreurs et en regardant le détail du job Pylint vous devriez avoir
un score de 10/10 !

## Un test orienté sécurité ?

Un test en lien avec la sécurité est possible a cette étape du developpement (et oui !). Ajoutons un test qui fera plaisir
à votre équipe de sécurité, en ajoutant l'execution de `bandit` (un outil d'analyse statique du code Python) dans un autre job.
Ce job ne sera pas executé en parallèle des autres tests et nous allons ajouter une étape dédiée au pipeline appelée `sast` :

```yaml
stages:
  - syntax
  - sast

[...]

Static Code Analysis Tests:
  stage: sast
  image: $PYTHON3_IMAGE
  script:
    - pip install -r requirements.txt
    - bandit app.py
```

**Note :** SAST est l'accronyme signifiant Static Application Security Testing (SAST).

Ajoutez vos modifications (en utilisant la version `1.3.0`) dans un commit et poussez les sur votre branche :

```bash
$ git add .
$ git commit -m "Ajout d'un test SAST"
$ git push origin add_pipeline_conformity
```

> Question 1.3
>
> - Pourquoi le pipeline a t-il échoué ?
>

Vous devriez être capable de corriger seuls le problème. Effectuez le correctif et soumettez le changement
à Gitlab.

## Création de tests unitaires pour notre application

Créez le fichier de tests suivant:

```python
# tests/test_sample1.py
from flask import request
from app import APP

def test_version():
    with APP.test_request_context('/version', method='GET'):
        assert request.path == '/version'
        assert request.method == 'GET'
```

Pour executer les tests, vous aurez besoin de la commande `pytest` que vous pouvez installer avec la commande suivante :

```bash
$ pip3 install pytest
```

Testez maintenant l'execution de vos tests :

```bash
$ python3 -m "pytest"
```

Vous devriez obtenir l'output suivante :

```
================================================================================= test session starts =================================================================================
platform darwin -- Python 3.7.4, pytest-5.2.4, py-1.8.0, pluggy-0.13.0
rootdir: [...]/sample-app
collected 1 item

tests/test_sample1.py .                                                                                                                                                         [100%]

================================================================================== 1 passed in 0.18s ==================================================================================
```

### Ajout du job exécutant les tests unitaires

Dans notre `.gitlab-ci.yml`, ajoutons un job exécutant la commande `pytest` que nous venons d'utiliser :

```yaml
stages:
  - syntax
  - sast
  - unittests

[...]
Launch Unit Tests:
  stage: unittests
  image: $PYTHON3_IMAGE
  script:
    - pip install -r requirements.txt
    - python -m pytest --junit-xml=pytest-report.xml
```

> Note :
> Pour éviter un warning de la librairie pytest, nous allons ajouter également dans le dépôt un fichier `pytest.ini` avec le contenu suivant:
> ```
> [pytest]
> junit_family=legacy
> ```

Nous ajoutons un `stage` qui viendra s'executer après les tests du stage `sast` dans lequel nous n'avons qu'un seul job exécutant `pytest`.

Ajoutez vos modifications dans un commit et poussez les sur votre branche :

```bash
$ git add .gitlab-ci.yml pytest.ini tests/
$ git commit -m "Ajout d'un stage pour les tests unitaires"
$ git push origin add_pipeline_conformity
```

Surveillez l'évolution du pipeline.

> Question 1.4
>
> - Pourquoi le pipeline a t-il échoué ? Qu'avons-nous oublié de faire ?
>

Corrigez le problème et soumettez votre changement. Le pipeline ne doit plus échouer.

> **Tips**:
> - La version de `pytest` est récupérable dans votre environnement en exécutant la commande `pip show pytest`.
> - La commande `pip freeze` permet de lister l'ensemble des dépendances installées dans votre environnement Python.


## Merge de votre changement

Ce TP est maintenant terminé. Dans un contexte réel il serait temps de demander une revue de votre code à l'un de vos pairs.
Dans le cadre de la formation, vous pouvez "merger" vous-même vos changements (vérifiez-bien que la branche cible est celle de votre dépôt forké et non celle du dépôt d'origine faute d'avoir les droits de valider la merge request).

Retournez dans l'interface Gitlab sur votre Merge Request et cliquez sur le bouton "Merge".
