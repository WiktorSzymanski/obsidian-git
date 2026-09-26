# Event Driven Architecture
---
**Event-driven architecture** (**EDA**) is a software design pattern that allows systems to detect, process, manage, and react to real time events as they happen. (cite https://www.confluent.io/learn/event-driven-architecture/#how-it-works). Once an event occurred it's being published so all components of the system that listen for that event, no matter where they are can access that event and continue work flow. This creates loose coupling between components inside the application and eliminates the rigidity of traditional dependency injection, (cite https://dev.to/dariomannu/the-design-pattern-making-dependency-injection-obsolete-2n8b) or even allows services to be fully separated, making the system more resilient, scalable and independent while deploying. (cite https://www.ijsat.org/papers/2025/1/2907.pdf).

Event it's an information that something important happened inside the system. Events are fundamental data structures that record any occurrence or change in the system or environment (cite https://www.ibm.com/think/topics/event-driven-architecture), that another part of the system may find important for it's own reasons.

An event has the following characteristics:
- It is a record of something that has happened.
- It captures an immutable fact that cannot be changed or deleted.
- It occurs whether or not a service applies any logic upon consuming it.
- It can be persisted indefinitely, at a large scale, and consumed as many times as necessary.

Event-driven architectures have three key components: event producers, event routers, and event consumers. A producer publishes an event to the router, which filters and pushes the events to consumers. Producer services and consumer services are decoupled, which allows them to be scaled, updated, and deployed independently. (cite https://aws.amazon.com/event-driven-architecture/)
In other words some action may produce an event that is to be published, for example via EventBus. Any part of the system that awaits an action to happen, subsribes for specific events so once they happen they can continue the workflow. An action inside the system may produce one or even many events.

EDA promotes asynchronous communication patterns that fundamentally alter how components interact. Unlike traditional request-response models where systems communicate synchronously, EDA enables what's called "temporal decoupling," where system components can operate independently without requiring the simultaneous availability of their communication partners.(cite https://www.ijsat.org/papers/2025/1/2907.pdf).

EDA enables real-time operations, seamless data synchronization, and enhanced customer experiences through decoupled and resilient design. Analysis of 83 enterprises across financial services, e-commerce, and healthcare sectors found that teams working within event-driven ecosystems completed feature development cycles 37% faster than counterparts using traditional request-response patterns.(cite https://www.ijsat.org/papers/2025/1/2907.pdf).

EDA allows system to response fast for any event that occur in asynchronous manner with no need for continous pooling. This makes it more responsive and less resource hungry (cite solace.com/blog/event-driven-architecture-pros-and-cons/)

Most significant advantages of EDA are:
 - Loose Coupling and Scalability: eda promotes loose copuling between components, since now they can interact via asynchronous event messages, enabling them to be developed, deployed, and scaled independently. This loose coupling allows for better modularity, flexibility, and agility in the system. New components can be added or modified without affecting the existing components, facilitating scalability and accommodating changing business requirements. (cite https://www.confluent.io/learn/event-driven-architecture/#loose-coupling-and-scalability)
 -  Real-time processing : Event-driven systems allow for push-based messaging and clients can receive updates without needing to continuously poll remote services for state changes. (cite https://cloud.google.com/eventarc/docs/event-driven-architectures)
 - Reliability and Fault Tolerance: In an event-driven system, events are generated asynchronously, and can be issued as they happen without waiting for a response. Loosely coupled components means that if one service fails, the others are unaffected. If necessary, you can log events so that the receiving service can resume from the point of failure, or replay past events. (cite https://cloud.google.com/eventarc/docs/event-driven-architectures)
   
Event-driven architecture is widely used across various industries and use cases. In e-commerce applications, when a customer places an order, an event is triggered to initiate inventory management, payment processing, and shipping coordination. Netflix's implementation serves as a prominent example, where their event-driven microservices ecosystem processes approximately 6.5 trillion events daily through a sophisticated event mesh topology. (cite https://www.ijsat.org/papers/2025/1/2907.pdf)

While implementing EDA delivering events is a key component, for that we can use any from below:
Popular event broker technologies include Apache Kafka, designed for high-throughput, fault-tolerant, publish-subscribe messaging; RabbitMQ, which implements multiple messaging protocols with robust routing capabilities; and ActiveMQ, an enterprise-grade message broker supporting various cross-language clients (cite https://www.ijsat.org/papers/2025/1/2907.pdf).