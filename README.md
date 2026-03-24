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

1. In this tutorial, we used `RwLock<>` to synchronise the use of `Vec<Notification>`. Explain why it is necessary for this case, and explain why we do not use `Mutex<>` instead?

    `RwLock<>` is necessary because Rust's ownership and borrowing rules don't allow mutable access to a `static` variable without some form of synchronization. Since `NOTIFICATIONS` is a shared static variable that can be accessed by multiple threads (Rocket handles requests concurrently), we need a synchronization primitive. We use `RwLock` instead of `Mutex` because `RwLock` allows multiple concurrent readers while only needing exclusive access for writers. In this app, reading the notification list (like listing all messages) is expected to happen more often than writing (adding new notifications). With `Mutex`, every access, even read-only ones, would need exclusive locking, which creates unnecessary contention. `RwLock` is better for this read-heavy pattern since it lets multiple threads read at the same time.

2. In this tutorial, we used `lazy_static` external library to define `Vec` and `DashMap` as a `static` variable. Explain based on your understanding of using Rust, is it possible to have a mutable `static` variable in Rust without using `lazy_static`?

    In Rust, you can technically have a mutable static variable using `static mut`, but it's unsafe and strongly discouraged. Accessing `static mut` variables requires `unsafe` blocks because the compiler can't guarantee thread safety, since multiple threads could read and write at the same time and cause data races. Rust's design philosophy is all about enforcing safety at compile time, so `static mut` is intentionally hard to use. `lazy_static` (or the newer `std::sync::OnceLock`/`std::sync::LazyLock` in recent Rust versions) gives us a safe way to initialize static variables lazily at runtime and combine them with thread-safe wrappers like `RwLock` or `DashMap` to allow mutation without any unsafe code.

#### Reflection Subscriber-2

1. Have you explored things outside of the steps in the tutorial, for example: `src/lib.rs`? If not, explain why you did not do so; if yes, explain things that you have learned from those other parts of the code.

    Yes, I looked into `src/lib.rs` in the receiver app. It defines the `AppConfig` struct with three fields: `instance_root_url`, `publisher_root_url`, and `instance_name`, and it auto-generates getters for each using the `getset` crate. The configuration gets loaded from environment variables prefixed with `APP_` using `dotenvy` and Rocket's `Figment`. There's also the `REQWEST_CLIENT` for making outgoing HTTP requests and the shared `Result`/`Error` types that are used across the whole application. The main difference compared to the publisher's `lib.rs` is that the receiver needs to know the publisher's URL so it can send subscribe/unsubscribe requests, and it also has an instance name field used for identification in notifications.

2. Since you have finished the tutorial by now and have tried to test your notification system by running 3 instances of Receiver and 1 instance of Main app, explain how Observer pattern eases you to plug in more subscribers and how spawn more than one instance of Main app will affect your notification system.

    The Observer pattern makes it really easy to plug in more subscribers because the publisher doesn't need to know anything about subscribers in advance. Any new receiver instance can just subscribe to the publisher's notification system at runtime by calling the `/subscribe/<product_type>` endpoint. The publisher keeps a dynamic list of subscribers and loops through all of them when sending notifications, so adding a new subscriber doesn't require any code changes. You just spin up a new receiver instance on a different port and have it subscribe. On the other hand, spawning more than one instance of the Main app would cause problems because each instance keeps its own in-memory `SUBSCRIBERS` DashMap. Subscribers that register with one publisher instance wouldn't exist in the other one, so notification delivery would be inconsistent. To support multiple publisher instances, we'd need some kind of shared external data store like a database or Redis for the subscriber list.

3. Have you tried to make your own Tests, or enhanced documentation on your Postman collection? If you have tried those features, tell us whether it is useful for your project or not.

    Postman has been really useful for testing the endpoints of this Observer pattern implementation. The provided Postman collection lets you quickly verify the subscribe/unsubscribe flows, product creation, and notification delivery across multiple receiver instances. Being able to organize requests into folders (Publisher vs Receiver) and set environment variables for different ports makes it pretty convenient to test the whole system end-to-end. For group projects, features like shared collections, automated test scripts (using the Tests tab), and environment management would be really helpful for making sure API contracts are met and for doing regression testing.
