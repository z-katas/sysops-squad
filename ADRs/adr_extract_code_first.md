## ADR - Extract code first for asynchronous ticket processes.

### Status
Proposed

### Context
In the process of migrating the monolith, asynchronous ticket processes, like assign, route and completion, were identified as the initial components to be migrated to reap the business benefit along with ease of decomposition. It was debated whether to extract code first or db first for ticket service.

### Decision
Because there is an urgent need to salvage the platform and not lose customers, it is decided to move the code first to an independent ticket service and make that service depend on the monolith for fetching or updating the ticket data. If data were to be extracted/separated first, there would not be any immediate business benefit.

### Consequences
Ticket service would not be a pure microservice yet. The data which it works on in the bounded context is not within its control. Care must be taken to move the ticket related tables to the ticket service immediately after moving the asynchronous components.
