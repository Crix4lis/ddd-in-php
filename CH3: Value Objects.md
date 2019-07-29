# CHAPTER 3: VALUE OBJECTS

- Value Object's task is to:
    - measure
    - quantifie
    - describe
- Value Object's properties:
    - small or/and simple class
    - **equality** is based on the content it holds (state), rather than **identity**
    - they are used quite widely
    - they are **immutable** - if you want to change value object you must replace it with new object instead of modify
 the old one

- Value Object's examples:
    - numbers
    - dates
    - monies
    - strings
    - full name
    - email
    - phone no.
    - colors
    - postal-codes

>Vale Object by example:<br>
Each $5 note has its own identity (serial number), but the cash economy relies on every $5 note having the same
**value** (state) as every other $5 note.

## Value Object vs. Entity by example
- Value Object - When peaple exhange dollar bills, they do **dot distinguish** between each unique bill
(it's serial number), they are only concerned about the face value of the dollar bill. Dollar bill is equal to another
dollar bill when it's **property (it's value) equals to another**
- Entity - Most airlines distinguish each seat uniquely on every flight. Each seat is an entity in this context.
They might distinguish it if they need to book a seat for a specific passenger. If they do not need to book it, each
seat is identical there fore in that context that would be value object. Entity is equal to another entity if their
**unique identifiers are equal**

## Creating Value Objects
```php
class Money
{
    private $amount, $currency;
    public function __construct($amount, Currency $currency)
    { 
        $this->setAmount($amount);
        $this->setCurrency($currency);
    }
    
    private function setAmount($amount) { ... } //use setters (but private)
    private function setCurrency(Currency $currency) { ... }
    public function amount() { return $this->amount; }
    public function currency() { return $this->currency }
}
```
## Value Objects properties
### IMMUTABILITY
**ALWAYS** returns new instance in valid state instead of modifying it's properties
```php
class Money
{
    ...
    public function increaseAmountBy($amount)
    {
        return new self( // SELF keyword, not static
            $this->amount() + $amount,
            $this->currency()
        );
    }
}
```
### EQUALITY
```php
class Money
{
    ...
    public function equals(Money $m)
    {
        //note that $this->amount() is public getter, not the field itself
        return $m->currency()->equals($this->currency()) && $m->amount() === $this->amount();
    }
}
```
```php
class Currency
{
    ...
    public function equals(Currency $c)
    {
        return $c->isoCode() === $this->isoCode(); // public getter
    }
}
```

## PERSISTING VALUE OBJECTS
Entity
```php
class Product
{
    private $id, $name, Money $price;
    public fundtion __construct($id, $name, Money $price)
    {
        $this->setId($id);
        $this->setName($name);
        $this->setPrice($pirce)
    }
}
```
Client's code
```php
...
$product = new Product(
    $productRepository->nextIdentity(), //just an example
    'name',
    new Money(1, new Currency('USD')
);
$productRepo->persist($product); // saved to db

```
Methods of persisting:
- **USE EMBEDDED VALUE with Doctrine** (mapping type) [page 69] (it's easier with Doctrine >= 2.5)
- use **serialized LOB** (single column as TEXT) - I don't really like it, also the book dissuade this approach
as it serializes classes and during name refactor it generates lots of problems

### Persisting a collection of Value Objects
Entity
```php
class HistoricalProduct extends Product
{
    protected Money[] $prices;
    
    public function __construct($id, $name, Money $currentPrice, Money[] $histryPrices) { ... }
}
```
>You cannot use Embeded Values (Doctrine) as you can never know how many items in collection might be

#### Aproaches
- Collection Serialized into Singe Column - easiest but I still don't like it
- Collection Backed by a Join Table - although they are still value object they are persisted as entities with
its own id in an external table (many-to-many rel type). To resolve problem with ID being placed in value object
we must use surrogate id or or place it in extended class
    - surrogate id - the new identity is only required due to the database, there fore it's not part of a Domain
    - extended class - would break inversion principle
    
##### Reccomended surrogate id approach
```php
class Money
{
    private $amount, $currency, $surrogateId, $surrogateCurrencyIsoCode;
    publid function __construct($amount, Currency $currency) { ... }
    private function setAmount($amoun) { ... }
    private function setCurrency(Currency $c)
    {
        $this->currency = $currency;
        $this->surrogateCurrencyIsoCode = $currency->isoCode();
    }
    public function currency()
    {
        if (null === $this->currency) {
            $this->currency = new Currency($this->surrogateCurrencyJsoCode)
        }
        return $this->currency;
    }
    public function amoun()
    {
        return $this->amount;
    }
    public function equals(Money $m) { ... }
    
}
```
> page 83, 84


#### Summarization and conclusion
- Value Object vs Entity
    - if you distinguish two different objects by their internal properties (their state) it is value object
    - if you distinguish two different objects by their uniqual identity (ID) it is entity
- Value Object properties and characterisation
    - use setters - **don't omit** but make them **private**
    - favor Value Object over Entities - they are easier to create, maintain and test
    - **IMMUTABLE**
    - they **must** always be in correct state (validation in constructor)
    - use **self** over **static**
    - even within VO itself **use** it's public **getters** not fields directly
- `==` vs `===` (php behaviour)
    - `==` compares instance's class, properties and it's values
    - `===` compares references
- persisting value objects
    - embeded values in doctrine >= 2.5 [p. 69], for doctrine < 2.5 read about surrogate values
    - serialize LOB (dissuaded)
- persisting collcection of value objects
    - use surrogate id

#### TODO
- read about surrogate properties
