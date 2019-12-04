>author: [Michał Powała](https://github.com/Crix4lis)<br>
>source repository: [ddd-in-php](https://github.com/Crix4lis/ddd-in-php)

# ddd-in-php
This repository is a study of a book [Domain-Driven Design in PHP](https://www.amazon.com/Domain-Driven-Design-PHP-Carlos-Buenosvinos-ebook/dp/B06ZYRPHMC/ref=sr_1_1?keywords=Domain-Driven+Design+in+PHP&qid=1575460294&sr=8-1)
written by *CARLOS BUENOSVINOS*, *CHRISTIAN SORONELLAS* and *KEYVAN AKBARY*:

![DOMAIIN-DRIVEN DESIGN IN PHP COVER](https://images.gr-assets.com/books/1537792285l/32284709.jpg)

## Book's code snip sets
[Domain Driven Design In PHP code snip sets (GitHub)](https://github.com/dddinphp)

## Mentioned books (I decided to point out)
- [Applying Domain Driven Design Patterns by Jimmy Nilsson](https://www.amazon.com/Applying-Domain-Driven-Design-Patterns-Examples/dp/0321268202)<br>
- [Introducing Eventstorming by Alberto Brandolini](https://leanpub.com/introducing_eventstorming)<br>
- [NoSQL Distilled: A Brief Guide to the Emerging World of Polyglot Persistence by Pramod J. Sadalag](https://bigdata-ir.com/wp-content/uploads/2017/04/NoSQL-Distilled.pdf)<br>
- [Patterns of Enterprise Application Architecture by Martin Fowler](https://www.amazon.com/Patterns-Enterprise-Application-Architecture-Martin-ebook/dp/B000OZ0NAI)<br>
- [Enterprise Integration Patterns by Gregor Hohpe and Bobby Woolf](https://www.amazon.co.uk/Enterprise-Integration-Patterns-Designing-Addison-Wesley/dp/0321200683)<br>
- [Building Microservices by Sam Newman](https://www.goodreads.com/book/show/22512931-building-microservices)

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
- Command buses and message systems are used outside the domain model!!! Domain model only published events
but all the rest is handled in app layer or infrastructure layer!
- "AsMartin Fowler said in the PoEAA book, the first law of distributed systems is always:Don’t distribute" ❤️
- on message subscribes run commands to the domain using CommandBus (message subscribers are on infrastructure)

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
- read about data-structures, like Sets (repository is an implementation of set)
- read more about data transformers****
- read about aggregates and bounded contexts, how does two different aggregate roots within single context cooperate
