# CHAPTER 11: APPLICATION
Application layer separates Domain Moder from the clients that query or change it's state. It connects all external
points to the domain, like: html forms, API clients, command line, frameworks, UI, etc.

## Application Service - by example
In order to register a new user following steps must be done:
- get email/login from the client
- check if the email is already in use
- create a new user
- add this new user to the existing user set
- return the user we've just created

### Requests / DTO
```php
namespace Application;

class SignUpUserRequest
{
    private $email, $password;
    public function __construct($email, $password) { ... }
    public function email() { return $this->email; }
    public function password() { erturn $this->password(); }
}
```
**Build your DTO / Request in your controller**. It might come form the request or form, doesn't matter.

### Application Service
```php
namespace Applciation;

class SignUpUserService
{
    private $userRepo;
    public function __construct(UserRepository $ur) { ... }
    
    public function execute(SignUpUserRequest $request)
    {
        $email = $request->email();
        $passwd = $request->password();
        
        $user = $this->userRepository->ofEmail($email); // LOOK! it returns Aggregate,
                                            // I guess Domain Serive should be injected like repo, but dunno..
        
        if (!$user) {
            throw new UserAlreadyExistsException();
        }
        
        $this->userRepo->add(new User($this->userRepo->nextIdentity(), $email, $passwd));
    }
}
```

#### Summarization and conclusion
- DTO / Requests:
    - build DTO / Requests within controller (from request or Form)
    - to create DTO use **primitives** - don't complicate data design
        - that gives you easy way to serialize and deserialize object event to log it or message away
    - **do not** put logic inside
    - DTO is an **data structures** not **object** - **DON'T TEST THEM**
- Application Service:
    - keep **application service thin**, use it only to **coordinate** task on the model
    - one class per each scenario, but if you wish you could use "common" service desing like UserRepository and create
        more that one method
    - return data to controller as DTO - **never domain**!
    - you can use external class called asembler to convert DTO into domain and vice versa (therefore
    app service would not be responsible for that) - code reusage
    - data transformers ??? 
- **Domain Events MUST BE** configured before any Application Service fires
- you can execute application service by a command bus

#### NOTES
Command buses and message systems are used outside the domain model!!! Domain model only published events
but all the rest is handled in app layer or infrastructure layer!
