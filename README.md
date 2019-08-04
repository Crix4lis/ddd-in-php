>author: [Michał Powała](https://github.com/Crix4lis)<br>
>source repository: [ddd-in-php](https://github.com/Crix4lis/ddd-in-php)

# ddd-in-php
This repository is a study of a book [Domain-Driven Design in PHP](http://xeroxmobileprint.net/DiscoveryTable/test/folder1/Domain-Driven_Design_in_PHP.pdf)
written by *CARLOS BUENOSVINOS*, *CHRISTIAN SORONELLAS* and *KEYVAN AKBARY*:

![DOMAIIN-DRIVEN DESIGN IN PHP COVER](https://images.gr-assets.com/books/1537792285l/32284709.jpg)

## Book's code snip sets
[Domain Driven Design In PHP code snip sets (GitHub)](https://github.com/dddinphp)

## Mentioned books (I decided to point out)
- [Applying Domain Driven Design Patterns by Jimmy Nilsson](https://www.amazon.com/Applying-Domain-Driven-Design-Patterns-Examples/dp/0321268202)<br>
- [Introducing Eventstorming by Alberto Brandolini](https://leanpub.com/introducing_eventstorming)<br>
- [NoSQL Distilled: A Brief Guide to the Emerging World of Polyglot Persistence by Pramod J. Sadalag]()<br>


## Entities and surrogate Id
User contains $userId, $email, $password

```php
class DoctrineUser extends User
{
    private $surrogateUserId;
    
    public function __construct(UserId $userId, $email, $password) {
        parent::__construct($userId, $email, $password);
        $this->surrogateUserId = $user->id()
    }
}
```

### Notes
- Doctrine is an implementation of a `Data Mapper pattern`
- I always wondered where validation (assertion) should lay - **ASSERTION SHOULD BE ENCLOSED WITHIN PRIVATE SETTER**
(unless that small ValueObject - don't need setter imho, then)
- **Public** VS **private constructor** (**named** constructor):
    - **public constructor**: if object is created with all needed (necessary) properties provided from outside the class
    - **private constructor and public static named constructor**: if object is created without provided all necessary
    data (it generates it)
    - addictionally it allows (static constructor) to implement constructor method without throwing an event, and in business specific
    method that is named static constructor you can **publish events**
- If **other bounded context** generates ID, necessary is **Event Driven architecture**
- Constructing an object from plain data such as an array is called hydratation
- "Factories help us keeping the business invariants" [page 177] - read more about factories

### TODO
- read more about streams
- read more about redis and elasticsearch
- read about surrogate properties in a context to database
- read about active records (don't like it, but still I need to know more)
- read about `Data Mapper patter`
- read about `Unit of Work pattern`
- **read more about messaging (not publishing itself) events!**
- read about invariants in context of aggregates
- read about database mechanism pessimistic and optimistic concurrency control
