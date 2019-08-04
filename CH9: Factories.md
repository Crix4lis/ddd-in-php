# CHAPTER 9: FACTORIES
"In the Domain, factories help in decoupling the client from knowing how to build complex objects and Aggregates.
You could use them in order to create entire Aggregates as an entire piece, enforcing their invariants." [page 183]

## Factory method pattern
`public function makeWish(WishId $wishId, $email, $content)` - method inside User aggregate from previous chapter
is an factory method

### Forcing Invariants
Enforcing business rules that allow or disallow creation of an aggregate.

Forum's aggregate:
```php
class Forum
{
    ...
    public function publishPost(PostId $postId, $content)
    {
        if ($this->isClosed()) {
            throw new ForumClosedException(); //invariant - business rule that you cannot publish post if formu is closed
        }
        
        $post = new Post($this->id, $postId, $content);
        
        DomainEventPublisher::instance()->publish(new PostPublished($postId));
        
        return $post; //I guess it returns Post because Forum is within Post aggregate
    }
}
```

## Abstract Factory pattern
It allows to create aggregates if creation should be handled as **specification**
(might have different variants and implementations)

```php
namespace Domain;

interface PostSpecificattionFactory
{
    public function createLatestPost(\DateTime $since);
{
```

Now let's just create factory for different Repository types, for example `InMemoryPostSpecificationFactory`:
```php
class InMemoryPostSpecificationFactory implements PostSpecificationFactory
{
    public function createLatestPost(\DateTIme $since)
    {
        return new InMemoryLatestPostSpecification($since); // in memory impelmentation of Post?
    }
}
```

Client's (service) code:
```php
...
$post = $this->postRepostiory->query($this->postSpecificationFactory->createLatestPost($request->since));
```


## Object Mother pattern
It is used to create example data for the test - creates them with usage of static methods, like `createOne()`
```php
class AuthorObjectMother
{
    public static function createOne()
    {
        return new Author(new Username('johndoe'), new FullName('John','Doe'), new Email('john@doe.com'));
    }
}
```

## Data Builder
It is used for testing to create example object with different variants of set data
```php
class AuthorBuilder
{
    private $username, $email, $fullName; //those objects are VO, not primitive values

    private function __construct()
    {   
        //default values
        $this->username = new Username('johndoe');
        $this->email = new Email('john@doe.com');
        $this->fullName = new FullName('John','Doe');
    }

    public static function anAuthor()
    {
        return new self();
    }
    
    public function withUsername(UserName $username) {
        $this->username = $aUsername;
        
        return $this;
    }
    
    public function build()
    {
        return new Author($this->username, $this->fullName, $this->email);
    }
    //rest methods
}
```

#### Summarization and conclusion
 - default method to create an aggregate is an factory method
 - more complex creation of aggregate should be done via abstract factory
 - to ease aggregate tests create Object Mother pattern or Data Builder pattern

#### Notes
- I wondered if `query method` can return something, according to
[stackoverflow](https://stackoverflow.com/questions/43433318/cqrs-command-return-values) post it can:
    - execution result: success or failure
    - error messages or validation errors (in case of failure)
    - the aggregate's new version number (in case of success)
