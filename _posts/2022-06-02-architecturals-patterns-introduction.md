---
title: 'Introduction to Architectural patterns'
date: 2022-06-02
permalink: /posts/2022/06/architectural-patterns/
tags:
  - architectural pattern
  - SOA
  - micro-services
  - EDA
  - Serverless
---

There are various architectural patterns in software development, each with its own characteristics and purposes. Here are a few common architectural types :

1. Monolithic Architecture:
   - Monolithic architecture is a traditional approach where an entire application is built as a single, self-contained unit.
   - All the components and modules of the application are tightly coupled and deployed together.
   - It is relatively simple to develop and deploy but can become complex and difficult to scale and maintain as the application grows.

2. Client-Server Architecture:
   - In client-server architecture, the application is divided into two primary components: client and server.
   - The client sends requests to the server, which processes them and returns the response.
   - This architecture allows for a clear separation of concerns and enables multiple clients to interact with a centralized server.

3. Service-Oriented Architecture (SOA):
   - SOA is an architectural style where applications are composed of services that communicate with each other via well-defined interfaces.
   - Services are loosely coupled and can be developed and deployed independently.
   - SOA promotes reusability, flexibility, and scalability by breaking down complex systems into smaller, manageable services.

4. Microservices Architecture:
   - Microservices architecture is an extension of SOA where an application is decomposed into a collection of small, independent services.
   - Each microservice focuses on a specific business capability and communicates with other microservices via APIs.
   - Microservices are developed, deployed, and scaled independently, allowing for flexibility, scalability, and easier maintenance of complex systems.

5. Event-Driven Architecture (EDA):
   - EDA is an architectural pattern based on the concept of events.
   - Applications in EDA communicate through the exchange of events, which represent significant changes or occurrences in the system.
   - It enables loose coupling, scalability, and asynchronous processing of events, making it suitable for real-time systems and event-based workflows.

6. Serverless Architecture:
   - Serverless architecture, also known as Function as a Service (FaaS), allows developers to focus on writing functions without managing infrastructure.
   - Applications are built as a collection of stateless functions that are triggered by events or requests.
   - Serverless architecture provides auto-scaling, reduced operational overhead, and pay-per-use pricing models.
   
These are just a few examples of architectural types, and there are many more variations and combinations based on specific requirements and use cases. Choosing the right architecture depends on factors such as scalability needs, system complexity, team capabilities, and business goals.

