# Guidelines

In an ever-evolving technological landscape, it is crucial to establish clear principles and standards to ensure consistency, scalability, and maintainability. This document serves as a foundation for best practices, guiding decision-making and fostering a culture of excellence in software development.

## General

We follow a modular approach based on event-driven architecture and microservices, enabling scalability, flexibility, and efficient communication between system components. Additionally, we adhere to the principles of Trunk-Based Development, ensuring that the `main` branch always contains production-ready code.

## Versioning

We use **asdf** to manage versions of programming languages. Each repository needs a `.tool_version` in his root to define which versions of language and platform is used.

## Programming languages

We use following Programming Languages:

- Python
- TypeScript

Every programming language **must** use a package manager. The execution of standard tasks **should** be named unified. For example for Python `make test` and for TypeScript `npm run test`.

- **test** for executing all tests
- **lint** for style check the code
- **typecheck** for type checking the code
- **format** for formatting the code
- **e2e** for executing e2e tests

### Python

We use **Python** for backend services. Code in Python **must** be typed. For task automation a `Makefile` **must** be in the root of the repositorie. If possible used packages **should** be configured in a `pyproject.toml` in the root folder. We use following standard packages:

- **poetry** as package manager
- **flask** as web application framework
- **SQLAlchemy** as ORM
- **pika** as AMQP client
- **ruff** as style check
- **ruff** as formatter
- **mypy** for static type checking
  - mypy **must** be runned in strict mode
- **pytest** for testing Python code
- **factory_boy** for factories of models and value objects in tests
- **python-dotenv** for loading local environments


### TypeScript

We use TypeScript for frontend services. We use following standard packages:

- **npm** as package manager
- **jest** for testing TypeScript code
- **openapi-typescript** for client generation
- **next.js** for our frontends
- **eslint** for linting
- **prettier** for formatting
- **playwright** for e2e tests
- **@tanstack/react-query** for fetching, caching and updating asynchronous data in client components
- **@mui/material-nextjs** for components
- **tailwind** for styling
- **react-final-form** for Forms


## Web Services

Our HTTP APIs are designed and documented using **OpenAPI** to ensure clarity, consistency, and ease of integration. The root element of the OpenAPI specification is placed in the backend service at `openapi/openapi.yaml`.

### Routes

- the path **must** be in kebap-case
- query parameters **must** be in camel-case
- JSON payload
  - keys **must** be in camel-case
  - dates and times **must** be encoded as ISO8601
  - Content-Type **must** be `application\json`

## Asynchronous Messaging

We utilize **RabbitMQ** as a message broker to enable reliable, asynchronous communication between services. Connections to RabbitMQ **must** have a name.

### Publish

- For each backend service **must** exist a topic-exchange with the name of of the backend service.
- For the creation, update and deletion of a resource each backend service **must** send an event on the backend service exchange.
- Each event **must** contain a routing key
  - the routing key **must** start with the resource following by the id and end with the action.
  example: `book.2984.delete`
  - the routing key **should** get extended with informations. like the value which has changed
  example: `book.2984.title.update`
- The publish of an event **must** be transactional. This can achived for example with the [transactional outbox pattern](https://microservices.io/patterns/data/transactional-outbox.html) and [polling publisher](https://microservices.io/patterns/data/polling-publisher.html).

### Consume

- Queues **must** start with the name of the consuming service. Example: `book-store.updates`
  - Binding routing keys for actions **must** use the `#` wildcard. Example: `book.#.update`
  - If an event fails to be processed, it **must** be sent to a retry queue. The retry queue **should** implement an exponential backoff strategy to handle retries.

## Project structure

We **should** follow a **feature-based folder structure**, where each feature is self-contained and encapsulates its logic, including controllers, services, and models. This approach improves modularity, scalability, and maintainability. The code **must** be placed in a `src`folder. If one feature requires access to another in the same Microservice, it should do so through direct imports rather than network-based communication. A common `shared/` module should be used for utilities, database connections, and other cross-cutting concerns. 

Example:

TODO

## Deployment

TODO

-GitOps, ArgoCD, Kubernetes, GCloud etc.


## Database

We **must** use snake case for naming all database entities, including databases, tables, columns and constraints. Tables **must** be named in the plural form of the resource they store.

## Monitoring

Monitoring is essential for understanding the health and performance of our systems. Without monitoring, we are blind to failures and can only react when users complain. With monitoring, we are proactive. We use **Sentry** for error tracking and **Grafana** for metrics and dashboards. Together, they give us visibility into the four golden signals:

1. **Latency**  
   - How long it takes to serve a request.  
   - Includes both *successful* and *failed* requests (since errors can be very fast).  

2. **Traffic**  
   - The demand on the system.  
   - Measured as requests per second, queries per second, transactions per second, etc.  

3. **Errors**  
   - The rate of failed requests.  
   - Includes explicit failures (e.g., 5xx responses) and implicit ones (e.g., incorrect results).  

4. **Saturation**  
   - How “full” the system is.  
   - Indicates how close the system is to its limits (CPU, memory, queue length, thread pools, etc.).

## Tracking

Tracking is necessary to record User usage and behaviour. Tracking **should** not impact the performance of production code. Any tracking logic should be designed to run asynchronously and should not introduce latency or block critical application flows.

- Dependencies required for tracking **must** not be included in the main production bundle unless absolutely necessary.
- Use environment-based configuration to enable or disable tracking in different environments.
- If tracking requires external services, ensure that failures do not affect the core functionality of the application.
- Consider using an event-driven approach to decouple tracking from business logic.
