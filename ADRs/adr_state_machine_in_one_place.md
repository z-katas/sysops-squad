
## ADR - Have the ticket state machine in one place - ss.ticket.TicketServiceImpl.

### Status
Proposed

### Context
The ticket goes through various state changes during its lifecycle. Each state change may trigger different behaviours:
- Change from DRAFT to CREATED triggers an event to assign relevant Expert for the task.
- Change from CREATED to ASSIGNED triggers an event to notify the assigned Expert about the task.
- Change from INPROGRESS to COMPLETED triggers an event to send relevant the survey to the Customer email.

See diagram for the state changes allowed.
  
### Decision
Having the state machine behaviour spread in multiple places or components may cause confusion and may lead to bugs. It is better to have it in one place - **ss.ticket.TicketServiceImpl**. Any service or component trying to change the status of the ticket must go through this class.

### Consequences
Developers must be careful not to add any code after the procedure call to update the ticket status in the component services. If any code is added, any failure in the code after the call to update the ticket state would cause the actor to fail and the ticket would be retried (by RabbitMQ).


