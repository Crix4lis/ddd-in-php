# CHAPTER 2: ARCHITECTURE STYLES

## Layered architecture - to split up concepts, create layers for each different concern
From top to bottom (higher abstraction to lower)
1. LAYER: USER INTERFACE - encapsulates orchestration of all following layers
2. LAYER: APPLICATION AND DOMAIN - encapsulates data access and manipulation
3. LAYER: INFRASTRUCTURE - handle infrastructure concerns

In Layered Architecture each layer must be tightly coubled with the layers **beneath** it. The core idea is that
it **does separate different components and concerns** of an application. It's the same concept as with MVC
architecture, actually it follows those rules (MVC one example of layared achitecture)

## Hexagonal Architecture (ports and adapters) - exposes requirements such as blogs, static pages, and so on
To create Hexagonal Architecture you must Invert dependencies which means that core layer must be unaware of
infrastructural concerns. You only need follow Dependency Inversion Principle.

> Dependency Inversion Principle (DIP)<br>
High-level modules should not depend on low-level modules.
Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.

If you extort interfaces in domain layer (for example for infrastructure), that domain becomes unaware of
infrastructure implementation. Hexagonal architecture "talks" in terms **outside** and **inside** rather than
**lower** or **higher**. Everything that wants to come **inside** (like external application via API
or data from datastorage) must come inside via exorted port (which lays "inside") throught adapter (which lays
"outiside" and is an implementation of that port and encapsulates implementation details from outside word
(like ralational database or api calls)

**Port**<br>
```php
interface RealRepository
{
    public function byId(RealSthId $id);
    public function add(RealSth $entity);
}
```
**Adapter**<br>
```php
class PDORealRepository implements RealRepository
{
    private $db;
    public function __construct(PDO $db) { ... }
    ...
}
```
**Core layer**<br>
```php
class RealService
{
    private $repo;
    /** this is it! you "inject" interface instead of real class */
    public function __construct(RealRepository $repo) { ... }
}
```

## CQRS - to expose aggregate details to the different clients
It separates queries which only can read current state of a model (Read model) from
commands which only can change state of a model (Write model)

### Write model
It is made up from:
- **Repository interface** which is deprived of all read methods except ```byId()``` which is responsible
for loading data from storage
    ```php
    interface Repository
    {
        public function save(Repository $repo);
        public function byId(RepositoryId $id);
    {
    ```

- **Aggregate root class** which consists only those methods that are able to change aggregate state
which then can publish domain events
    ```php
    abstract class AggregateRoot
    {
        private $recordedEvents = [];
        
        /** adds new model state change */
        protected function recordThat(DomainEvent $dEvent)
        {
            $this->recordedEvents[] = $dEvent;
        }
        
        /** truly changes modle state */
        protected function applyThat(DomainEvent $dEvent)
        {
            $modifier = 'apply' . get_class($dEvent);
            $this->$modifier($dEvent); // I don't like it, it just example ripped from the book
        }
        
        /** propagates domain event */
        protected function publishThat(DomainEvent $dEvent)
        {
            DomainEventPublisher::getInstance()->publish($dEvent); // I don't like it either
        }
        
        /** "main" method to change model state */
        protected function recordApplyAndPublishThat(DomainEvent $dEvent)
        {
            $this->recordThat($dEvent);
            $this->applyThat($dEvent);
            $this->publishThat($dEvent);
        }
        ... //there are some methods missed but I just focused on the most important ones (for me)
    }
    ```
    ```php
    class RealAggregateRoot extends AggregateRoot
    {
        private $id, $title, $content, $published = false, $categries; //stfu!
        
        public function __construct(AggregateId $id) {...}
        
        /** never liked the idea of "static constructor" but it does make sense here. I wonder if it's easy to unit test it
         *  the mothod presents business-specific behavior */
        public static function writeNewFrom($title, $content)
        {
            $id = RealAggregateRootId::create();
            $aggregate = new static($id);
            $aggregate->recordApplyAndPublishThat(new PostWasCreated($id, $title, $content));
        }
      
        /** another business-specific behavior */
        public function publish()
        {
            $this->recordApplyAndPublishThat(new PostWasPublished($this->id));
        }
        
        /** business specific methods */
        public function categorizeIn(CategoryId $categoryId)
        {
            $this->recordApplyAndPublishThat(new PostWasCategorized($this->id, $categoryId));
        }
        public function changeContentFor($newContent) {...}
        public function changeTitleFor($newTitle) {...}
        
        /** all above actions that change model state, TRULY change model state only VIA domain events */
        protected function applyPostWasCreated(PostWasCreated $e) {
            $this->id = $event->id();
            $this->title = $event->title();
            $this->content = $event->content();
        }
        protected function applyPostWasPublished(PostWasPublished $e) {$this->published = true;}
        protected function applyPostWasCategorized(PostWasCategorized $e) {...}
        protected function applyPostContentWasChanged(PostContentWasChanged $e) {...}
        ...
    }
    ```
    
### Read Model
All read concerns are treated as reporting processes, an infrastructure concert. In genera, when using CQRS, the Read
Model is subject to the needs of the UI and how complex the views compounding the UI are. It means query is on
infrastructure layer (in Hexagonal Architecture). **It does not need object-relational mapper (ORM), as this might be
an overkill.

### Synchronizing Read and Write models
- Read models is synchronized with Write model by **capturing domain events** throw by write model.
- For **each domain event** there must be executed specific projection
- There can be different kinds of projections depending on infrastructure. For example there can be
  Elasticsearch projection

Projection interface:
```php
interface Projection
{
    public function listensTo();
    public function project(Event $e);
}
```

**Elasticsearch** projection for **PostWasCreated** event
```php
namespace Infrastructure\...

class PostWasCreatedProjection implements Projection
{
    private $client;
    
    public function __construct(Client $client) {$this->client = $client;}
    public function listensTo() {return PostWasCreated::class;}
    
    public function project(Event $e)
    {
        $this->client->index([
            'index' => 'posts',
            'type' => 'post',
            'id' => $e->getPostId(),
            'body' => [
                'content' => $e->getPostContent(),
                ...
            ]
        ]);
    }
}
``` 
**Projector (observer)** - this is the place where Messaging system could be used
```php
namespace Infrastructure...

class Projector {
    private $projections = [];
    public function register(array $projections) { ... }
    public function project(Event[] $events) { ... }
}
```

#### Summarization and conclusion
- CQRS is made from:
    - Write model (aggregate itself)
    - Read model (direct query to data storage)
    - Synchronization read and write model:
        - projection (per event)
        - projector (event listener / observer)
- write model:
    - domain events **can be** instantiated directly from within aggregates themself
    - aggregate state **must be** changed only via domain events: `$this->id = $event->getId();`
    - methods that represent business logic model state change **do not** directly change aggregates state.
    They do it via domain events (which they can freely create)
    - **only** methods that represent business logic model state change **can and must be** `public`
    - all other methods **must be** `protected` or `private` and be hidden from aggregate's client code
- synchronization:
    - projector is a kind of **domain event listener**
    - projection is subject (in terms of observer pattern)
    - projector is the place where you can use RabbitMQ if you desire asynchronous behaviour

## Event Sourcing - to track all the different operations
The fundamental idea behind Event Sourcing is to express the state of Aggregates as a linear sequence of events.
In ES there is not such operation on database as UPDATE TABLE..., **only INSERT** operations are supported.
To increase performance it's good to take snapshot of an aggregate state and replay only the events in the event stream
that occured after the snapshot was taken.

**Event sourced aggregate**
```php
interface EventSourcedAggregateRoot
{
    public static function reconstitute(EventStream $events);
}
```
```php
/** keep in mind that AggregateRoot class is the same like it was in CQRS */
class RealAggregateRoot extends AggregateRoot implements EventSourcedAggregateRoot
{
    public static function reconstitute(EventStram $history)
    {
        $aggregate = new static($history->getAggregateId());
        
        foreach ($events as $e) {
            $aggregate->applyThat($e);
        }
        
        return $aggregate;
    }
}
```

To create such `ConcreteAggregate` there must be created **adapter** for a repository in infrastructure layer.
Let's assume we already have `AggregateInterface`
```php
class EventStorePostRepository implements AggregateRepository
{
    private $eventStore, $projector;
    public function __construct($eventStore, $projector) { ... }
    
    /** it saves current state (in application layer) to the actual database (by $eventStore) */
    public function save(ConcreteAggregate $aggregate)
    {
        $events = $aggregate->recordedEvents();
        
        $this->eventStore->append(new EventStream($aggregate->id(), $events);
        $aggregate->clearEvents();
        
        $this->projector->project($events);
    }
    
    public function byId(AggregateId $id)
    {
        return RealAggregateRoot::reconstitute($this->eventStore->getEventsFor($id);
    }
}
``` 

Implementation of an event store - also infrastructure layer (on redis example)
```php
class EventStore
{
    private $redis, $serializer;
    public function __construct($redis, $serializer) { ... }
    
    public function append(EventStream $eventStream)
    {
        foreach ($eventsStream as $event) {
            $data = $this->serializer->serialize($event, 'json');
        }
        
        $date = (new DateTimeImmutable())->format('YmdHis');
        
        $this->redist->rpush(
            'events:' . $event->getAggregateId(),
            $this->serializer->serialize([
                'type' => get_class($event),
                'created_on' => $date,
                'data' => $data
            ], 'json')
        );
    }
    
    public function getEventsFor($id)
    {
        $serializedEvents = $this->redis-lrange('events:' . $id, 0, -1);
    
        $eventStream = [];
        $foreach($seriailzedEvents as $e) {
            $eventData = $this->serilizer->deserialize($serializedEvent, 'array', 'json');
            
            $eventStream[] = $this->serializer->deserialize($eventData['data'], $eventData['type'], 'json');
        }
        
        return new EventStream($id, $eventStream);
    }
}
```
Implementation of an snapshot repository
```php
class SnapshotRepository
{
    public function byId($id)
    {
        $key = 'snapshots:' . $id;
        $metadata = $this->serializer->unserialize($this->redis->get($key));
        
        if (null === $metadata) {
            return;
        }
        
        return new Snapshot(
            $metadata['version'],
            $this->serializer->unserialize(
                $metadata['snapshot']['data'],
                $metadata['snapshot'['type'],
                'json'
            )
        );
    }
    
    public function save($id, Snapshot $snapshot)
    {
        $key = 'snapshots:' . $id;
        $aggregate = $snapshot->aggregate();
        
        $snapshot = [
            'version' => $snapshot->version(),
            'snapshot' => [
                'type' => get_class($aggregate),
                'data' => $this->serializer->serialize($aggreagte, 'json')
            ]
        ];
        
        $this->redis->set($key, $snapshot);
    }
    
}
```
Implementation a snapshot strategy for given n events
```php
class EventStoreAggregateRepository implements AggregateRepository
{
    ...
    
    public function byId(AggregateId $id)
    {
        $snapshot = $this->snapshotRepository->byId($id);
        
        if (null === $snapshot) {
            return RealAggregateRoot::reconstitute($this->eventStore->getEventsFrom($id);
        }
        
        $aggregate = $snapshot->aggregate();
        $aggregate->replay($this->eventStore->fromVersion($id, $snapshot->version()));
        
        return $aggregate;
    }
    
    public function save(ConcreteAggregate $aggregate)
    {
        $id = $aggregate->id();
        $events = $aggregate->recordedEvents();
        $post->clearEvents();
        $this->eventStore->append(new EventStream($aggregate->id(), $events));
        $countOfEvents = $this->eventStore->countEventsFor($id);
        $version = $countOfEvents / 100;
        
        if (!$this->snapshotRepository->has($post->id(), $version) {
            $this->snapshotRepository->save($id, new Snapshot($post, $version));
        }
        
        $this->projector->project($events);
    }
}

```

#### Summarization and conclusion
- event sourcing is made from:
    - aggregate root that has a method to reconstitute aggregate state from given events
    - repository (that lays on infrastructure layer) within which injected is Projector and EventStore
    - event store
    - projector (which originates from cqrs)
    - snapshot (repository) - not necessary needed but they increase performance
- event sourcing properties:
    - you can store all aggregate changes within one table
    - each model state change is stored as **INSERTed** event on database
    - there is no need for ORM
- event store is made from two methods:
    - append(EventStream $es) - "**inserts**"\* events to the database
    - getEventsFor($id) - returns all
- snapshot repository:
    - its is a serialized version of an aggregate at given time or n events occured
    - you can store snapshots in the same database
- in my opinion there is no need for relational database in this approach, moreover relation database would be
pain in the ass
- i wonder how to handle "database migrations" - if there is business need for another event type and this code goes
to production you cannot "undo" code to previous version because that code might not be able to handle already stored
events in database. So you need to "undo" the database itself and loose all the data. Am I correct? 

---
\* - *inserts* - is in quotes because it doesn't need to be relational database, but i wanted to point out that
in this apporach you cannot do other operation on database other than add or create new record or document
