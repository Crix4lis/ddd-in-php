# CHAPTER 12: INTEGRATING BOUNDED CONTEXTS

## Integration Relationships

1. Unidirectional relationship (client-customer)<br>
One acts as provider (upstream), the other as client of it (downstream). It ends up with *Customer-Supplier
Development Teams*.
1. Separate Ways<br>
Decrease a BOUNDED CONTEXT to have no connections to the others at all (it will create legacy system that in future
will be suppressed)
1. Conformist - no close relationship<br>
Third party supplier of a system will not participate therefore Creating Bounded Context in this model is focused
to acceptance and conform to **their** domain model.

## Implementing Bounded Context Integrations
>it's assumed we have Customer-Supplier relationship

## Notes
- on message subscribes run commands to the domain using CommandBus
