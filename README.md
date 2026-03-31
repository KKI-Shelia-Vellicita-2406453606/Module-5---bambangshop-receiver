# BambangShop Receiver App
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
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create SubscriberRequest model struct.`
    -   [ ] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Notification repository.`
    -   [ ] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Commit: `Implement receive_notification function in Notification service.`
    -   [ ] Commit: `Implement receive function in Notification controller.`
    -   [ ] Commit: `Implement list_messages function in Notification service.`
    -   [ ] Commit: `Implement list function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1
1. In this tutorial, RwLock<> (Read-Write Lock) is used to synchronize the Vec of notifications because it is more efficient for scenarios where data is read frequently but modified less often. Unlike a standard Mutex<>, which only allows a single thread to access the data at any given time regardless of the operation, an RwLock<> allows multiple threads to read the data simultaneously as long as no thread is currently writing to it. In the case of the Receiver app, many users might be "reading" the list of messages at once, while "writing" (adding) a new notification only happens when the Publisher sends an update. Using RwLock<> prevents readers from blocking each other, ensuring the application remains responsive under high read volume while still maintaining thread safety during updates.

2. Rust's strict refusal to allow the direct mutation of static variables unlike Java. It is rooted in its core philosophy of memory safety and the prevention of data races. In Java, static variables can be modified globally, but this often leads to unpredictable behavior or crashes if multiple threads attempt to change the same variable at the same time without manual synchronization. Rust's compiler is designed to catch these "data races" at compile time, it treats global mutable state as inherently "unsafe" because there is no built in guarantee of synchronized access. By using the lazy_static library, we can safely initialize complex static variables like a Vec or DashMap at runtime while wrapping them in synchronization primitives like RwLock. This forces the developer to handle thread safety explicitly, satisfying the compiler's rigorous constraints and ensuring a "fearless" concurrent program.

#### Reflection Subscriber-2
1. Exploring files like src/lib.rs provides critical insight into how the application manages its global state and configuration beyond the specific service logic. In this project, src/lib.rs is where essential components like the APP_CONFIG are defined, which handles the loading of environment variables such as the publisher's root URL and the specific instance name. It also initializes the REQWEST_CLIENT, a shared HTTP client used throughout the application to ensure efficient connection pooling when sending requests. Understanding these "behind-the-scenes" parts of the code is vital for a developer to see how a Rust application bootstraps itself and manages shared resources in a thread-safe manner before the controllers even begin handling requests.

2. The Observer pattern significantly eases the process of "plugging in" new subscribers because the Publisher remains completely decoupled from the specific implementation of each Receiver. Because the Publisher only needs to maintain a list of URLs and names, any new instance like the three receivers you spawned on ports 8001, 8002, and 8003. It can be added to the system simply by registering its endpoint. However, scaling the system to include multiple instances of the Main App (Publisher) introduces a new layer of complexity. Since the current implementation stores the subscriber list in an in-memory DashMap, a new Publisher instance would have its own independent, empty list of subscribers. To make this truly scalable, the Publishers would need to share a centralized persistent database for the subscriber list so that a subscription made on one instance is recognized by all others.

3. Utilizing Postman is highly effective for verifying that the REST API behaves as expected, particularly when checking if the application correctly returns specific HTTP status codes like 201 Created for new subscriptions or 200 OK when viewing message lists. Enhancing a Postman collection with detailed documentation and automated test scripts is incredibly useful for software engineering projects because it allows for "regression testing," ensuring that new code changes haven't accidentally broken existing notification formats. For any developer, these tools are essential for bridging the gap between back-end logic and the actual data consumption, providing a reliable way to simulate various user scenarios and ensure the JSON payloads are accurately structured before deployment.