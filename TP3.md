# TP #3 - Déclenchement conditionnel de job

> **Objectifs du TP**
> * Exécuter un job sous condition
> * Etiqueter (tagguer) une image Docker


## Exécution conditionnelle du job de construction de l'image Docker

Dans ce TP, nous allons gérer dans notre pipeline l'exécution conditionnelle de job et introduire la gestion des *release* (version).

Vous devrez :
- lancer les tests unitaires seulement sur les `branches` et les `merge requests` ;
- construire les images Docker uniquement sur des commits (ou des merges sur master)
- tester cette image
- poser des labels sur les images Docker en fonction des tags posés sur le code.

### Ajout de conditions sur les différentes étapes de tests

Ajouter les lignes suivantes aux étapes de `syntax`, `sast` et `unittests` :

```yaml
rules:
  - if: '$CI_COMMIT_BRANCH || $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"'
    when: always
```


### Ajout de conditions sur l'étape de build Image

Ajouter les lignes suivantes aux étapes de `Build Image` ainsi que `Integ Tests` :

```yaml
rules:
  - if: '$CI_COMMIT_REF_NAME == "master"'
    when: always
```

### Ajout d'une nouvelle étape `release` qui ne sera exécutée que si le commit associé correspond à un **tag** git

Ouvrez le fichier `.gitlab-ci.yaml` et ajouter une étape de "Tag Image Docker for Release" :

Ajoutez une étape de release :

```yaml
stages:
[...]
  - release
```

Et ajoutez le code suivant en fin de fichier :

```yaml
[...]

Tag Image Docker for Release:
  stage: release
  image:
    name: solsson/crane@sha256:58647d756b9008f312827227d0344ca2a7439c99222668c95e93d99dcc94d9ac
    entrypoint: [""]
  script:
    - test -z "$DOCKER_CONFIG" && export DOCKER_CONFIG=/
    - |
      cat > "${DOCKER_CONFIG}/config.json" <<EOF
      { "auths": { "${CI_REGISTRY}": { "username": "gitlab-ci-token", "password": "${CI_JOB_TOKEN}" } } }
      EOF
    - crane cp "${CONTAINER_COMMIT_IMAGE}" "$CI_REGISTRY_IMAGE:${CI_COMMIT_TAG}"
  rules:
    - if: '$CI_COMMIT_TAG'
      when: always
```

> Question 3.1
>
> De nouvelles variables "magiques" ont été introduites.
>
> À quoi correspondent-elles ?

Expliquez ce que font les "rules" dans ce code.

Envoyez votre code en le poussant sur une branche :

```bash
$ git add .gitlab-ci.yml
$ git commit -m "ajout de conditions et gestion des releases"
$ git checkout -b add_docker_release_condition
$ git push origin add_docker_release_condition
```

> Question 3.2
>
> Vérifiez l'état du pipeline qui est lancé suite à votre push.
>
> Que constatez-vous ?

### Merge de votre branche dans master

Effectuez une merge request depuis l'interface de gitlab-ci (ou en ligne de commande).

> Question 3.3
>
> Quel est le comportement attendu au niveau du pipeline ?

Maintenant, effectuez le merge de votre branche dans `master`.

> Question 3.4
>
> Quel est le comportement attendu au niveau du pipeline ?

### Release de l'image Docker

Vérifions maintenant que l'étape "Release Image" est bien effectuée lorsque le code est étiqueté ("taggué").

Revenez sur `master` et mettez le code à jour :

```bash
$ git checkout master
$ git pull origin master
```
> Question 3.5
>
> Pourquoi doit-on lancer la commande `git pull origin master` ?

Tagguez maintenant le code :

```bash
$ git tag -a v1.0 -m "version 1.0 du code"
```
Pushez votre tag. Attention, la commande `git push` seule ne permet pas de pousser un tag sur le serveur distant.

Il faut spécifier le tag à pousser :

```bash
$ git push origin v1.0
```

> Question 3.6
>
> Quel est le comportement attendu au niveau du pipeline ?

Une fois celui-ci terminé, allez dans la registry de votre projet `Packages & Registries` puis `Container Registry` et trouvez l'image que vous venez de "labelliser". Tadaaa !

# Bonus

> **Objectif du TP bonus**
> * Factoriser le code de sa pipeline
>

## Factorisation du code de sa pipeline

Dans ce TP, nous allons factoriser le code de notre pipeline grâce à l'utilisation d'`extends`.
Précédemment, nous avons ajouté les lignes suivantes aux étapes de `syntax`, `sast` et `unittests` :

```yaml
rules:
  - if: '$CI_COMMIT_BRANCH || $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"'
    when: always
```

### Création d'un template

Créer un template en ajoutant le code suivant juste avant le premier job :

```yaml
.onlyBranchesOrMergeRequests:
  rules:
    - if: '$CI_COMMIT_BRANCH || $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"'
      when: always

[...]
```

Puis remplacer, `rules` précédemment ajouté aux étapes `syntax`, `sast` et `unittests` par le code suivant :

```yaml
[...]

PyCodeStyle:
  [...]
  extends: .onlyBranchesOrMergeRequests

[...]
```

Envoyez votre code en le poussant sur une branche :

```bash
$ git add .gitlab-ci.yml
$ git commit -m "factorisation du code de la pipeline"
$ git checkout -b refactor_pipeline_code
$ git push origin refactor_pipeline_code
```

> Question 3.7
>
> Vérifiez l'état du pipeline qui est lancé suite à votre push.
>
> Que constatez-vous ?

### Merge de votre branche dans master

Effectuez une merge request depuis l'interface de gitlab-ci (ou en ligne de commande) et c'est fini. 
