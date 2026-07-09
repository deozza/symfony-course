# Voters

## Sommaire

<!--toc:start-->
- [Voters](#voters)
  - [Sommaire](#sommaire)
  - [Définition](#définition)
  - [Création d'un Voter](#création-dun-voter)
  - [Utilisation](#utilisation)
  - [Changer la stratégie de décision](#changer-la-stratégie-de-décision)
<!--toc:end-->

## Définition

Un Voter permet de gérer les autorisations d'accès à une ressource par un utilisateur de l'application. Utiliser un voter plutôt qu'une gestion "manuelle" dans un controller a plusieurs avantages :

- avoir une gestion plus fine des autorisations (je suis un utilisateur connecté, j'essaye de modifier une ressource de l'application, je ne suis pas propriétaire de cette ressource et je n'ai pas le rôle `ROLE_ADMIN` => je n'ai pas le droit de faire cette action)
- centraliser et réutiliser ces autorisations
  - plutôt que de les dupliquer dès qu'on en a besoin
- réunir le résultat de plusieurs Voter pour autoriser ou non
  - avoir au moins 1 Voter qui autoriser, avoir la majorité de Voter qui autorise, avoir tous les Voters qui autorisent

## Création d'un Voter

Par convention, un `Voter` est placé dans le dossier `src/Security`. Un Voter doit étendre la classe `Symfony\Component\Security\Core\Authorization\Voter\Voter`. Exemple d'un Voter qui accorde la permission pour voir ou éditer un post d'un blog :

```php
<?php
namespace App\Security;

use App\Entity\Post;
use App\Entity\User;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authorization\Voter\Vote;
use Symfony\Component\Security\Core\Authorization\Voter\Voter;

class PostVoter extends Voter
{
    // these values are arbitrary strings; you can use anything
    const VIEW = 'view';
    const EDIT = 'edit';

    protected function supports(string $attribute, mixed $subject): bool
    {
        // if the voter doesn't support this attribute, return false
        if (!in_array($attribute, [self::VIEW, self::EDIT])) {
            return false;
        }

        // only vote on `Post` objects
        if (!$subject instanceof Post) {
            return false;
        }

        return true;
    }

    protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token, ?Vote $vote = null): bool
    {
        $user = $token->getUser();

        if (!$user instanceof User) {
            // the user must be logged in; if not, deny access
            $vote?->addReason('The user is not logged in.');
            return false;
        }

        // you know $subject is a Post object, thanks to `supports()`
        /** @var Post $post */
        $post = $subject;

        return match($attribute) {
            self::VIEW => $this->canView($post, $user),
            self::EDIT => $this->canEdit($post, $user, $vote),
            default => throw new \LogicException('This code should not be reached!')
        };
    }

    private function canView(Post $post, User $user): bool
    {
        // if they can edit, they can view
        if ($this->canEdit($post, $user)) {
            return true;
        }

        // the Post object could have, for example, a method `isPrivate()`
        return !$post->isPrivate();
    }

    private function canEdit(Post $post, User $user, ?Vote $vote): bool
    {
        // this assumes that the Post object has a `getAuthor()` method
        if ($user === $post->getAuthor()) {
            return true;
        }

        $vote?->addReason(sprintf(
            'The logged in user (username: %s) is not the author of this post (id: %d).',
            $user->getUsername(), $post->getId()
        ));

        return false;
    }
}
```

Explications :

- la fonction `supports` est appelée en premier
  - elle détermine si le Voter est concerné par l'action en cours
    - si la permission est `view` ou `edit`
    - si la ressource concernée par l'action est une instance de la classe `Post`
  - si il n'est pas concerné, le voter *s'abstient*
- la fonction `voteOnAttribute` contient la logique d'autorisation et renvoie le résultat

## Utilisation

Exemple dans un Controller :

```php
<?php
use Symfony\Component\Security\Http\Attribute\IsGranted;

class PostController extends AbstractController
{
    #[Route('/posts/{id}', name: 'post_show')]
    // check for "view" access: calls all voters
    // the second argument is the name of the controller argument passed to the voter
    #[IsGranted('view', 'post')]
    public function show(Post $post): Response
    {
        // ...
    }

    #[Route('/posts/{id}/edit', name: 'post_edit')]
    // check for "edit" access: calls all voters
    // the second argument is the name of the controller argument passed to the voter
    #[IsGranted('edit', 'post')]
    public function edit(Post $post): Response
    {
        // ...
    }
}
```

## Changer la stratégie de décision

- `affirmative` (par défaut) : au moins 1 voter accorde la permission
- `consensus` :  la majorité des voters accorde la permission
- `unanimous` : tous les voters doivent accorder la permission
- `priority` : le résultat du premier voter qui ne s'abstient pas est retenu

Dans le fichier `config/packages/security.yaml` :

```yml
# config/packages/security.yaml
security:
    access_decision_manager:
        strategy: affirmative 
        allow_if_all_abstain: false
```

## Sources

- https://symfony.com/doc/current/security/voters.html
- https://cours.davidannebicque.fr/symfony/semestre-4/seance-10-voters
- https://writecode.fr/tutoriel/la-gestion-des-permissions-avec-les-voters
