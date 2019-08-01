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

### Notes
- Doctrine is an implementation of a `Data Mapper pattern`
- I always wondered where validation (assertion) should lay - **ASSERTION SHOULD BE ENCLOSED WITHIN PRIVATE SETTER**
(unless that small ValueObject - don't need setter imho, then)
- **Public** VS **private constructor** (**named** constructor):
    - **public constructor**: if object is created with all needed (necessary) properties provided from outside the class
    - **private constructor and public static named constructor**: if object is created without provided all necessary
    data (it generates it)
- If **other bounded context** generates ID, necessary is **Event Driven architecture**

### TODO
- read more about streams
- read more about redis and elasticsearch
- read about surrogate properties in a context to database
- read about active records (don't like it, but still I need to know more)
- read about `Data Mapper patter`
- read about `Unit of Work pattern`
