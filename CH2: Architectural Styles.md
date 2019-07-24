# CHAPTER 2: ARCHITECTURE STYLES

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

## Hexagonal Architecture (ports and adapters) - exposes requirements such as blogs, static pages, and so on
  
## Layred architecture
From top to bottom (higher abstraction to lower)
- USER INTERFACE LAYER - orchestrates application & domain layer
- APPLICATION LAYER - encapsulates domain and data access
- DOMAIN LAYER - encapsulates domain and data access
- INFRASTRUCTURE LAYER - encapsulates infrastructure concerns

    Higher layer (for ex. interface layer) can always bypass lower layer or layers (for ex. app layer
    or/and domain layer) but NEVER lower layer can call higher layer!
- Model-View-Controller

## Projection 

## Dependency Inversion Principle (DIP)
High-level modules hould not depend on low-level modules.
Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.
