# CHAPTER 5: SERVICES
When there are operations that need to be represented, we can consider them to be services. There are three
different types of services which are as follows:
- Application Service:
    - operate on **scalar types** (any type that is unknow to the domain model and primitive types)
    - they **do not contain any business rules nor domain logic** They exist to coordinate, orchestrate and
    execute operations that belong to the domain model
- Domain Service:
    - Operate **only** on types belonging to the domain
    - they contain meaningful concepts that can be founc within the domains ubiquitous language (methods
    that have business meaning)
- Infrastructure:
    - Operations that fulfil infrastructure concerns, such as sending emails, logging meaningful data

## Application service
They are middleware between the outside world and the domain logic. Its purpose is to transform commands from
outside world into meaningful domain instructions

### Symfony application
1. Symfony controller (UI) - transforms request data into DTO / Command
2. Application service contains (injected aggregates / entities, or domain services? - don't know yet),
receives know structure (DTO or Command) and orchestrates that entity / aggregate / domain service (?)

Might as well orchestrate cross aggregate / entity transaction [p 105]

## Domain service
"Throughout conversations with domain experts, you will come across concepts in the Ubiquitous Language that cannot
be neatly represented as either an Entity or Value"
> page: 106

For example User (also User class) cannot sign in to the system because that user does not yet exists.
Therefore there needs to be created business path that allows for that and here is where domain service fits in.

Another example is that **cart domain cannot create order** from itself (it cannot promote itself to be an order).<br>
Example:
```php
class CreateOrderFromCart
{
    public function execute(Cart $cart) { ... }
}
```

## How to Avoid Anemic Domain Model!?
"The way to avoid falling into an anemic domain model is to instead when starting a new project or feature, to think
of the behaviour first. Databases, ORMs, and so on are just implementation details,and we should strive to push
the decision to use these tools as late in the development process as we can. In doing this we can focus on the
one true attribute that matters, the behaviour."
> page 121

#### Summarization and conclusion
- APPLICATION SERVICE
    - uses only scalar types (unknown domain types, and primitive types)
    - do not contain business logic or meaningful domain language
    - transform outside commands to domain instructions
    - **orchestrates database transactions** - page 105
- DOMAIN SERVICE
    - if a case does not belong to aggregate / entity and to the application layer (service)
    - domain services are **stateless operations**
    - **use public execute method** for domain service operation- this ensures statelessness
    - **DON'T ABUSE DOMAIN SERVICES**! If you do your entites might become anemic

- DTO can be treated as a domain command

#### NOTES
I think that DDD Policy is a class injected to the ddd service (implements Strategy Pattern) and it comes along
with different implementations of the domain service itself - each specific domain service implementation can
have injected different strategy for doing some stuff (like encrypting a password is a strategy, but different kinds
of domain service that creates user with different kind of password encryption is separate implementation of a 
domain service itself)
