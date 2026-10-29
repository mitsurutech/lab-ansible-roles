# Lab Ansible rôles pour la classe : un rôle, des handlers et Vault sur une VM

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md) | [Deutsch](README.de.md)

[![check](https://github.com/mitsurutech/lab-ansible-roles/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-ansible-roles/actions/workflows/check.yml)

Un **lab Ansible sur les rôles** prêt à l’emploi pour enseigner l’infrastructure as code, issu de la [plateforme d’apprentissage MitsuruTech](https://mitsurutech.co.uk/fr/learning), où les établissements scolaires donnent à chaque élève ses propres machines et son terminal dans le navigateur. C’est le deuxième niveau après [lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics). Chaque élève reçoit une **VM** Ubuntu avec **ansible-core** installé et un petit projet, et construit un **rôle** à partir de rien : ses variables, un modèle Jinja2, des tâches avec des étiquettes (tags), un **handler** qui ne s’exécute qu’une fois et un jeton gardé dans **Ansible Vault**. La machine elle-même est la cible : il ne faut rien d’autre et rien n’est installé sur l’ordinateur de l’élève.

## Ce que fait l’élève

Le lab laisse un projet dans `~/roles-lab` et un `README.md` dans le dossier personnel qui explique quoi faire en sept étapes sans donner aucune réponse :

```
ansible.cfg          pointe vers inventory.ini et le dossier des rôles
inventory.ini        deux hôtes, web1 et spare1, tous deux cette machine
site.yml             un play sur web qui utilise le rôle appconf
vault-password.txt   le mot de passe du coffre pour ce lab
roles/               l’endroit où va le rôle
```

1. Lancer le playbook et le voir s’arrêter, car le rôle `appconf` n’existe pas encore.
2. Créer le rôle avec `ansible-galaxy role init` et remplir son `meta/main.yml`.
3. Définir des variables dans `defaults/main.yml` et `vars/main.yml`. `site.yml` fixe déjà `app_env` et `app_dir` dans le play : l’élève doit découvrir laquelle l’emporte.
4. Écrire les tâches, étiquetées `config` ou `user` : un dossier, un fichier créé à partir d’un modèle avec une boucle et une condition, un fichier par fonctionnalité, un fichier de jeton, un utilisateur système et deux petits fichiers, dont un seulement si l’utilisateur a vraiment été créé.
5. Faire en sorte qu’un handler notifié par trois tâches ne s’exécute qu’une fois, et que le rôle lance ses handlers en cours de route.
6. Garder `app_api_token` dans un fichier créé avec `ansible-vault`, ouvert avec le fichier de mot de passe, et écrit en clair nulle part ailleurs.
7. L’exécuter deux fois (la seconde ne doit rien changer), puis essayer `--check --diff`, `--tags` et `--limit`.

## Comment il est noté

Le lab comporte une **vérification**, un script qui note une VM rendue sans que l’enseignant ait à la lire. Elle s’exécute sur une copie de la machine enregistrée, après un démarrage à neuf. Elle lit d’abord le rôle : les dossiers standard, un `meta/main.yml` rempli, au moins six tâches dans le rôle et non dans le playbook, un handler, une boucle, des étiquettes, et un modèle avec une boucle `for` et un `if`. Elle refuse `shell`, `command`, `raw` et `script` (et quelques autres façons de contourner les modules d’Ansible), ainsi qu’un lookup `pipe` qui lance un programme. Elle ouvre le fichier du coffre avec le mot de passe du lab et refuse un jeton écrit en clair dans le projet.

Elle **exécute ensuite le `site.yml` de l’élève sept fois avec des valeurs que personne n’a jamais vues** : sur une machine propre, de nouveau sans rien à changer, en simulation (dry run) qui doit montrer le changement sans rien écrire, avec d’autres valeurs, avec chaque étiquette, et avec `--limit` sur l’hôte que le play n’utilise pas. Elle compte les exécutions du handler dans la sortie réelle, si bien qu’un handler lancé trois fois est refusé, et lit les fichiers produits. La première ligne affichée dit ce qui ne va pas.

Ce qu’elle ne fait pas : elle ne redémarre jamais un vrai service, car une vérification ne peut pas le prouver, et un élève qui a les droits root sur sa propre VM pourrait encore trafiquer Ansible lui-même. C’est une vérification pour la classe, pas une preuve.

## Ce que contient le fichier

`ansible-roles.lab.yaml` est une **définition de lab VM** complète, exactement ce que la plateforme exécute :

| Clé | Ce que c’est |
| --- | --- |
| `cpus`, `memory`, `disk` | la taille de la VM de chaque élève : 1 CPU, 1024 Mo et 10 Go |
| `open` | les ports qui s’ouvrent dans le navigateur : aucun, le lab est le terminal |
| `setup` | un script exécuté une fois, en tant que root, au premier démarrage de la VM de chaque élève : il installe `ansible-core` depuis l’archive d’Ubuntu et écrit le projet, le playbook, un faux mot de passe de coffre et le README |
| `check` | le script de notation, exécuté en tant que root sur une copie du disque enregistré après un démarrage à neuf ; son code de sortie est le résultat et `0` signifie réussi |

Le mot de passe du coffre dans le projet est un **élément de test** : il dit lui-même qu’il n’est pas secret, et le jeton que l’élève chiffre est une donnée de démonstration, jamais un vrai secret.

## L’utiliser sur la plateforme

**Pour le donner comme travail à une classe :**

1. Dans votre module, ajoutez une tâche et choisissez la carte **Lab VM Ubuntu**.
2. Appuyez sur l’exemple Ansible rôles, ou collez le contenu de `ansible-roles.lab.yaml`. Utiliser un exemple remplace le texte, et Annuler rétablit le vôtre.
3. Publiez et essayez d’abord votre propre copie. Chaque élève a sa propre VM, conservée d’une session à l’autre.
4. Distribuez-le comme n’importe quelle tâche. Une fois rendu, lancez la vérification sur la ligne d’un élève dans la liste de la classe : elle démarre une copie de sa machine enregistrée, qui n’est pas sauvegardée en retour, et indique si elle a réussi et ce qu’elle a affiché.

Une leçon de la série Basics de la plateforme se travaille dans ce lab. La [page de ce lab](https://mitsurutech.co.uk/fr/help/lab-ansible-roles) dans le manuel, son [guide des labs VM Ubuntu](https://mitsurutech.co.uk/fr/help/labs) et sa [page des exemples de labs](https://mitsurutech.co.uk/fr/help/sample-labs) disent le reste.

Pas encore de compte pour votre établissement ? [Votre première année est gratuite](https://mitsurutech.co.uk/fr/pricing) jusqu’à 50 élèves, et [l’inscription](https://mitsurutech.co.uk/fr/signup) ne demande qu’une adresse e-mail.

## Vérifier le fichier

À chaque push, `.github/workflows/check.yml` vérifie que le fichier est un YAML valide avec les clés ci-dessus et que `setup` et `check` tiennent chacun dans la limite de 16 Ko de la plateforme. Il ne démarre pas de VM : cela se fait sur les machines de la plateforme.

## Ce qui ne fonctionne pas dans un lab

La VM atteint l’archive de paquets d’Ubuntu, PyPI, npm et GitHub, et rien d’autre. Le lab n’utilise que l’archive d’Ubuntu : chaque travail se fait donc avec les modules d’Ansible sur des fichiers, des dossiers et un utilisateur système. Il n’y a aucun service à redémarrer : le handler se montre par ce qu’une vérification peut voir, le nombre de fois où il a tourné et un fichier qu’il écrit. La cible est la VM elle-même, en connexion locale.

## À propos

Réalisé par [Mitsuru Technologies](https://mitsurutech.co.uk/fr) pour la [plateforme d’apprentissage MitsuruTech](https://mitsurutech.co.uk/fr/learning) : Python, Java, Go, Node, SQL et Linux dans le navigateur pour les écoles et les établissements, avec des labs de conteneurs, des labs VM, des applications web déployées depuis GitHub et des quiz qui se corrigent tout seuls. Essayez du code sans compte dans le [bac à sable](https://mitsurutech.co.uk/fr/playground).

D’autres labs du même éditeur : [lab-netlab](https://github.com/mitsurutech/lab-netlab), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea), [lab-sample](https://github.com/mitsurutech/lab-sample), [lab-sdn-lab](https://github.com/mitsurutech/lab-sdn-lab), [lab-reverse-lab](https://github.com/mitsurutech/lab-reverse-lab), [lab-formal-lab](https://github.com/mitsurutech/lab-formal-lab), [lab-hdl-lab](https://github.com/mitsurutech/lab-hdl-lab), [lab-k3s-lab](https://github.com/mitsurutech/lab-k3s-lab), [lab-tofu-lab](https://github.com/mitsurutech/lab-tofu-lab) et [lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics).

## Licence

Cette œuvre est sous licence [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Vous pouvez la partager et l’adapter pour apprendre et enseigner, à condition de citer Mitsuru Technologies Ltd, de ne pas l’utiliser commercialement et de partager tout ce que vous en tirez sous la même licence. Le texte complet est dans le fichier `LICENSE`.

[Ansible](https://www.ansible.com) (`ansible-core`, GPL-3.0) est installé depuis l’archive d’Ubuntu au démarrage d’une VM ; rien n’en est distribué ici.
