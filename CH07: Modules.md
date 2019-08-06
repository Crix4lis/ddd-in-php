# CHAPTER 7: MODULES
"When you place some classes together in a Module, you are telling the next developer who looks at your design to
think about them together. If your model is telling a story,the Modules are chapters"
> Eric Evans in Domain-Driven Design

## Structuring code
```
|--src<br>
|   |---Application / Organisatin name
|                      |
|                      |-------- Domain Name
|                                       |
|                                       |------ Domain Model
|                                       |          |
|                                       |          |------ Model Name
|                                       |
|                                       |------ Infrastructure
|                                                  |
|                                                  |------ Logging
|                                                  |
|                                                  |------ Messaging
|                                                  |
|                                                  |------ Persistence
|                                                               |
|                                                               |--- Redis
|                                                               |
|                                                               |--- In Memory
|                                                               |
|                                                               |--- Doctrine
```
> page [159]
