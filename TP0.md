
# TP0 PREPARATION - Création compte gitlab.com et configuration Git en local

> **Objectifs du TP**
> * Créer, si ce n'est pas déjà fait, un compte sur gitlab.com
> * Configurer les paramètres globaux de Git en local
> 

# Création d'un compte sur gitlab.com

Si vous ne possédez pas encore de compte sur gitlab, procédure pour en créer un :

Aller sur le site [https://gitlab.com/](https://gitlab.com/) et cliquer sur `Register`

Remplir le formulaire de création de compte, après avoir soumis le formulaire, consultez votre boite email et **cliquez sur le lien d'activation reçu de la part de gitlab pour activer votre compte**. Vous devez avoir un compte activé pour pouvoir utiliser les runners partagés.

> Votre nom d'utilisateur gitlab (username) est important car vous l'utiliserez tout au long des TP pour vous connecter sur gitlab, pensez donc à le noter.
 

# Configuration de Git sur votre poste local

## Configuration de l'environnement local pour annoter les commits

Première chose à faire sera d'instaler git, je vous renvoie sur ce lien pour trouver le bon moyen d'installation selon votre environnement de travail: 
https://git-scm.com/book/fr/v2/Démarrage-rapide-Installation-de-Git

Il est cependant nécessaire de personnaliser votre environnement Git pour indiquer à Git comment annoter vos commits par exemple. 
Vous ne devriez avoir à réaliser ces réglages qu’une seule fois ; ils persisteront lors des mises à jour. Vous pouvez aussi les changer à tout instant en relançant les mêmes commandes.

> Git propose la commande `git config` pour vous permettre de voir et modifier les variables de configuration qui contrôlent le comportement de Git. Ces variables peuvent être stockées dans trois endroits différents :
> - Fichier `/etc/gitconfig` : Contient les valeurs pour tous les utilisateurs et tous les dépôts du système. Si vous passez l’option `--system` à git config.
> - Fichier `~/.gitconfig` : Spécifique à votre utilisateur. Vous pouvez forcer Git à lire et écrire ce fichier en passant l’option `--global`.
> - Fichier `config` dans le répertoire Git (c’est-à-dire `.git/config`) du dépôt en cours d’utilisation : spécifique au seul dépôt en cours.
> 
> Sur les systèmes Windows, Git cherche le fichier `.gitconfig` dans le répertoire $HOME (%USERPROFILE% dans l’environnement natif de Windows)

```bash
$ git config --global user.name "<Prenom Nom>" 
$ git config --global user.email <votre email>
$ git config --global color.ui true
```

Exemple de résutat de fichier `~/.gitconfig`

```ini
[core]
	editor = /usr/bin/vim
[user]
	email = contact@samuelantunes.fr
	name = Samuel Antunes
[ui]
	color = true
```


# Installation de votre clé SSH dans votre compte Gitlab

Si vous essayez de cloner ou pousser du code sur Gitlab.com (ne le faites pas) vous obtiendrez probablement l'erreur suivante :
```bash
git push -u origin --all
Warning: Permanently added 'gitlab.com,35.231.145.151' (ECDSA) to the list of known hosts.
git@gitlab.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

Cette erreur est lié au fait qu'on utilise le protocole `git` pour envoyer les données du dépôt sur gitlab. Celui-ci se base à son tour sur le protocole SSH. Pour s'authentifier sur gitlab, il est nécessaire d'avoir configuré au préalable sur gitlab.com sa clé SSH.

Vous devrez donc générer votre clé SSH si ce n'est pas déjà fait afin de l'importer sur gitlab.com, pour cela je vous renvoie sur ce tuto : 
https://docs.gitlab.com/ee/ssh/ (sur Windows vous pouvez utiliser PuTTY)

Depuis l'interface gitlab.com, cliquez alors sur le bouton de votre profil en haut à droite, puis cliquez sur `Settings`.

- la clé privée, que vous ne devez jamais sortir de votre machine se trouve dans `~/.ssh/id_ed25519`
- la clé publique, que vous ne pouvez utiliser sans soucis se trouve dans `~/.ssh/id_ed25519.pub`

Peu importe si le format de votre clé difére un peu, que ce soit "id_rsa" ou autre.

Vous pouvez afficher votre clé publique avec cette commande : 

```bash
$ cat ~/.ssh/id_ed25519.pub
ssh-ed25519 AAAAC3... samuelantunes@mac
```

Puis allez dans la page `SSH Keys` :

Copier-coller maintenant **la partie publique** de **votre clé SSH** dans cette interface puis valider. 
Il est maintenant temps de retester d'envoyer votre dépôt sur Gitlab.

Les prérequis sont réalisés, bravo !!
