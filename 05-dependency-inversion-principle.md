# Dependency Inversion Principle (DIP)

## L'Objectif : Dépendre d'abstractions, pas de concrétions

Le Principe d'Inversion de Dépendance peut sembler complexe, mais il formalise une idée très puissante qui change la façon dont les modules d'un logiciel interagissent. Il est composé de deux règles :

> **A. Les modules de haut niveau ne doivent pas dépendre des modules de bas niveau. Les deux doivent dépendre d'abstractions (ex: des interfaces).**
>
> **B. Les abstractions ne doivent pas dépendre des détails. Les détails (classes concrètes) doivent dépendre des abstractions.**

Qu'est-ce que ça veut dire ?
-   **Module de haut niveau :** Le code qui contient la logique métier importante (ex: un service qui gère la politique de notification des utilisateurs).
-   **Module de bas niveau :** Le code qui gère les détails techniques d'implémentation (ex: une classe qui envoie concrètement des emails via un service externe, ou une classe qui écrit dans un fichier de log).
-   **Le flux de contrôle traditionnel (SANS le DIP) :** Haut niveau -> Bas niveau.
-   **Le flux de dépendance avec le DIP :** Haut niveau -> Abstraction <- Bas niveau.

Le mot "inversion" vient du fait qu'on **inverse la direction de la dépendance**. Au lieu que la logique métier dépende du détail technique, c'est le détail technique qui dépend d'un contrat défini par la logique métier.

## Le Problème (Code "Avant")

Imaginons un service `PasswordReminder` (module de haut niveau) qui a besoin d'accéder à une base de données pour trouver un utilisateur. Il dépend directement d'une classe `MySQLConnection` (module de bas niveau).

```php
<?php
// Module de bas niveau (un détail d'implémentation)
class MySQLConnection
{
    public function connect()
    {
        // ... logique de connexion à MySQL
        return "Données de l'utilisateur depuis MySQL";
    }
}

// Module de haut niveau
class PasswordReminder
{
    private MySQLConnection $dbConnection;

    public function __construct()
    {
        // Notre module de haut niveau est DIRECTEMENT COUPLÉ
        // à une implémentation de bas niveau.
        $this->dbConnection = new MySQLConnection();
    }
    
    public function getUserData(): string
    {
        return $this->dbConnection->connect();
    }
}
```
**Pourquoi est-ce un problème ?**
-   **Couplage fort :** Le `PasswordReminder` est intimement lié à MySQL. Si demain, nous décidons de migrer vers une base de données PostgreSQL, nous sommes **obligés de modifier la classe `PasswordReminder`**.
-   **Non réutilisable :** Le `PasswordReminder` ne peut pas fonctionner dans un environnement qui n'a pas MySQL.
-   **Difficile à tester :** Pour tester unitairement `PasswordReminder`, nous sommes obligés d'avoir une vraie connexion à une base de données MySQL. Il est impossible de "simuler" (mocker) la connexion.

## La Solution (Code "Après")

La solution est d'introduire une **abstraction** (une interface) que le module de haut niveau va définir et dont il va dépendre. Le module de bas niveau devra ensuite implémenter cette interface.

**1. L'abstraction (le contrat défini par le haut niveau)**
Le `PasswordReminder` ne se soucie pas de *comment* les données sont récupérées. Il a juste besoin de *quelque chose* qui puisse se connecter. Définissons ce besoin dans une interface.
```php
<?php
interface DBConnectionInterface
{
    public function connect(): string;
}```

**2. Le module de haut niveau (dépend de l'abstraction)**
Le `PasswordReminder` ne connaît plus `MySQLConnection`. Il ne connaît que `DBConnectionInterface`.
```php
<?php
class PasswordReminder
{
    private DBConnectionInterface $dbConnection;

    // La dépendance est maintenant injectée !
    public function __construct(DBConnectionInterface $dbConnection)
    {
        $this->dbConnection = $dbConnection;
    }
    
    public function getUserData(): string
    {
        return $this->dbConnection->connect();
    }
}
```

**3. Le module de bas niveau (dépend aussi de l'abstraction)**
La classe `MySQLConnection` doit maintenant se conformer au contrat défini par l'abstraction.
```php
<?php
class MySQLConnection implements DBConnectionInterface
{
    public function connect(): string
    {
        return "Données de l'utilisateur depuis MySQL";
    }
}

// On peut maintenant créer de NOUVELLES implémentations sans
// jamais toucher à PasswordReminder.
class PostgreSQLConnection implements DBConnectionInterface
{
    public function connect(): string
    {
        return "Données de l'utilisateur depuis PostgreSQL";
    }
}
```

**Utilisation (assemblage des pièces)**
C'est au point d'entrée de l'application (ou dans un conteneur d'injection de dépendances) que l'on choisit quelle implémentation concrète on veut utiliser.
```php
// Je choisis d'utiliser MySQL aujourd'hui.
$mysqlConnection = new MySQLConnection();
$reminder = new PasswordReminder($mysqlConnection);
echo $reminder->getUserData(); // Affiche : Données de l'utilisateur depuis MySQL

// Demain, je peux changer d'avis sans modifier PasswordReminder.
$pgsqlConnection = new PostgreSQLConnection();
$reminder = new PasswordReminder($pgsqlConnection);
echo $reminder->getUserData(); // Affiche : Données de l'utilisateur depuis PostgreSQL
```
**Quels sont les bénéfices ?**
-   **Découplage :** La logique métier (`PasswordReminder`) est complètement découplée des détails d'implémentation.
-   **Flexibilité et extensibilité :** On peut changer de système de base de données sans modifier le code métier. Le système est ouvert à l'extension (OCP).
-   **Testabilité :** Dans un test, on peut facilement créer un "mock" `MockDBConnection` qui implémente `DBConnectionInterface` et retourne des données de test sans se connecter à une vraie base de données.

## En Bref

| Principe | Définition | Bénéfice Principal |
| :--- | :--- | :--- |
| **DIP** | Les modules de haut niveau et de bas niveau doivent tous deux dépendre d'abstractions. Les détails d'implémentation doivent dépendre des abstractions, et non l'inverse. | Crée un code flexible, découplé et hautement testable en séparant la logique métier des détails techniques. C'est le principe qui rend l'**Injection de Dépendances** si puissante. |