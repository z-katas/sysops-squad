## ADR - RabbitMQ for handling events.

### Status
Proposed

### Context
In order to reliably handle the events related to tickets or notifications, concepts such as message broker (RabbitMQ) and distributed streaming platform (Kafka) were considered for evaluation.

### Decision
Message broker was chosen because the system does not have telemetry kind of events** to go for a distributed streaming platform. Also, the load on the system is the range of 100s or 1000s of users per day, creating few tickets each, which RabbitMQ can easily handle.

Also, RabbitMQ has very good fault tolerance mechanisms such as delivery retries and dead-letter exchanges, which would ensure that the ticket is processed or moved to a place where manual intervention is needed.

### Consequences
Messaging bus (RabbitMQ) can not replay the events, once consumed. This should be fine for the system because the ticket service stores ticket history.
