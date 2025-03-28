# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

1. Based on my understanding, an interface is beneficial for flexibility and scalability. In a regular Observer pattern, the `Subscriber` is often an interface so that different types of subscribers can implement their own behavior for handling updates. However, if all subscribers have the same structure and behavior, a single Subscriber struct is enough.

2. A `Vec` (list) could work if the primary operation was appending subscribers or iterating over all of them. However, since we require frequent lookups and deletions based on the `url`, a `Vec` would be inefficient. On the other hand, a `DashMap` provides O(1) average-time complexity for these operations, making it a more appropriate choice for managing a list of `Subscribers`.

3. The Singleton pattern ensures that there is only one instance of a resource shared across the application. The `SUBSCRIBERS` static variable implemented with `lazy_static!` essentially behaves like a Singleton already, ensuring centralized access to the subscribers' data. However, the unique feature of `DashMap` is that it provides built-in thread safety which is critical in Rust when data is shared across threads.

#### Reflection Publisher-2

1. Separating `Service` and `Repository` from the `Model` in the MVC pattern established design principles like the Single Responsibility Principle and Separation of Concerns. The `Model` inherently handles both business logic and data storage in the MVC architecture, but combining these two responsibilities into one leads to a system that's harder to maintain, test, and scale. By introducing a `Service` layer, the business logic is encapsulated separately, making it easier to adapt workflows and transaction handling without impacting data representation or storage. Similarly, the `Repository` layer abstracts the data access logic, isolating it from business logic.

2. If we only use the Model without separating these layers, the interactions between models like `Program`, `Subscriber`, and `Notification` would become intertwined. If each model directly accesses the database and manages its own business logic, it would lead to duplicated code and conflicts in how each model handles shared data or dependencies. This complexity makes debugging and testing difficult because changing something in one model can inadvertently break another. 

3. Regarding Postman, I’ve explored it and it's really helpful in testing APIs in real-time. I use Postman to send HTTP requests to endpoints like the ones defined in the `Notification` controller. It allows me to test functionality, validate responses, and debug issues without needing to open the browser directly. 

Features I find useful: 
- Collection Runner
- Environment Management

#### Reflection Publisher-3

1. We are using the Push Model. The `notify` method pushes data directly to each `Subscriber` by calling the `update` method on each `subscriber` and immediately sending the notification payload through an HTTP request using the `REQWEST_CLIENT`.

2. 
    Advantages:
    - Subscribers have more control over when and how they fetch the data, avoiding unnecessary updates if the data is not needed immediately.
    - It can reduce payload size in the notification requests since only the notification of the event itself is communicated.

    Disadvantages:
    - Subscribers must implement logic to fetch the relevant data which requires them to know how to query the publisher.
    - Subscribers must make additional requests to the publisher which introduces delays compared to the Push Model where the data is delivered immediately.
    - Risk of inconsistency if the data changes between the notification and the subscriber's fetch.

3. If we removed multi-threading from the notification process, the program would send notifications to subscribers synchronously. This means the main thread would process each subscriber's notification before moving to the next one which would caused signigicant delays.