# Open/Closed Principle (OCP)

## L'Objectif : Ouvert à l'extension, fermé à la modification

Le Principe Ouvert/Fermé, formulé par Bertrand Meyer, est l'un des plus importants pour créer un logiciel maintenable à long terme. Sa définition est :

> **Les entités logicielles (classes, modules, fonctions, etc.) devraient être ouvertes à l'extension, mais fermées à la modification.**

Cela signifie que vous devriez être capable d'**ajouter de nouvelles fonctionnalités** à votre application **sans jamais changer le code source existant** qui a déjà été testé et validé. L'idée est d'étendre le comportement d'une classe en ajoutant du nouveau code, pas en modifiant l'ancien.

## Le Problème (Code "Avant")

Imaginons une classe `InvoiceCalculator` qui calcule le montant total d'une facture. Au début, elle ne gère que les factures standards.

```php
<?php
class Invoice
{
    public float $amount;
    public string $type; // 'standard', 'tax_free', etc.

    public function __construct(float $amount, string $type)
    {
        $this->amount = $amount;
        $this->type = $type;
    }
}

// Cette classe viole l'OCP.
class InvoiceCalculator
{
    public function calculateTotal(Invoice $invoice): float
    {
        $total = 0;
        if ($invoice->type === 'standard') {
            // Ajoute une TVA de 20%
            $total = $invoice->amount * 1.20;
        } elseif ($invoice->type === 'tax_free') {
            // Pas de TVA
            $total = $invoice->amount;
        }
        // ... d'autres types de factures ?
        
        return $total;
    }
}
```
**Pourquoi est-ce un problème ?**
-   **Fermé à l'extension :** Si demain, nous devons gérer un nouveau type de facture, par exemple une "facture avec réduction", nous sommes **obligés de modifier** la méthode `calculateTotal` en y ajoutant un nouveau `elseif`.
-   **Risque de régression :** Chaque modification du fichier `InvoiceCalculator.php` risque d'introduire un bug dans la logique de calcul existante.
-   **Complexité croissante :** La méthode `calculateTotal` va devenir de plus en plus grosse et difficile à lire à mesure que de nouvelles règles métier sont ajoutées.

## La Solution (Code "Après")

La solution consiste à utiliser des **abstractions** (interfaces ou classes abstraites) et le **polymorphisme**. Nous allons utiliser le **Pattern Strategy**.

1.  Créer une interface `InvoiceCalculationStrategy` qui définit un contrat pour le calcul.
2.  Créer des classes concrètes pour chaque type de calcul.
3.  La classe `InvoiceCalculator` utilisera la stratégie appropriée sans connaître les détails de son implémentation.

**1. L'interface (le contrat)**
```php
<?php
interface InvoiceCalculationStrategy
{
    public function calculate(float $amount): float;
}
```

**2. Les stratégies concrètes (les extensions)**
Chaque nouvelle règle de calcul est une nouvelle classe.
```php
<?php
class StandardCalculationStrategy implements InvoiceCalculationStrategy
{
    public function calculate(float $amount): float
    {
        return $amount * 1.20;
    }
}

class TaxFreeCalculationStrategy implements InvoiceCalculationStrategy
{
    public function calculate(float $amount): float
    {
        return $amount;
    }
}

// NOUVELLE FONCTIONNALITÉ : Ajoutée sans modifier le code existant !
class DiscountCalculationStrategy implements InvoiceCalculationStrategy
{
    public function calculate(float $amount): float
    {
        // 10% de réduction puis 20% de TVA
        return ($amount * 0.90) * 1.20;
    }
}
```

**3. Le `InvoiceCalculator` (fermé à la modification)**
Il ne connaît que l'interface et ne changera plus jamais.
```php
<?php
class InvoiceCalculator
{
    // Le calculateur délègue le calcul à la stratégie injectée.
    public function calculateTotal(float $amount, InvoiceCalculationStrategy $strategy): float
    {
        return $strategy->calculate($amount);
    }
}
```

**Utilisation de la nouvelle structure**
```php
$calculator = new InvoiceCalculator();

$standardInvoiceAmount = 100;
$taxFreeInvoiceAmount = 100;
$discountInvoiceAmount = 100;

// On choisit la stratégie de calcul en fonction du besoin.
echo $calculator->calculateTotal($standardInvoiceAmount, new StandardCalculationStrategy());   // 120
echo $calculator->calculateTotal($taxFreeInvoiceAmount, new TaxFreeCalculationStrategy());    // 100
echo $calculator->calculateTotal($discountInvoiceAmount, new DiscountCalculationStrategy());  // 108
```
**Quels sont les bénéfices ?**
-   **Ouvert à l'extension :** Pour ajouter un nouveau type de calcul, il suffit de créer une nouvelle classe qui implémente `InvoiceCalculationStrategy`.
-   **Fermé à la modification :** Les classes `InvoiceCalculator`, `StandardCalculationStrategy` et `TaxFreeCalculationStrategy` n'ont **jamais** été modifiées. Elles sont stables et testées.
-   **Découplage :** Le calculateur est découplé des algorithmes de calcul spécifiques.
-   **Respect du SRP :** Chaque classe de stratégie a une seule et unique responsabilité.

## En Bref

| Principe | Définition | Bénéfice Principal |
| :--- | :--- | :--- |
| **OCP** | Les entités logicielles doivent être ouvertes à l'extension, mais fermées à la modification. | Permet d'ajouter de nouvelles fonctionnalités sans réécrire ou risquer de casser le code existant, ce qui rend le logiciel plus stable et plus facile à maintenir. |