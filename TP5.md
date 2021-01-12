# TP #6 - Labels et Coverage

> **Objectifs du TP**
> * Afficher les labels build et coverage

## Répertoire de travail

Assurez-vous de bien être dans le répertoire de travail `~/sample-app` que nous avons utilisé dans les TPs de cours.

## Création de labels

Nous souhaitons afficher un label de build et un label de couverture des tests unitaires.

Nous allons commencer par ajouter les dépendances suivantes au fichier `requirements.txt` :

```
coverage==4.5.4
importlib-metadata==0.23
pytest-cov==2.8.1
typed-ast==1.4.1
zipp==0.6.0
```

Ajouter ensuite le bloc `before_script` au jobs ci-dessous et `--cov=./` à la commande `python -m pytest`, comme ceci :

```yaml

PyCodeStyle:
  stage: syntax
  image: $PYTHON3_IMAGE
  before_script:
    - apk add --no-cache gcc musl-dev
  [...]

Pylint:
  stage: syntax
  image: $PYTHON3_IMAGE
  before_script:
    - apk add --no-cache gcc musl-dev
  [...]

Static Code Analysis Tests:
  stage: sast
  image: $PYTHON3_IMAGE
  before_script:
    - apk add --no-cache gcc musl-dev

Launch Unit Tests:
  stage: unittests
  image: $PYTHON3_IMAGE
  before_script:
    - apk add --no-cache gcc musl-dev
  script:
    - pip install -r requirements.txt
    - python -m pytest --junit-xml=pytest-report.xml --cov=./
```

Afin d'ajouter les dépendances, modifier votre `Dockerfile` :

```
FROM python:alpine

ARG APP_VERSION
ENV APP_VERSION=$APP_VERSION

# Add dev tools to build python deps
RUN apk add --no-cache --virtual .build-deps gcc musl-dev           # Ligne ajoutée
# Create and change working directory
WORKDIR /app
# Add application requirements
COPY requirements.txt .
# Install requirements
RUN pip install -r requirements.txt
# Remove dev tools
RUN apk del .build-deps                                             # Ligne ajoutée
# Add application
COPY app.py .
# Create a specific user to run the Python application
RUN adduser -D my-user -u 1000

USER 1000

EXPOSE 8000

# Launch application
ENTRYPOINT ["gunicorn"]
CMD ["-b", "0.0.0.0:8000", "app:APP"]
```

Depuis l'IHM de Gitlab, dans Settings > CI/CD > General pipelines > Test coverage parsing, saisir la regex correspondant à "pytest-cov (Python)" :

```
^TOTAL.+?(\d+\%)$
```

Puis cliquer sur le bouton "Save Changes".

Nous allons ensuite créer un fichier `README.md` à la racine, il faudra y ajouter les lignes suivantes pour ajouter les labels. Il faudra remplacer le namespace par le votre :

```
# sample-app

[![build](https://gitlab.com/<VOTRE_LOGIN_GITLAB>/sample-app/badges/master/pipeline.svg)](https://gitlab.com/<VOTRE_LOGIN_GITLAB>/ci-app/commits/master)
[![coverage](https://gitlab.com/<VOTRE_LOGIN_GITLAB>/sample-app/badges/master/coverage.svg)](https://gitlab.com/<VOTRE_LOGIN_GITLAB>/ci-app/commits/master)
```

Soumettez vos changements dans une nouvelle Merge Request :

```
$ git checkout -b add_labels
$ git add README.md .gitlab-ci.yml Dockerfile requirements.txt
$ git commit -m "Ajout de labels"
$ git push origin add_labels
```

Les tests doivent passer sans erreurs.

> Question
>
> - Pourquoi le label coverage affiche t-il `unknown` ? 
>

## Merge de votre changement

Retournez dans l'interface Gitlab sur votre Merge Request et cliquer sur le bouton "Merge".

Une fois le code mergé et l'exécution du pipeline de master terminé, le label coverage doit afficher la couverture des TU sur le README de la branche master.
