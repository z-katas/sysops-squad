
## ADR - Have products as part of User Service

### Status
Proposed

### Context
A products table (with expertise) would have to introduced to help assign a ticket to the right Expert.
  
### Decision
Products can be part of Customer service, as these are not updated often and they can be added to a contract when the customer subscribes to a plan. 

### Consequences
Ticket service must call Customer Service to verify product_id when a ticket is being created.
This adds to the coupling between Ticket service and Customer Service. But, given that the load on the service is only 100s or 1000s of users per day, this might be okay to have.

Also, Customer Service would have to call User Service to validate the expertise when a new product is created. This adds to the coupling between User service and Customer Service. But, this is fine as products are not created that often.
