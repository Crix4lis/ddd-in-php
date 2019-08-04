# CHAPTER 8: AGGREGATES
Aggregate is a **cluster of domain objects** that can be treated as a single unit!
Any references from outside the aggregate should only go to the aggregate root

## Design aggregates - by example
To model aggregates there needs to be found **INVARIANT**. It's a rule that is **always** true no matter
what new **requirements** there will be. For example **Order** and **OrderItem**. If there is an order placed for one
new item (pencil) total amount of ordered items in single **Order** is 1. If then client will order two rubbers,
there total amount of ordered items (belonging to order itself, not order item) will be 3. If user will want to reduce
total number of items he will must reduce items from one of the items ordered, therefore there is
"two way relationship", which for me is an invariant. That invariant (relationship) ensures that whole model is 
**consistent**.

### NO INVARIANT therefore TWO SEPARATE AGGREGATES
Let there be web application that makes wishes. A **user** can make any **wish**. Let's consider one use case:
`As a user I want to make a wish`.

if you model your domain ang aggregate correct way, within a application service code should look like this:
```php
class MakeWishApplicationService //should be without 'application'
...
if (null === $user) {
    throw new UserDoesNotExistException();
}
$wish = $user->makeWish($this->wishRepository->nextId(), $address, $content);
$this->wishRepository->add($wish);
```

**Note that `$user->makeWish()` returns Wish object** (that's because they are not connected within single aggregate)<br>
User's aggregate:
```php
...
public function makeWish(WishId $wishId, $address, $content)
{
    return new Wish($wishId, $this->id(), $address, $content);
}
```

#### Sum up
Wish and User **are linked** with each other by UserId **but** they **do not conform an aggregate** because there is
no invariant between them

### Another business rule
We want new users to have a maximum of three wishes available, and if user want to have more, he needs to pay for
premium.
 ```php
 class User {
 ...
    public function makeWish(WishId $wishId, $address, $content)
    {
        $this->wishes[] = new Wish($wishId, $this->id(), $address, $content); //it ensures db consistency, but I guess 
                                                     //it creates aggregate.. dunno because chapter eneded unfinished..
    }
 }
 ```

ALL CHAPTERS ARE `TBW` SO NOT MANY IFNO.. =( Too bad, I needed that chapter.. ==((

#### Summarization and conclusion
- aggregate properties:
    - aggregate is a **cluster of domain objects** that can be treated as a single unit
    - aggregate will have **one of its component objects** be **the aggregate root**
    - any references from outside the aggregate should only go to the aggregate root
    - the **root ensures integrity** of the aggregate as a whole (aggregate is a **group of objects** that must 
    be **consistent together**)
    - **db transactions should not cross aggregate boundaries**
    - **modify one aggregate** per transaction - **exception is UX case**
- designing aggregate:
    - find model **invariant** - "two way relationship" that is always true, and ensures model consistency

#### Notes
- if entity within aggregate can live without it's root, must it not belong to the root?
aggregate of entities (and vo) must contain "relations" to different entities only if they have no meaning
without aggregate root? - that's my question, not statement
- read about invariants in context of aggregates (it's a rule that is always true, before and after some operation)
- "Factories help us keeping the business invariants" [page 177] - read more about factories
- if there are two separate aggregates, one aggregate can return new another aggregate??? Is it because they
are not modeled as single aggregate, but one aggregate's rule (method) creates new separate object? What's more, 
within one application service those aggregates are managed together [page 178] - I wonder if it's bad design example
of it is correct and true. If so, it means there can be many aggregate roots within single domain, and they can be
managed together, but still they do not share **invariant** therefore there is no need for a db transaction.
- read about database mechanism pessimistic and optimistic concurrency control