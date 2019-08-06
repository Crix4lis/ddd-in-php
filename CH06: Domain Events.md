# CHAPTER 6: DOMAIN EVENTS
"Domain Events are one specific type of event used for notifying Domain changes to local or remote Bounded Contexts."
> page 123

"(...) an occurrence capture of something what happened in the domain."
> by Vaugh Vernon [page 123]

"a full-fledged part of the domain model, a representation of something that happened in the domain.
Ignore irrelevant domain activity while making explicit the events that the domain experts want to track or
be notified of, or which are associated with statechange in the other model objects."
> by Eric Evans [page 123]

"captures the memory of something interesting which affects the domain"
> Martin Fowler [page 123]

1. Domain "persists itself" and throws event that is to be persistend **in the same transaction**
1. Batch process takse Domain Event (from persistence layer) and queues it to (for ex. Rabbit)
    - event gets distributed into two queues, one for the same local bounded context, and another for external

## Persisting Domain Events
Domain Events are persisted in **Event Store** which is **domain events repository**

Domain Event Store repository inteface
```php
interface Event Store
{
    public function append(DomainEvent $event);
    public function allStoredEventsSince($eventId);
}
```
Domain Event Store Doctrine repository
```php
class DoctrineEventStore extends EntityRepository implements EventStore
{
    private $serializer
    
    public function append(DomainEvent $domainEvent)
    {
        $storedEvent = new StoredEvent(
            get_class($domainEvent),
            $domainEvent->occuredOn(),
            $this->serializer()->serialize($domainEvent, 'json)
        );
        
        $this->getEntityManager()->persiste($storedEvent);
    }
    
    public function allStoredEventsSince($eventId)
    {
        $query = $this->createQueryBuilder('e');
        if ($eventId) {
            $query->where('e.eventId > :eventId');
            $query->setParameters(['eventId' => $eventId);
        }
        $query->orderBy('e.eventId');
        
        return $query->getQuery()->getResult();
    }
    
    private function serializer()
    {
        if (null === $this->serializer) {
            $this->serializer = SerializerBuilder::create()->build();
        }
        
        return $this->serializer;
    }
}
``` 
Stored Event - entity mapped to database
```php
class StoredEvent implements DomainEevnt
{
    private $eventId; //real database id
    private $eventBody;
    private $occuredOn;
    private $typeName;
    
    public function __construct( ... );
    // getters
}
```

## Publishing Domain Events
Publishing domain events shall be done via Observer Pattern

clients class (Aggregate)
```php
class User
{
    protected $userId, $email, $password;
    
    protected function __construct(UserId $userId, $email, $password)
    {
        $this->setUserId($userId);
        $this->setEmail($email);
        $this->setPassword($password);
    }
    
    public static function fromRawDate($data)
    {
        return new self(new UserId($data['user_id']), $data['email'], $data['password']);
    }
    
    public static function registerUser(UserId $userId, $email, $password)
    {
        $newUser = new self(new UserId($data['user_id']), $data['email'], $data['password']);
        $newUser->publishEvent(new UserRegistered($this->userId));
    }
    
    protected function publishEvent(Event $e)
    {
        DomainEventPublisher::instance()->publish($e);
    }
    
}
```

publisher class (singleton xD)
```php
class DomainEventPublisher
{
    private $subscribersn, $instance= null;
    
    publicstatic function instance()
    {
        if (null===static::$instance) {
            static::$instance = newstatic();
        }
        
        return static::$instance;
    }
    
    private function __construct()
    {
        $this->subscribers = [];
    }
    
    public function __clone()
    {
        throw new\BadMethodCallException('Clone is not supported');
    }
    
    public function subscribe(DomainEventSubscriber $aDomainEventSubscriber)
    {
        $this->subscribers[] = $aDomainEventSubscriber;
    }
    
    public function publish(DomainEvent $anEvent)
    {
        foreach($this->subscribersas$aSubscriber) {
            if ($aSubscriber->isSubscribedTo($anEvent)) {
                $aSubscriber->handle($anEvent);
            }
        }
    }
}
```
Event subscriber interface
```php
interface DomainEventSubscriber
{
    public function handle(DomainEvent $aDomainEvent);
    public function isSubscribedTo(DomainEvent $aDomainEvent): bool;
}
```

Persisting Domain Events Subscriber
```php
class PersistDomainEventSubscriber implements DomainEventSubscriber
{
    private $eventStore;
    public function__construct( ... )
    
    public function handle($aDomainEvent) { $this->eventStore->append($aDomainEvent); }
    public function isSubscribedTo($aDomainEvent) {return true; }
}
```

**REMEMBER TO FIND A PLACE IN AN APPLICATION / FRAMEWORK WHERE YOU CAN SET UP SUBSCRIBER WITH A LISTENER**

## Spreading Domain events via
1. Messaging system
    > page [from 142 to 153]
    - Needed components (classes / layers):
        - notification service
            - composed from:
                - eventStore (event repository - to collect saved events from the database)
                - published message tracker - I guess that's the service that collects events from database and
                  and marks them as published 
                - message producer - rabbit class implementation for example
            - with single method:
                - publishNotifications (via exchange)
                
        
2. REST
    > page [153] 

#### Summarization and conclusion
- events **must be** persisted in the **same transaction** that aggregate changes its state
- events **must be** distributed to at least two queues, one altering local Context (the same from which it was thrown)
and for the external contexts
    - that second queue might be published to Logs Server or Data Lake or Data Mining
    - application that does not have access to the messaging system **must** provide those events via **REST API**
- events should be **represented** as **verbs** in the **past** form
- the **minimum information** that event can contain is an **entity id** it was generated by and **datetime stamp**
it occured at
- save your domain events in the same transaction before pushing it to the external world
- domain events are persisted in **Event Store** which is **domain events repository**
- because there are different events their body is serialized
- publishing domain events shall be done from entity or value object, try **not to** publish them form services
- publishind event from domain is not messaging them! You can message an event after it's saved in the database
    (publishing is saving)

#### Notes
How to share events via REST API? You cannot have track which event was collected by which system. I guess that
logic is to be implemented in external system.. Does not sound terrific..
