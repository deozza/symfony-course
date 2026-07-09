# Event based development

## Sommaire

<!--toc:start-->
- [Event based development](#event-based-development)
  - [Sommaire](#sommaire)
  - [Définition](#définition)
  - [Event](#event)
    - [Créer ses propres Events](#créer-ses-propres-events)
    - [Émettre un Event](#émettre-un-event)
  - [EventListener](#eventlistener)
  - [EventSubscriber](#eventsubscriber)
  - [Sources](#sources)
<!--toc:end-->

## Définition

Traditionnellement, pour exécuter une fonctionnalité en programmation orientée objet, on instancie un objet de la classe contenant la méthode et on appelle directement cette méthode. Qui elle-même peut appeler d'autres méthodes de la classe ou d'autres classes. Les avantages sont qu'on peut facilement retracer le chemin pris durant l'exécution de la fonctionnalité et qu'il suffit d'appeler directement une fonctionnalité par son nom pour l'exécuter. On gagne du temps pendant le développement et à la relecture du code. Cependant, dans une application plus complexe, on peut se retrouver avec énormément de couplage entre les différentes classes. Ce couplage risque d'introduire :

- du code dupliqué
  - si les mêmes appels de fonctions sont requises dans plusieurs fonctionnalités
- une désynchronisation de version du code
  - si on a retiré un appel d'un groupe de fonctions d'une fonctionnalité mais pas dans les autres
  - et inversement si on ajoute un appel de fonction dans une fonctionnalité mais pas dans les autres

Pour réduire ce couplage et rendre l'application plus flexible, on peut se tourner vers une architecture `event-based`. Les points d'entrée des fonctionnalités émettent des évènements, uniquement les parties responsables de ces évènements les prennent en charge et résolvent les besoins.

## Event

Tous les events doivent étendre la classe `Symfony\Component\EventDispatcher\Event` .

### Créer ses propres Events

Par convention, tous les `Event` sont placés dans le dossier `src/Event`. Un Event doit étendre la classe `Syfmony\Contracts\EventDispatcher\Event`. Un Event ne contient pas de fonctionnalité mais uniquement des données. Exemple :

```php
	<?php
	
	namespace App\Event;
	
	use Symfony\Contracts\EventDispatcher\Event;
	
	class UserRegisteredEvent extends Event
	{
	    const NAME = 'user.registered';
	
	    private $user;
	
	    public function __construct(User $user)
	    {
	        $this->user = $user;
	    }
	
	    public function getUser()
	    {
	        return $this->user;
	    }
	}
?>
```

### Émettre un Event

Il faut avoir accès au service `EventDispatcher` pour émettre un Event. Exemple depuis un controller :

```php
	<?php
	
	namespace App\Controller;
	
	use App\Event\UserRegisteredEvent;
	use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
	use Symfony\Component\EventDispatcher\EventDispatcherInterface;
	use Symfony\Component\HttpFoundation\Response;
	use Symfony\Component\Routing\Annotation\Route;
	
	class UserController extends AbstractController
	{
	    /**
	     * @Route("/register", name="user_register")
	     */
	    public function register(EventDispatcherInterface $eventDispatcher): Response
	    {
	        // ...
	
	        $event = new UserRegisteredEvent($user);
	        $eventDispatcher->dispatch($event, UserRegisteredEvent::NAME);
	
	        // ...
	    }
	}
?>
```

## EventListener

Par convention, tous les `EventListener` sont placés dans le dossier `src/EventListener`. En plus d'une classe, un EventListener doit être configuré dans le fichier `config/services.yaml`.

Exemple :

```php
<?php
namespace App\EventListener;

use Symfony\Component\HttpKernel\Event\ExceptionEvent;

class ExceptionListener
{
    public function onKernelException(ExceptionEvent $event)
    {
        // ... Code à déclencher en cas d'exception
    }
}
```

```yml
services:
    App\EventListener\ExceptionListener:
        tags:
            - { name: kernel.event_listener, event: kernel.exception } # name pour le nom du service, event pour spécifier quel Event sera pris en charge par le service
```

## EventSubscriber

Par convention, tous les `EventSubscriber` sont placés dans le dossier `src/EventSubscriber`. Un EventSubscriber doit implémenter l'interface `Symfony\Component\EventDispatcher\EventSubscriberInterface`. Exemple :

```php
<?php
namespace App\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\Event\ExceptionEvent;
use Symfony\Component\HttpKernel\KernelEvents;

class EventSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents()
    {
        // retourne les événements souscrits, les méthodes associées et leurs priorités
        return [
            // On souscrit à l'événement KernelEvents::EXCEPTION
            UserRegisteredEvent::NAME=> [
                // Les 3 méthodes ci-dessous seront appelées,
                // le nombre (optionnel) permet de savoir l'ordre d'appel.
                // Les méthodes associées au nombre le plus haut seront exécutées en premier,
                // puis celles avec un nombre plus bas, etc.
                ['processEvent', 10],
                ['logEvent', 0],
                ['notifyEvent', -10],
            ],
        ];
    }

    public function processEvent(UserRegisteredEvent $event)
    {
        // ...
    }

    public function logEvent(UserRegisteredEvent $event)
    {
        // ...
    }

    public function notifyEvent(UserRegisteredEvent $event)
    {
        // ...
    }
}

```

## Sources

- https://symfony.com/doc/current/event_dispatcher.html#creating-an-event-listener
- https://symfonycasts.com/screencast/deep-dive/events
