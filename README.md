## Introduction
Since tickets form the crux of the Sysops Squad business, ticket related problems in the current system should be prioritised in order to continue business. Those problems are:

- Customers are complaining that consultants are never showing up due to lost tickets.
- Oftentimes the wrong consultant shows up to fix something they know nothing about.
- Customers and call-center staff have been complaining that the system is not always available for web-based or call-based problem ticket entry.

Also the following need to be addressed as part of or soon after solving the above problems:

- Change is difficult and risky in this large monolith. Whenever a change is made, it takes too long and something else usually breaks.
- Due to reliability issues, the monolithic system frequently “freezes up” or crashes.

Diagram
## Assumptions
- The existing application is a single process monolithic system.
- In the architecture components diagram given in the presentation, the bold bordered components exist as modules in a repository (say github).
- It is a Java based spring application.
- The monolith exposes REST API endpoints to the outside world.
- There are materialised views in the database for the various reports.
- The current monolithic system has the following asynchronous operations:
  - Assigning an Expert to a ticket, post its creation.
  - Notifying an Expert when a ticket is assigned.
  - Notifying a Customer after an Expert is assigned.
  - Sending a survey to a Customer, post ticket completion.
  - Sending billing or offers related notifications to Customers.
- Ticket Shared component has business logic to create,update (change state) and fetch tickets in the system.
- Ticket Maintenance component has simple APIs to fetch, create and update tickets from the database. Only Ticket Shared component accesses Ticket Maintenance component.
- There are interfaces for all the behaviours exposed by the components. This would enable swapping implementations in case of roll backs during migration.
## Solution
### Step 1
Investigate the current system to find the root cause of the problems.

- Check the logs for errors or stack traces. See what is causing the tickets to go missing. Some of the speculations that can be made are:
  - A ticket may be in limbo due to a faulty state machine when the system is unable to find an Expert for the job at the time of processing (first run).
  - There might not be a reliable queue due to which the async events may be getting lost when the system freezes.
  - Memory overflow issues, especially during peak loads.
- Check if the logic for assigning an expert



### Step 2
Swap the implementation of the current asynchronous blackbox delivery system from Ticket Shared to Ticket Assigned, Ticket Route, Ticket Completion with a reliable messaging service implementation, like RabbitMQ, if not already present. Check *ADR - RabbitMQ for handling events.*

- Introduce a publisher class to publish ticket events to RabbitMQ.
- Use the Actor pattern to listen to ticket events and pass it on to the corresponding implementations.
  - Limit the no. of threads the actor can spawn and use in the RabbitMQ consumer configuration.
- Use feature flags or environmental variables to conditionally switch between old and new implementations in case of rollback.
- Deploy and test the change in dev environment.
- Deploy the change to production.
  - Once the deployment is complete, sanity test the environment.
  - Monitor the system.
  - Roll back if there are any failures or performance issues.
    - Since we are using a reliable queue, any failures in message delivery can be handled independently after the rollback.

Diagram

Similar
### Step 3
Identify the services in the final refactored architecture. Diagram

### Step 4
Identify the architecture pattern. Diagram

### Step 5
Prioritise the components that need to be migrated from the existing system. Diagram

### Step 6
Migrate Ticket related components from monolith to Ticket Service. Use *Strangler Fig* pattern with shallow extractions to extract out Ticket Assign, Ticket Route and Ticket Completion components. Here’s how to extract Ticket Assign

- Create an empty Ticket Service microservice.
  - The ticket related tables would continue to live in the monolith for sometime.
- Copy over the code from **ss.ticket.assign** namespace to the new microservice.
  - Check *ADR - Extract code first for asynchronous ticket processes.*
- In the new microservice, modify the implementation of **TicketAssignImpl** class to call corresponding REST controllers in the monolith to fetch expert data and update ticket status.
- Have a feature flag to not initialise TickerAssignActor in the monolith. Leave the rest of the **ss.ticket.assign** namespace in the monolith as is, in case of a roll back.
- Deploy the change to production.
  - Once the deployment is complete, sanity test the environment.
  - Monitor the system.
  - Roll back if there are any failures or performance issues.
    - Since we are using a reliable queue, any failures in message delivery can be handled independently after the rollback.
- If the migration goes as expected, remove the code from **ss.ticket.assign** namespace  in the monolith.

Diagram

Use the same technique to extract out Ticket Route and Ticket Completion components.

### Step 7
Move the foreign key relationships on the ticket tables from database layer to code layer. Currently,
###
### Step 7
Migrate Reports related components from monolith to Reports Service

### Step 8
Migrate Knowledge base related components from monolith to Knowledgebase Service
### Step 9
Migrate Survey related components from monolith to Survey Service

### Step 8
Split User Service
## Other Suggestions
- NO relation between survey and ticket
- No relation between contract and expertise

## Note
Repository layer in the service has been omitted in many diagrams for brevity.
## ADRs




**ADR 1**

Products can be part of CMS, as these are updated not often. Ticket service may call CMS to verify product id when a ticket is being created.

LImitation - This adds to the coupling between TIcket service and CMS. But given that the load on the service is only 100s or 1000s of users per day, this might be okay to have.

**ADR 2**

Notification component currently co-exists with Login Component. Ideally notification service may exist independently, catering to billing notifications for customers and general info for now. (for now and internal users later on). But the notifications may exist independently as its scale and purpose are different from the rest of the. Notification service may also maintain customer’s/expert’s ticket related notifications, which are currently sent via email/SMS, for tracking (to avoid repudiation).

Going with RabbitMQ because Kafka might be an overkill


References

<https://betterprogramming.pub/rabbitmq-vs-kafka-1779b5b70c41>

