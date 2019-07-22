# HOW TO FIND UBIQUITOUS LANGUAGE
- identify key business processes, their inputs and their outputs
- create a glossary of terms and definitions
- capture importand software concepts with some kind of documentation
- share and expand upon the collected knowledge with the rest of the team (Devs and Domain experts)
> The most common technique for building the Ubiquitous Language is *Event Storming*

# WHEN (NOT) TO USE DDD
- DON'T USE:
  - if application is data-centric
  - if use-cases mainly manipulate rows in a database
  - if use-cases mainly perform pure CRUD (create, read, update, delete) operations
  - if application is not data-centric but has less than 30 use-cases (and is not likely to grow) - it might
  be easier to base this custom, yet small logic on framework flow and its tools
  
- DO USE:
  - if application use-cases are likely to grow over 30 use-cases
  - if application is likely to grow in complexity
  - if application is likely to change
  - if application is not well understood and it's hard to get to know by new developers 
  it's likely you should try to refactor it towards DDD models

# PROBLEM SPACE VS SOLUTION SPACE
## Problem space
DDD uses **Domains** and **Subdomains** to group and organize what company wants to solve. It means that
the problem space is a place where you model you organisation specific business logic.

## Solution space
DDD provides two patterns: **Bounded Contexts** and **Context Maps**. The goal is to define how 
to provide an implementation to all the identified Subdomains by defining their interactions and
the details of those interactions.

## Problem space AND solution space
- Each **subdomain** should be solved with a **bounded context** *implementation*
- **Context map** should show how each **bounded context** is **RELATED** to the rest. It should 
show type of relation two bounded contexts have (for example: customer-supplier, patterns)

The ideal approach is to have each **subdomain** impelmented by one **bounded context** **BUT THAT IS NOT
ALWAYS POSSIBLE**.

> NOTE: If the domain is not too complex, applying the strategical part of DDD can add unnecessary overhead
and slow down your development speed

# MICROSERVICES
## CHARACTERIZATION OF A MICRO-SERVICE
1. Each service is and **autonomus** web app and is able to run it's internal logic without calling out 
to external services.
1. Each service **is owned by one team**.
1. Communication with other services or 3rd party system **is asynchronus wherever possible**. Especially
internal connection (micro-service to micro-service) **must not be** synchronus!
1. It all should have its own UI as well as API (for me this need is dumb as fuck! Y do you need UI when you dont?)
1. Each service must include **both data and logic**.

# NOTES
This book suggests that DDD approach results in distributed architecture (micro-services) but as far
as I am concerned it is possible to create so-called *modular monolithic* application. The book also
suggests that each *bounded context* might be independent micro-service, which is not true as far as I know.
It's not as easy as it seems that application can be split into multiple micro-services like 1:1
(bounded context : micro-service). I've heard that this is not correct. And a **subdomain** can be
powered by more than one micro-service and also the other way around - few **subdomains** can be 
run as single micro-service or whole application itself. In my opinion it is good to avoid micro-services
unless they are not truly needed to solve efficiency problems. To separate code, one should focus on
well designed classes by Design Patterns and tactical DDD patterns.

According to MICROSERVICES part of this file I am right that single context should not be single 
micro-service and introducing microservices too early is just a reason to fail with the project!

> books:<br>
> [Applying Domain Driven Design Patterns by Jimmy Nilsson](https://www.amazon.com/Applying-Domain-Driven-Design-Patterns-Examples/dp/0321268202)<br>
> [Introducing Eventstorming by Alberto Brandolini](https://leanpub.com/introducing_eventstorming)<br>
