# Single Responsibility Principle (SRP)

## L'Objectif : Chaque classe a une seule et unique mission

Le Principe de Responsabilité Unique est peut-être le plus simple à énoncer, mais l'un des plus difficiles à maîtriser. Sa définition formelle est :

> **Une classe ne devrait avoir qu'une seule raison de changer.**

En termes plus simples, cela signifie qu'une classe ne devrait avoir **qu'une seule responsabilité**, une seule et unique tâche à accomplir dans le système. Si une classe fait à la fois la gestion de la base de données ET la mise en forme de l'affichage, elle viole ce principe car elle a deux raisons de changer : une modification du schéma de la base de données, ou une modification du design de l'affichage.

## Le Problème (Code "Avant")

Imaginons une classe `Report` qui est responsable de contenir les données d'un rapport, mais aussi de le formater en JSON.

```php
<?php
class Report
{
    private string $title;
    private string $content;

    public function __construct(string $title, string $content)
    {
        $this->title = $title;
        $this->content = $content;
    }

    public function getTitle(): string
    {
        return $this->title;
    }

    // Cette méthode viole le SRP. La classe gère maintenant AUSSI le formatage.
    public function formatAsJson(): string
    {
        return json_encode([
            'title' => $this->title,
            'content' => $this->content,
        ]);
    }
}

// Utilisation
$report = new Report('Rapport Mensuel', 'Les ventes sont en hausse...');
echo $report->formatAsJson();
```
**Pourquoi est-ce un problème ?**
-   **Deux raisons de changer :** La classe `Report` doit être modifiée si la structure des données du rapport change (ajout d'une date, par exemple), **ET** si le format de sortie JSON doit changer (ajouter une clé `meta`, par exemple).
-   **Couplage fort :** La logique métier (les données du rapport) est couplée à la logique de présentation (le format JSON).
-   **Difficile à tester :** Tester la logique de formatage est lié à l'objet `Report`.
-   **Non réutilisable :** Que faire si nous voulons formater le rapport en HTML ou en PDF ? Nous devrions ajouter encore plus de méthodes à cette classe, la rendant de plus en plus complexe et fragile.

## La Solution (Code "Après")

La solution est de séparer les responsabilités. Nous allons créer deux classes, chacune avec sa propre mission :
1.  **`Report`** : Sa seule responsabilité est de **contenir les données** du rapport.
2.  **`JsonReportFormatter`** : Sa seule responsabilité est de **formater un objet `Report`** en JSON.

**1. La classe `Report` (simplifiée)**
Elle ne s'occupe plus du formatage.
```php
<?php
class Report
{
    public string $title;
    public string $content;

    public function __construct(string $title, string $content)
    {
        $this->title = $title;
        $this->content = $content;
    }
}
```

**2. La nouvelle classe `JsonReportFormatter`**
Elle est spécialisée dans une seule tâche.
```php
<?php
class JsonReportFormatter
{
    public function format(Report $report): string
    {
        return json_encode([
            'title' => $report->title,
            'content' => $report->content,
        ]);
    }
}
```

**Utilisation de la nouvelle structure**
```php
// Chaque objet a sa propre responsabilité
$report = new Report('Rapport Mensuel', 'Les ventes sont en hausse...');
$formatter = new JsonReportFormatter();

// On combine les deux pour obtenir le résultat
$jsonOutput = $formatter->format($report);
echo $jsonOutput;
```
**Quels sont les bénéfices ?**
-   **Une seule raison de changer :** `Report` change si les données changent. `JsonReportFormatter` change si le format JSON change.
-   **Découplage :** La donnée est indépendante de sa présentation.
-   **Extensibilité :** Si demain nous avons besoin d'un format HTML, il suffit de créer une nouvelle classe `HtmlReportFormatter` sans jamais toucher aux classes existantes.
-   **Testabilité :** On peut tester le formateur indépendamment de l'objet `Report`.

## En Bref

| Principe | Définition | Bénéfice Principal |
| :--- | :--- | :--- |
| **SRP** | Une classe ne devrait avoir qu'une seule responsabilité, et donc une seule raison de changer. | Rend le code plus facile à comprendre, à tester et à maintenir en isolant les responsabilités. |