# Liskov Substitution Principle (LSP)

## L'Objectif : Les sous-classes ne doivent pas surprendre

Le Principe de Substitution de Liskov, formulé par Barbara Liskov, est une extension du Principe Ouvert/Fermé. Il garantit que l'héritage est utilisé de manière correcte et prévisible. Sa définition formelle est :

> **Si S est un sous-type de T, alors on doit pouvoir substituer des objets de type T par des objets de type S sans altérer les propriétés désirables du programme.**

En termes plus simples : une sous-classe (ou une classe qui implémente une interface) doit être **complètement substituable** à sa classe parente (ou son interface) sans que le code qui l'utilise ne s'en rende compte et sans créer de comportement inattendu.

Une sous-classe ne doit pas être "plus stricte" que sa classe parente. Elle doit se comporter de la même manière.

## Le Problème (Code "Avant")

Le cas d'école classique est le problème du "carré qui est un rectangle". Mathématiquement, un carré *est* un rectangle. Essayons de modéliser cela avec l'héritage.

```php
<?php
class Rectangle
{
    protected int $width;
    protected int $height;

    public function setWidth(int $width): void
    {
        $this->width = $width;
    }

    public function setHeight(int $height): void
    {
        $this->height = $height;
    }
    
    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

// Un carré est un rectangle... n'est-ce pas ?
class Square extends Rectangle
{
    // Pour qu'un carré reste un carré,
    // changer la largeur doit aussi changer la hauteur.
    public function setWidth(int $width): void
    {
        $this->width = $width;
        $this->height = $width; // <-- Effet de bord inattendu
    }

    public function setHeight(int $height): void
    {
        $this->width = $height;
        $this->height = $height; // <-- Effet de bord inattendu
    }
}
```

Maintenant, écrivons un code client qui s'attend à manipuler un `Rectangle`.
```php
function printArea(Rectangle $rectangle): void
{
    $rectangle->setWidth(5);
    $rectangle->setHeight(4);

    // Le code client s'attend logiquement à ce que l'aire soit 5 * 4 = 20
    $expectedArea = 20;
    $actualArea = $rectangle->getArea();
    
    echo "Aire attendue : $expectedArea, Aire calculée : $actualArea\n";
}
```
**Utilisation**
```php
$rect = new Rectangle();
printArea($rect); // Affiche : Aire attendue : 20, Aire calculée : 20 (OK)

$square = new Square();
printArea($square); // Affiche : Aire attendue : 20, Aire calculée : 16 (BUG !)
```
**Pourquoi est-ce un problème ?**
-   Nous avons substitué un `Rectangle` par un `Square` (son sous-type), mais cela a **changé le comportement du programme**. La fonction `printArea` ne fonctionne plus comme prévu.
-   La classe `Square` **viole le contrat** de la classe `Rectangle`. Un utilisateur de `Rectangle` s'attend à ce que `setWidth()` ne modifie QUE la largeur. La classe `Square` a un comportement surprenant et brise cette attente.
-   Pour faire fonctionner ce code, il faudrait ajouter un `if ($rectangle instanceof Square)` dans la fonction `printArea`, ce qui violerait le Principe Ouvert/Fermé.

## La Solution (Code "Après")

La solution est de reconnaître que, du point de vue du comportement, un `Square` n'est pas un `Rectangle` substituable. L'héritage n'est pas le bon outil ici. On préférera une abstraction commune.

Une meilleure modélisation serait de se baser sur le comportement, pas sur la taxonomie mathématique.

```php
<?php
interface Shape
{
    public function getArea(): int;
}

// Chaque classe implémente l'interface, mais sans lien d'héritage entre elles.
class Rectangle implements Shape
{
    private int $width;
    private int $height;

    public function __construct(int $width, int $height)
    {
        $this->width = $width;
        $this->height = $height;
    }
    
    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square implements Shape
{
    private int $side;
    
    public function __construct(int $side)
    {
        $this->side = $side;
    }
    
    public function getArea(): int
    {
        return $this->side * $this->side;
    }
}
```
**Quels sont les bénéfices ?**
-   **Prévisibilité :** Il n'y a plus de comportement inattendu. Chaque classe a sa propre logique interne, mais elles respectent toutes le contrat de l'interface `Shape`.
-   **Pas de substitution incorrecte :** Le code client qui a besoin de définir une largeur ET une hauteur dépendra explicitement d'un `Rectangle`, pas d'une `Shape` générique.
-   **L'héritage est utilisé à bon escient :** Le LSP nous force à réfléchir si l'héritage est la bonne solution. Souvent, la composition ou les interfaces sont de meilleures alternatives.

## Règles du LSP (en résumé)

Pour qu'une sous-classe respecte le LSP, elle doit suivre ces règles :
1.  Les préconditions ne peuvent pas être renforcées (ses méthodes ne doivent pas exiger plus que la méthode parente).
2.  Les postconditions ne peuvent pas être affaiblies (ses méthodes doivent garantir au moins autant que la méthode parente).
3.  Les types des paramètres de la méthode doivent être les mêmes ou plus abstraits (contravariance).
4.  Le type de retour de la méthode doit être le même ou un sous-type (covariance).
5.  Elle ne doit pas lever d'exceptions que la classe parente n'est pas censée lever.

## En Bref

| Principe | Définition | Bénéfice Principal |
| :--- | :--- | :--- |
| **LSP** | Une sous-classe doit être entièrement substituable à sa classe parente sans altérer la cohérence du programme. | Garantit que l'héritage est utilisé de manière fiable et prévisible, évitant des bugs subtils et renforçant la stabilité du code. |