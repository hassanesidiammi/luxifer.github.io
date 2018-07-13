---
layout: post
title: "Déployer un projet Symfony 2 avec Capifony"
tags:
- capifony
- capistrano
- déploiement
- Dev
- git
- php
- scm
- symfony
- web
status: publish
type: post
published: true
meta:
  thumb: 'Capture-du-2012-04-25-161150.png'
  dsq_thread_id: '663485541'
dark: true
---
Bonjour à tous!

Aujourd'hui j'ai voulu tester [capifony](http://capifony.org/). C'est un petit programme basé sur [capistrano](https://github.com/capistrano/capistrano) qui permet de déployer facilement un projet symfony 1.4 ou 2 sur un serveur de production. Ce programme permet de gérer les releases de votre projet, de faire des rollback, de revenir sur une version anciennement déployée, etc. Il est écrit en ruby par un développeur de l'agence web [KnpLabs](http://knplabs.fr/).

Tout d'abord il faut installer la gem capifony :

```bash
gem install capifony
```

Ensuite il faut se rendre dans le répertoire de son projet pour initialiser capifony.

```bash
export PATH=$PATH:~/.gem/ruby/version/bin
cd /path/of/project
capifony .
```

Cette commande va créer un fichier `Capfile` à la racine de votre projet et un fichier `deploy.rb` dans le dossier `app/config/`, ou dans le dossier `config/` pour un projet symfony 1.4.

Ensuite il faut configurer le fichier deploy.rb afin de spécifier tous les paramètres nécessaires au déploiement.

```ruby
set :domain, "" # adresse du serveur de production
set :deploy_to, "" # répertoire ou déployer
set :app_path, "app"

set :repository, "" # adresse du scm
set :scm, :git
# Or: `accurev`, `bzr`, `cvs`, `darcs`, `subversion`, `mercurial`, `perforce`, `subversion` or `none`
set :deploy_via, :copy

set :model_manager, "doctrine"
# Or: `propel`

role :web, domain # Your HTTP server, Apache/etc
role :app, domain # This may be the same as your `Web` server
role :db, domain, :primary => true # This is where Rails migrations will run

set :use_sudo, false
set :use_composer, false
set :keep_releases, 3
```

Il est possible de définit une option en plus si l'utilisateur sur le serveur de production est différent de celui de la machine locale :

```ruby
set :user, "" # utilisateur distant
```

Pour un projet Symfony 2 il peut être utile d'ajouter ces options la, pour partager le dossier `vendors/` et `logs/`  entre toutes les releases et le fichier `parameters.ini`.

```ruby
set :shared_files,      ["app/config/parameters.ini"]
set :shared_children,     [app_path + "/logs", web_path + "/uploads", "vendor"]
set :update_vendors, true
```

Si votre projet utilise Composer comme gestionnaire de dépendances il faut aussi rajouter cette option :

```ruby
set :use_composer, true
```

Une fois ce fichier modifié, il faut initialiser la structure des dossiers sur le serveur de production en tapant cette commande :

```bash
cap deploy:setup
```

Il faut maintenant configurer sur le serveur de production le fichier `parameters.ini` :

```bash
ssh production_server
cd /path/to/project/shared
mkdir -p app/config
vim app/config/parameters.ini
```

Et une fois toutes ces étapes finies avec succès, il suffit juste de lancer la commande :

```bash
cap deploy
```

Cette commande va créer un nouveau dossier dans `/path/to/project/releases` et faire pointer le dossier `current/` vers cette dernière release. Ensuite installer les vendors si c'est un projet Symfony 2, publier les assets, vider le cache et lancer tous les hooks lié au scm.

Une fois les projet déployé, il suffit juste de faire pointer son domaine vers le dossier `current/` du projet.
Pour les autres commandes disponibles je vous laisse vous référer à la page de [capifony](http://capifony.org/).

Bon déploiement !
