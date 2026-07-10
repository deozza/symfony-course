# Router et controller

## Sommaire

<!--toc:start-->
- [Router et controller](#router-et-controller)
  - [Sommaire](#sommaire)
  - [Définitions](#définitions)
    - [Modèle MVC](#modèle-mvc)
    - [Router](#router)
    - [Controller](#controller)
  - [Ajouter une route](#ajouter-une-route)
  - [Sources](#sources)
<!--toc:end-->

## Définitions

### Modèle MVC

- design pattern
  - une architecture logicielle, testée et éprouvée, permettant de répondre efficacement à un besoin particulier : construire une application web
- Modèle Vue Controleur (Modele View Controller)

### Router

- dans un MVC, toutes les requêtes sont redirigées vers un point d'entrée unique : le router
- il vérifie que l'endpoint demandé existe bien dans l'application
- si c'est le cas, il redirige la requête vers le controller correspondant

### Controller

- reçoit la requête et ses données depuis le router
- autorise l'exécution de la requête en fonction des permissions utilisateur
- appelle le code métier pour récupérer les données correspondantes à la requête
- retourne une réponse

### Modèle

- contient le code métier de l'application
- dans Symfony, correspond aux Services, Entities, Repositories, ...
  - un service est appelé par un controller (ou un autre service), puis peut appeler un repository, qui va appeler une entity
    - l'ordre d'appel est important pour respecter les principes [SOLID de responsabilité](https://fr.wikipedia.org/wiki/Principe_de_responsabilit%C3%A9_unique) et la [loi de Déméter](https://fr.wikipedia.org/wiki/Loi_de_D%C3%A9m%C3%A9ter)

### Vue

- la vue est le contenur qui embarque les données récupérées depuis le modèle et qui sera renvoyer au client
  - sous forme d'une page web, d'une réponse JSON, ...
- elle est construite par le controller

### Le HttpKernel

Toute cette architecture et son fonctionnement est décrite dans le kernel (moteur) http de symfony : https://symfony.com/doc/current/components/http_kernel.html#the-request-response-lifecycle

## Ajouter une route

- créer une nouvelle classe dans le dossier `src/Conroller`
- la classe devra étendre de `Symfony\Bundle\FrameworkBundle\Controller\AbstractController`

```php
<?php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class HelloController extends AbstractController
{
}
```

- lorsqu'on souhaite rajouter un endpoint à ce controller, il faut créer une fonction avec un attribut `#Route`
- la fonctione devra retourner une instance de `Symfony\Component\HttpFoundation\Response`

```php
<?php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Attribute\Route;

class HelloController extends AbstractController
{
    #[Route('/hello-world')]
    public function helloWorld(): Response
    {
      return new Response('hello world', 200);
    }
}
```

- l'attribut `#Route` permet de configurer l'endpoint en lui donnant
  - une url
  - un `name` (qui pourra servir au debug)
  - la liste de méthode HTTP valide pour accéder à l'endpoint
  - ...

Exemple :

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
      return new Response('hello world', 200);
    }
}
```
Si on souhaite vérifier que l'endpoint créé est bien pris en charge par Symfony, utiliser la commande :

```bash
docker compose exec app php bin/console debug:router
```
Pour visualiser le résultat, il faut tout d'abord lancer le serveur de développement.

Soit via la CLI Symfony :

```bash
docker compose exec app symfony serve listen-ip=0.0.0.0 # pour pouvoir laisser passer le serveur au travers du conteneur
```

Soit via le serveur de développement PHP :

```bash
docker compose exec app php -S 0.0.0.0:8000 -t ./public
```
Utiliser ensuite un navigateur web ou un client API pour se rendre sur `127.0.0.1:8000/hello-world`.

## Sources

- https://symfony.com/doc/current/routing.html
- https://symfonycasts.com/screencast/symfony/route-controller
