# Interface Segregation Principle (ISP)

## L'Objectif : Pas de dépendance sur des méthodes inutiles

Le Principe de Ségrégation d'Interface est l'un des plus simples à comprendre et à appliquer. Sa définition est :

> **Un client ne devrait pas être forcé de dépendre de méthodes qu'il n'utilise pas.**

En d'autres termes, il vaut mieux avoir **plusieurs petites interfaces spécifiques** qu'une seule grosse interface générique. Si une classe implémente une interface, elle doit être capable de donner un sens à **toutes** les méthodes de cette interface.

Si vous vous retrouvez à implémenter une interface et à laisser certaines de ses méthodes vides ou à y lever une exception `NotImplementedException`, c'est un signe clair que vous violez l'ISP.

## Le Problème (Code "Avant")

Imaginons une interface `WorkerInterface` qui décrit les capacités d'un travailleur dans une entreprise.

```php
<?php
// Interface "fourre-tout" qui viole l'ISP
interface WorkerInterface
{
    public function work(): void;
    public function eat(): void;
    public function sleep(): void;
}
```

Maintenant, nous avons deux types de travailleurs : des humains et des robots.

```php
class HumanWorker implements WorkerInterface
{
    public function work(): void
    {
        echo "L'humain travaille.\n";
    }

    public function eat(): void
    {
        echo "L'humain mange.\n";
    }
    
    public function sleep(): void
    {
        echo "L'humain dort.\n";
    }
}

class RobotWorker implements WorkerInterface
{
    public function work(): void
    {
        echo "Le robot travaille.\n";
    }

    // Un robot ne mange pas. Que fait-on de cette méthode ?
    public function eat(): void
    {
        // On la laisse vide ? On lève une exception ?
        // Dans les deux cas, c'est une implémentation forcée et incorrecte.
    }
    
    // Un robot n'a pas besoin de dormir.
    public function sleep(): void
    {
        // Même problème ici.
    }
}
```
**Pourquoi est-ce un problème ?**
-   **Implémentation forcée :** La classe `RobotWorker` est obligée d'implémenter les méthodes `eat()` et `sleep()` même si elles n'ont aucun sens pour elle. Cela mène à du code vide ou qui lève des exceptions, ce qui est trompeur.
-   **Dépendances inutiles :** Un code client qui a besoin de faire travailler un `RobotWorker` dépend aussi des méthodes `eat()` et `sleep()`, même s'il ne les appellera jamais.
-   **Couplage fort :** Si on ajoute une nouvelle capacité à `WorkerInterface` (par exemple, `socialize()`), toutes les classes qui l'implémentent (y compris `RobotWorker`) seront "cassées" et devront être modifiées, même si cette nouvelle capacité ne les concerne pas.

## La Solution (Code "Après")

La solution est de **ségréger** (séparer) la grosse interface en plusieurs petites interfaces, chacune ayant une responsabilité claire et ciblée.

**1. Les nouvelles interfaces, petites et spécifiques**
```php
<?php
interface WorkableInterface
{
    public function work(): void;
}

interface EatableInterface
{
    public function eat(): void;
}

interface SleepableInterface
{
    public function sleep(): void;
}
```

**2. Les classes implémentent uniquement ce dont elles ont besoin**
Les classes sont maintenant libres de choisir les contrats qu'elles peuvent réellement respecter.
```php
class HumanWorker implements WorkableInterface, EatableInterface, SleepableInterface
{
    public function work(): void
    {
        echo "L'humain travaille.\n";
    }

    public function eat(): void
    {
        echo "L'humain mange.\n";
    }
    
    public function sleep(): void
    {
        echo "L'humain dort.\n";
    }
}

// Le robot n'implémente que ce qui le concerne.
class RobotWorker implements WorkableInterface
{
    public function work(): void
    {
        echo "Le robot travaille.\n";
    }
}
```

**3. Le code client dépend de la plus petite interface possible**
Le code qui gère le travail ne dépend que de l'interface `WorkableInterface`.
```php
function manageWork(WorkableInterface $worker)
{
    $worker->work();
}

// Le code fonctionne maintenant pour les deux types de travailleurs,
// et ne dépend que de ce qui est strictement nécessaire.
manageWork(new HumanWorker()); // Affiche : L'humain travaille.
manageWork(new RobotWorker()); // Affiche : Le robot travaille.
```
**Quels sont les bénéfices ?**
-   **Haute cohésion :** Chaque interface a une seule et unique responsabilité.
-   **Découplage :** Les classes ne dépendent que des contrats qui les concernent.
-   **Flexibilité :** Il est facile de créer de nouvelles combinaisons. Un "cyborg" pourrait implémenter `WorkableInterface` et `EatableInterface`, mais pas `SleepableInterface`.
-   **Pas d'implémentation vide :** On évite le code inutile et trompeur.

## En Bref

| Principe | Définition | Bénéfice Principal |
| :--- | :--- | :--- |
| **ISP** | Un client ne devrait pas être forcé de dépendre de méthodes qu'il n'utilise pas. Il faut préférer des interfaces petites et spécifiques à de grosses interfaces générales. | Réduit le couplage, augmente la flexibilité et la cohésion du code en s'assurant que les classes n'implémentent que les contrats qu'elles peuvent pleinement respecter. |