# CHAPTER 4: ENTITIES
No matter how many times data in the class changes, their identities remain the same (like person, or order).

Entity
```php
namespace Domain;

class Order
{
    private $id, $amount;
    public function __construct(OrderId $id, Amont $amount) { ... }
    public function amount() { return $this->amount; }
    // rest of getters
}
```
Value Object as Identity (it's not necessary but I would recommend that)
```php
namespace Domain;

class OrderId // Value Object
{
    private $id;
    // constructor, getter, equalsTo
}
```

## Entity identity
### Four mechanisms to provide entity identity
1. persistence provides identity
1. client provides identity
1. application provides identity - use UUID (ramsey UUID - PHP lib)
1. another bounded context provides identity

#### Surrogate identity
If persistence layer provides identity and it imposes for ex. integer identity but domain requires different type of
identity, use surrogate identity

OrderId Value Object
```php
class OrderId
{
    private $id;
    public function __construct ($id) { ... }
    public function id() { return $this->id; }
    public function equalsTo(OrderId $orderId) { return $orderId->id() === $this->id; } //needed for the identity
}
```

[Layer Supertype](https://martinfowler.com/eaaCatalog/layerSupertype.html) class
```php
namespace Domain;

// note it's methods are protected!
abstract class IdentifiableDomainObject
{
    private $id; //this is surrogate property - it is here only for db purpose
    protected function id() { return $this->id; }
    protected function setId($id) { $this->id = $id; }
}
```

Domain class
```php
namespace domain;

class Order extends identifabledomainobject
{
    private $orderid;
    
    public function __construct()
    public function orderId(OrderId $orderId, Amount $amount) { ... }
    {
        if (null === $this->orderid) {
            $this->orderId = new orderId($this->id()); //creating domain object id from surrogate id, but it's not necessary
        }
        
        return $this->orderid;
    }
}
```

#### Client provides identity
ISBN Value Object
```php
namespace Domain;

class ISBN
{
    private $isbn;
    private function __construct($anIsbn) { $this->setIsbn($anIsbn); }
    
    private function setIsbn($anIsbn)
    {
        $this->assertIsbnIsValid($anIsbn,'The ISBN is invalid.');
        $this->isbn=$anIsbn;
    }
    
    public static function create($anIsbn) { return new static($anIsbn); }
    private function assertIsbnIsValid($anIsbn, $string) { // ... Validates an ISBN code }
}
```
Entity
```php
class Book
{
    private $isbn; $title;
    public function __construct(ISBN $isbn, $title) { ... }
}
```
Client code
```php
$book = new Book(ISBN::create('...'), 'Book Title');
```

#### Application provides identity
f the client cannot provide the identity generally the preferred way to handle the identity operations to let the
application generate the identities, usually through a UUID.

Repository that generates identity (OrderId VO from database provided id used)
```php
namespace Domain;

class DoctrineOrderRepository extends EntityRepository implements OrderRepository
{
    public function nextIdentity()
    {
        return OrderId::create(strtoupper(Uuid::uuid4()));
    }
    public function add(Order$anOrder) { $this->getEntityManager()->persist($anOrder); }
    publicfunction remove(Order$anOrder) { $this->getEntityManager()->remove($anOrder); }
}
```

#### Another bounded context provides identity
"Probably this would be the most complex identity generation strategy, because it enforces to havea local entity to be
dependent not only on local bounded context events, but in external bounded contexts events. So in terms of maintenance,
the cost would be high.Other Bounded Context provides of some UI widget to select the identity of the local entity.
This can even grab some properties of the remote entity to its own.When synchronization is needed between the entities
of the Bounded Contexts, usually can be achieved with an Event Driven architecture on each of the Bounded Context that
need to be notifiedwhen the original entity is changed."
> Page 85

## Entity validation
### Entity fields validation
That's obvious, just keep in mind that it should be done in private setters and each validation path should be
separate function for example if title must not be shorter or longer that specified value, create not too short
validation function and not too long validate function

### Entire entity validation (like whole state validation)
Adding **such validation** to an entity is an **anti-pattern**!

Validator general class
```php
abstract class Validator
{
    private $validationHandler
    public function __construct(ValidationHandler $hander) { ... }
    protected function handerError($error)
    {
        $this->validationHander->handerError($error);
    }
    abstract public function validate();
}
```

Validation Handler class
```php
class LocationValidationHander implements Validationhandler
{
    public function handlerCityNotFoundInCountry();
    public function handerInvalidPostalCodeForCity();
}
```

Entity to validate
```php
class Location
{
    private $country, $city, $postalCode;
    public funciton __construct(Country $country, City $city, PostalCode $postalCode) { ... }
    // getters
}
```

Specific location validator
```php
class LocationValidator extends Validator
{
    private $location;
    public function __construct(Location $location, LocationValidationHandler $handler) { ... }
    
    public function validate()
    {
        if (!$this->location->country()->hasCity($this->location->city())) {
            $this->handler->handlerCityNotFoundInCountry();
        }
        
        if (!$this->location->City()->isPostalCodeValid($this->location->postalCode())) {
            $this->$handler->handerInvalidPostalCodeForCity();
        }
    }
}
```

Client's code lays within Location itself (but splits responsibilities - it delegatees validation to another class)
```php
class Location
{
    ...
    public function validate(ValidationHandler $validationHandler)
    {
        $validator = new LocationValidator($this, $validationHander);
        $validator->validate();
    }
}
```

### Object composition validation
"Validating object compositions can be complicated, because of this, the preferred way of achieving this is through
a Domain Service. The service then communicates with repositories in order t or retrieve the valid Aggregate. Due to
the likely complex object graphs that can be created, an Aggregate could be in an intermediate state, requiring
other aggregates to be validated before-hand.We can use Domain Events to notify other parts of the system that a
particular element has been validated."
> Page 100

#### Summarization and conclusion
- Entity lays in **domain** layer
- Repostiory lays in **infrastructure** layer
- RepositoryInterface lays in **domain** layer
- For the applicationd provided identity use UUID - **generate it within repository** not aggregate/entity
- Surrogate ID is the one that lays in the domain only for db purpose it's achieved by [**Layer Supertype**](https://martinfowler.com/eaaCatalog/layerSupertype.html)
- I always wondered where should lay validation (assertion) - **ASSERTION SHOULD BE ENCLOSED WITHIN PRIVATE SETTER**
- If you create object with all needed properties provided from outside the class - use constructor, 
otherwise use private constructor and public static named constructor
- If **other bounded context** generates ID, necessary is **Event Driven architecture**
- Validation of single fields must be done via private setters and each single rule should be single private validation
method
- Validation of whole entity state should be done via another Validator class (and handled by validation handler class)
- **Validation composed object** should be done via **domain service**

#### Notes
I still don't know how whole entity validation should look like. The book does not say directly how to "break" the
flow when entity state is not correct, but I think it must break it somehow because logging is not enough so the
validation handler should throw some kind of assertion exception.

#### TODO
Read about active records (don't like it, but still I need to know more)
