# Entity

## Sommaire

## ORM - définition

https://fr.wikipedia.org/wiki/Mapping_objet-relationnel

## Installation

```bash
docker compose exec app composer require symfony/orm-pack
```

- modifier le contenu du fichier `.env` pour relier l'application symfony au conteneur mysql :

```bash
DATABASE_URL="mysql://user:password@mysql:3306/db?serverVersion=8.0.37"
```
- lancer la commande suivante pour créer la base de données

```bash
docker composer exec app php bin/console doctrine:database:create
```

## Entity

- toutes les entités sont stockées dans le dossier `src/Entity`.

```php
<?php
namespace App\Entity;

use App\Repository\ProductRepository;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: ProductRepository::class)] # permet de lier la classe à l'ORM et donc la création de la table
class Product
{
    #[ORM\Id] # indique la Primary Key
    #[ORM\GeneratedValue] # indique que la valeur sera AUTO_INCREMENT
    #[ORM\Column] # indique que cette propriété devra être gérée par l'ORM et sera une colonne dans la table
    private ?int $id = null;

    #[ORM\Column(length: 255)]
    private ?string $name = null;

    #[ORM\Column]
    private ?int $price = null;

    public function getId(): ?int
    {
        return $this->id;
    }

    // ... getter and setter methods
}
```

- lancer la commande suivante pour mettre à jour la base avec la nouvelle table :

```bash
docker compose exec app php bin/console doctrine:schema:update
```

## Sauvegarder des données dans la base

- https://symfony.com/doc/current/doctrine.html#persisting-objects-to-the-database
- https://symfony.com/doc/current/doctrine.html#updating-an-object
- https://symfony.com/doc/current/doctrine.html#deleting-an-object

- à retenir :
    - lorsqu'on crée une nouvelle donnée, utiliser `persist` puis `flush`
        - pour chaque donnée qu'on crée, il faut utiliser 1 `persist`
        - si on doit créer plusieurs données à la chaine, il vaut mieux utiliser plusieurs `persist` puis 1 seul `flush` à la fin pour éviter des problèmes de synchronisation ou de cohérence des données
    - lorsqu'on met à jour une donnée existante, uniquement utiliser `flush`
    - lorsqu'on supprime une donnée existante, utiliser `remove` puis `flush`
        - pour chaque donnée qu'on supprime, il faut utiliser 1 `remove`
        - si on doit supprimer plusieurs données à la chaine, il vaut mieux utiliser plusieurs `remove` puis 1 seul `flush` à la fin pour éviter des problèmes de synchronisation ou de cohérence des données

## Sources

- https://symfonycasts.com/screencast/symfony-doctrine/entity
