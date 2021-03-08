# TP #2 - Pipeline avec tests d'intégration

> **Objectifs du TP**
> * Effectuer des tests qui nécessitent le démarrage de l'application
> * Démarrer une ressource requise par notre application au démarrage (une base de données) grâce à Gitlab et à Docker
>

## Répertoire de travail

### Si vous êtes sur votre poste de travail

Assurez-vous que vous êtes bien dans le répertoire racine du projet sample-app que vous avez cloné dans le TP1.

## Le plan d'attaque

Nous avons ajouté des tests qui ne nécessitent pas de démarrer l'application. Il est maintenant temps
de tester l'application pendant son runtime. Nous allons utiliser Docker pour packager notre application
et déclencher son exécution.

Notre première étape consistera donc à créer une image docker contenant notre application.

## Ajout d'une étape de build d'un container

La création d'une image passe par l'écriture d'un fichier `Dockerfile`. Le sujet du TP n'étant pas de créer
une image, le fichier `Dockerfile` est déjà présent dans le dépôt à la racine du projet.

Ajoutons un job dans le pipeline permettant la création d'une image docker.
Nous n'utiliserons pas Docker pour créer l'image, nous utiliserons Kaniko qui permet de créer une image sans accès au daemon docker (donc sans ouvrir la socket).

Ajoutez le code suivant à votre pipeline :

```yaml
variables:
  CONTAINER_COMMIT_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
  CONTAINER_LATEST_IMAGE: $CI_REGISTRY_IMAGE:latest
  PYTHON3_IMAGE: python:alpine

stages:
  - syntax
  - unittests
  - sast
  - build

[...]

Build Image:
  stage: build
  image:
    name: gcr.io/kaniko-project/executor:debug
    entrypoint: [""]
  script:
    - test -z "$DOCKER_CONFIG" && export DOCKER_CONFIG=/kaniko/.docker
    - |
      cat > "${DOCKER_CONFIG}/config.json" <<EOF
      { "auths": { "${CI_REGISTRY}": { "username": "gitlab-ci-token", "password": "${CI_JOB_TOKEN}" } } }
      EOF
    - >-
        /kaniko/executor
        --context "${CI_PROJECT_DIR}"
        --dockerfile "${CI_PROJECT_DIR}/Dockerfile"
        --build-arg APP_VERSION="${CI_COMMIT_SHORT_SHA}"
        --destination "${CONTAINER_COMMIT_IMAGE}"
        --destination "${CONTAINER_LATEST_IMAGE}"
```

Envoyez votre code en revue en poussant votre code sur une branche :

```bash
$ git add .gitlab-ci.yml
$ git commit -m "Ajout d'un stage pour la création d'une image docker"
$ git checkout -b add_docker_build
$ git push origin add_docker_build
```
Créez la Merge Request et attendez l'exécution du pipeline.

> Question 2.1
>
> - Sauriez-vous expliquer la manipulation que nous effectuons avec le fichier `config.json` ?
> - Pourquoi effectuons-nous cette opération ?
> - Comment pouvons-nous vérifier si l'image est bien disponible ?

Vous pouvez maintenant tester le démarrage de l'application en démarrant un container utilisant votre image :

```
$ docker container run --rm -it -d -p 8000:8000 registry.gitlab.com/<votre_login_gitlab>/sample-app:latest
```

Il est possible que la commande précédente n'ai pas fonctionné si vous avez des majuscules dans votre pseudo gitlab "must be lowercase". Si c'est le cas passez directement à la partie suivante. Sinon, vous pouvez maintenant tester l'application en envoyant une requête à l'application :

```
$ curl 127.0.0.1:8000
```

Celle-ci vous répondra `Hello, App!` si tout fonctionne correctement.

Une fois ce test concluant vous pouvez merger votre Merge Request (MR).
La suite du TP sera effectuée sur une autre MR.

## Création de tests d'integration pour notre application

Dans l'étape précédente nous avons créé un artefact (l'image de container) contenant notre application.
Nous souhaitons maintenant verifier que cet artefact est fonctionnel, nous allons donc le tester à travers
un nouveau job dans notre pipeline.

Nous avons utiliser le hash du commit comme tag pour notre image, il sera donc facile de retrouver notre image
à l'aide de la variable d'environnement `CI_COMMIT_SHORT_SHA` disponible dans les job Gitlab-CI.

Pour effectuer notre test, nous allons créer des tests effectuant des requêtes HTTP sur notre application
qui sera démarrée dans un container basé sur l'image créée dans le job précedent.

Commençons par créer le fichier `gitlab-ci-scripts/test_bats.sh` contenant les tests que nous souhaitons exécuter :

```bash
#!/usr/bin/env bats

@test "Check Home page" {
  curl -sfq my-app:8000/ | fgrep Hello
}

@test "Check version URL" {
  curl -sfq my-app:8000/version
}

@test "Check version URL Content" {
  curl -sfq my-app:8000/version | fgrep $CI_COMMIT_SHORT_SHA
}
```

Donnons les droits d'exécution au script :

```bash
$ chmod 740 gitlab-ci-scripts/test_bats.sh
```

Créons maintenant le job qui executera ces tests dans le pipeline :

```yaml
stages:
  - syntax
  - sast
  - unittests
  - build
  - inttests

[...]

Integ tests:
  image:
    name: dduportal/bats:0.4.0
    entrypoint: [""]
  stage: inttests
  script:
    - $CI_PROJECT_DIR/gitlab-ci-scripts/test_bats.sh
```

> Question 2.2
>
> - Pourquoi avons-nous changé la valeur du paramètre `image` ?

Nous devons maintenant demander à Gitlab de démarrer notre application avant d'exécuter notre test.
Nous allons utiliser le paramètre `services` qui permet de spécifier une liste d'images docker à démarrer avant d'exécuter
les commandes spécifiées dans le paramètre `script`.

```yaml
Integ tests:
  image:
    name: dduportal/bats:0.4.0
    entrypoint: [""]
  stage: inttests
  services:
    - name: $CONTAINER_COMMIT_IMAGE
      alias: my-app
  script:
    - $CI_PROJECT_DIR/gitlab-ci-scripts/test_bats.sh
```

En précisant un `alias` pour notre image, nous serons en capacité d'utiliser l'alias comme un nom d'hôte pour atteindre l'application.
Vous aurez peut-être remarqué que les tests que nous avons écrits dans `gitlab-ci-scripts/test_bats.sh` pointent sur une URL contenant `my-app:8000`
ce qui correspond déjà à la valeur de l'alias que nous utilisons ici.

Soumettez votre changement dans une nouvelle Merge Request :

```shell
$ git checkout -b add_integration_tests
$ git add .gitlab-ci.yml gitlab-ci-scripts/test_bats.sh
$ git commit -m "Ajout d'un job executant les tests d'integration"
$ git push origin add_integration_tests
```

Les tests doivent passer sans erreur.

Votre pipeline doit maintenant ressembler à celui-ci :
<img src="https://i.ibb.co/kXyhhWt/Screenshot-2021-01-11-at-23-36-53.png">

## Merge de votre changement

Ce TP est maintenant terminé. Dans un contexte réel il serait temps de demander une revue de votre code à l'un de vos pair.
Dans le cadre de la formation, vous pouvez "merger" vous même vos changements.

Retournez dans l'interface Gitlab sur votre Merge Request et cliquez sur le bouton "Merge".
