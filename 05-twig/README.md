# Twig

## Sommaire

<!--toc:start-->
- [Twig](#twig)
  - [Sommaire](#sommaire)
  - [Définition](#définition)
  - [Installation](#installation)
  - [Utilisation](#utilisation)
    - [Template](#template)
    - [Layout](#layout)
<!--toc:end-->

## Définition

- librairie logicielle permettant de créer des *templates* réutilisables et customisables
- ces templates serviront de vue HTML pour l'affichage des pages web

## Installation

```bash
docker compose exec app composer require twig
```

## Utilisation

- tous les templates seront stockés dans le dossier `templates`
- ils seront sous la forme de fichier `nom_du_template.html.twig`

### Template

- créer un nouveau fichier `templates/hello.html.twig`
- y ajouter le code suivant :

```twig
<h1>Hello {{ name }}</h1>
```

- dans le controller `src/Controller/HelloController.php`, modifier le contenu de la route :

```php
<?php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Attribute\Route;

class HelloController extends AbstractController
{
    #[Route('/hello-world', name: 'hello_world', methods:['GET'])]
    public function helloWorld(): Response
    {
        return $this->render('hello.html.twig', [
            'name' => 'Mr Mxyztplk',
        ]);
    }
}
```

- on utilise la fonction `render()` pour récupérer une vue et la remplir avec des données
  - le premier argument est le chemin de la vue depuis le dossier `templates`
  - le second argument est un tableau contenant les données à injecter dans la vue

- remarquer dans la vue la notation `{{}}`
  - elle est utilisée pour ajouter de la logique dans la vue et la rendre plus dynamique et intelligente
  - `{{ ... }}` permet d'afficher le contenu d'une variable
  - `{# ... #}` permet d'écrire un commentaire
  - `{% %}` permet d'exécuter des instructions logiques

```twig

{# condition : #}
{% if number % 2 == 0 %}
  <p>Nombre pair</p>
{% else %}
  <p>Nombre impair</p>
{% endif %}


{# boucles #}
{% for element in structure %}
  <li>{{ element }}</li>
{% endfor %}

```

- il est également possible d'applique des filtres et des modificateurs sur les variables qu'on manipule grâce à l'opérateur `|`

```twig
{{ 'WELCOME'|lower }}
{{ structure|length }}
{{ '  I like Twig.  '|trim }}
```

### Layout

- il est possible de hiérarchiser les templates pour pouvoir réutiliser la même structure sur plusieurs pages
- dans le fichier `templates/base.html.twig` remarquez :

```twig
{% block body %}{% endblock %}
```

- cela signifie qu'un template qui hérite de `templates/base.html.twig` pourra rajouter du contenu dans ce block
- modifier le fichier `templates/hello.html.twig` pour essayer :

```twig
{% extends 'base.html.twig' %}

{% block body %}
    <h1>Hello {{ name }}</h1>
{% endblock %}
```

Modifier le fichier `templates/base.html.twig` pour constater l'héritage :

```twig
...
    <body>
        Content from layout
        {% block body %}{% endblock %}
    </body>
...
```

## Sources

- https://symfony.com/doc/current/templates.html
- https://twig.symfony.com/doc/3.x/
- https://symfonycasts.com/screencast/symfony/twig
- https://symfonycasts.com/screencast/symfony/twig-inheritance
