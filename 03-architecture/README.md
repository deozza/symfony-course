# Architecture

- `./bin` : scripts exécutables dans la console
  - exemple: `bin/console` pour exécuter les commandes symfony
- `./config` : fichiers `.yml` de configuration du framework et de ses dépendances
- `./public` : fichiers accessibles directement depuis un client (dont le point d'entrée générale de l'application `index.php`)
- `./src` : code source de l'application
- `./var` : données en vrac (cache, logs, bdd sqlite, ...)
- `./vendor` : dépendances installées
- `composer.json` : fichier de configuration du projet et de ses dépendances
- `composer.lock` : fichier dynamique listant toutes les dépendances (et leurs dépendances) du projet
- `symfony.lock` : même chose que le `composer.lock` mais dédié à Symfony
