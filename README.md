# 👋 Hi, I'm Ben Nguyen

**🎓Computer Engineering Student at The University of Florida. 🐊**

Specialized in embedded systems and low-level software. <br>
I enjoy taking on tasks that challenge me in new ways. <br> Passionate about creating efficient and optimized devices.

## 📫 Connect with Me!
Phone: (239) 888-0663

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/nguyenhuuben)
[![Email](https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:benn5334@gmail.com)

## Featured Projects
<h2 align="center">🛒 E-Commerce Platform 🔗 <a href="https://github.com/HuuBen5334/ecommerce-fullstack">View Repo</a></h2>

<p align="center">
  <em>A full-stack, polyglot commerce backend engineered for correctness under concurrency — from a C++ pricing microservice up through a real-time React storefront.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Java-21-007396?style=for-the-badge&logo=openjdk&logoColor=white" alt="Java 21"/>
  <img src="https://img.shields.io/badge/Spring_Boot-3.5-6DB33F?style=for-the-badge&logo=springboot&logoColor=white" alt="Spring Boot"/>
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL"/>
  <img src="https://img.shields.io/badge/C%2B%2B-00599C?style=for-the-badge&logo=cplusplus&logoColor=white" alt="C++"/>
  <img src="https://img.shields.io/badge/gRPC-244c5a?style=for-the-badge&logo=grpc&logoColor=white" alt="gRPC"/>
  <img src="https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react&logoColor=black" alt="React 19"/>
  <img src="https://img.shields.io/badge/Locust-load_tested-2EAE5A?style=for-the-badge&logo=locust&logoColor=white" alt="Locust"/>
</p>

🔗 [View Repo](https://github.com/HuuBen5334/ecommerce-fullstack) 

---

## 🎬 Demo

<img width="1306" alt="E-commerce platform demo" src="https://github.com/user-attachments/assets/1d1935ba-ca4e-4035-9b00-263c9370942f" />

---

## 📖 Overview

This project is a production-shaped e-commerce backend with a complete storefront on top of it. It started as a CRUD REST API and grew into a **four-tier distributed system** built to answer one hard question that every real marketplace faces:

> **What happens when 50 people try to buy the last item at the same time?**

Getting that right meant deliberately reaching past a single language and a single process — a **C++ pricing engine** for hot-path dynamic pricing, **optimistic locking with automatic retry** for inventory integrity, a **WebSocket layer** for live order updates, and a **Locust load-test harness** to prove it all holds up under real concurrency rather than assuming it does.

---

## 🏗️ Architecture

```mermaid
flowchart LR
    subgraph Client["🖥️ React 19 Storefront :3000"]
        UI[Marketplace + Cart]
        WS_C[STOMP / SockJS client]
    end

    subgraph Backend["☕ Spring Boot API :8080"]
        CTRL[REST Controllers]
        SVC["ProductOrderService@Retryable + @Transactional"]
        WS_S[STOMP Broker /topic]
    end

    subgraph Pricing["⚙️ C++ Pricing Engine :50051"]
        GRPC["gRPC serverbulk discount + scarcity surge"]
    end

    DB[("🐘 PostgreSQLoptimistic lock @Version")]

    UI -->|REST /products /orders| CTRL
    CTRL --> SVC
    SVC -->|"GetPrice (gRPC)"| GRPC
    SVC -->|"JPA / Hibernate"| DB
    SVC -->|push order update| WS_S
    WS_S -.->|"/topic/orders/user/{id}"| WS_C
```

Each tier is independently runnable and speaks a purpose-fit protocol: **REST** for the storefront, **gRPC** for low-latency internal pricing calls, and **STOMP-over-WebSocket** for server-pushed order events.

---

## ⭐ Engineering Highlights

### 1. Surviving concurrent checkout

A 50-user load test exposed a **~96% `ObjectOptimisticLockingFailureException` rate** — users colliding on the same stock row. The layered fix:

- **`@Version` optimistic locking** on `Product` prevents silent overwrites of stock decrements.
- **`@Retryable` with backoff** (3 attempts, 50 → 100 ms) wraps *outside* the transaction so each retry runs fresh — ordering pinned via `TransactionConfig`.
- **FK indexes** on `product_order.product_id` and `user_id` keep order writes fast.

Result: contention is absorbed and the same test passes cleanly.

> **Optimistic over pessimistic** wins on throughput in the general case; pessimistic is only worth it under sustained flash-sale contention — a deliberate, documented tradeoff.

### 2. A C++ pricing microservice over gRPC

Pricing runs in a **multi-threaded C++ service** (`:50051`), not the JVM:

- Two RPCs — `GetPrice` (bulk-discount + scarcity-surge) and `UpdateDiscount` (admin override).
- A `std::shared_mutex` guards the override map: concurrent readers on the price path, exclusive writers on updates.
- The backend calls it via `PricingServiceBlockingStub` *before* snapshotting `priceAtPurchase`, so each order captures the exact price shown.

### 3. Real-time order notifications

No polling — orders surface instantly:

- STOMP broker on `/topic`; clients subscribe to `/topic/orders/user/{userId}`.
- React client connects via `@stomp/stompjs` + `sockjs-client`.
- Payload is a typed Java `record` (`OrderUpdateMessage`) → JSON.

### 4. Tested where it matters

- **Mockito unit tests** isolate `ProductOrderService` (mocked gRPC stub + notifier).
- **`@WebMvcTest`** covers the controller/HTTP contract.
- **`@SpringBootTest` integration test** runs a real STOMP client over SockJS against H2, asserting delivery within 5 s.

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| **API** | Spring Boot 3.5 · Java 21 · Spring Data JPA (Hibernate) |
| **Resilience** | Spring Retry · Spring AOP · optimistic locking (`@Version`) |
| **Database** | PostgreSQL · `BigDecimal` money · FK + PK indexing |
| **Pricing service** | C++ · gRPC · Protocol Buffers · `std::shared_mutex` |
| **Real-time** | STOMP · SockJS · in-memory `SimpleBroker` |
| **Frontend** | React 19 · React Router 7 · custom hooks · `React.memo` + lazy/`Suspense` |
| **Load testing** | Locust |

---

## 🔌 API Reference

<details>
<summary><strong>Products — <code>/products</code></strong></summary>

| Method | Path | Status |
|--------|------|--------|
| GET | `/products` | 200 |
| GET | `/products/{id}` | 200 / 404 |
| POST | `/products` | 201 |
| PUT | `/products/{id}` | 200 / 404 |
| DELETE | `/products/{id}` | 204 / 409 *(FK conflict)* |
</details>

<details>
<summary><strong>Users — <code>/users</code></strong></summary>

| Method | Path | Status |
|--------|------|--------|
| GET | `/users` | 200 |
| GET | `/users/{id}` | 200 / 404 |
| POST | `/users` | 201 |
| PUT | `/users/{id}` | 200 / 404 |
| DELETE | `/users/{id}` | 204 / 409 *(FK conflict)* |
</details>

<details>
<summary><strong>Orders — <code>/orders</code></strong></summary>

| Method | Path | Status |
|--------|------|--------|
| GET | `/orders` | 200 |
| GET | `/orders/{id}` | 200 / 404 |
| POST | `/orders?productId=&userId=&quantity=` | 201 *(→ `ProductOrderService`)* |
| PUT | `/orders/{id}` | 200 / 404 |
| DELETE | `/orders/{id}` | 204 |

**WebSocket:** subscribe to `/topic/orders/user/{userId}` → receives `{ orderId, userId, status, priceAtPurchase }` after each `placeOrder()`.
</details>

---

### <h2 align="center">🐟 Fish Schooling 🔗 <a href="https://github.com/HuuBen5334/FishSchooling">View Repo</a></h2>

An interactive 2D ocean sandbox built in Godot 4 + C# that brings autonomous steering behaviors to life through a full predator-prey ecosystem. Spawn different sea creatures, tune their behavior parameters in real time, and watch emergent dynamics unfold — schools scatter, predators intercept, and eels guard their territory.

![FishSchooling](https://github.com/user-attachments/assets/1bac9d65-154a-4b03-9257-5d022f44e2a8)

**Features:**
- **5 distinct creature types** — Nemo (schooling fish), Sharks, Starfish, Eels, and Orcas — each with a unique behavior loadout and role in the ecosystem
- **11 modular steering behaviors** built on a weighted `ISteeringBehavior` interface: Alignment, Cohesion, Separation, Pursuit, Interception (predictive), Flee, Path Following, Home Guard, Territory Defense, Wander, and Obstacle Avoidance
- **Classic Reynolds flocking** (Alignment + Cohesion + Separation) drives schooling fish, while predators use either direct pursuit or velocity-prediction interception
- **Territorial AI** — Eels maintain a home position and aggressively defend a radius around it; Orcas act as apex predators that hunt both Nemo and Eels using predictive movement
- **Live parameter control** — sliders apply to all existing fish instantly: adjust separation radius to tighten/loosen schools, tune predator hunt range, or change max speed mid-simulation
- **Debug visualization system** — toggle force-vector arrows per fish with 11 color-coded behavior channels (green = alignment, red = separation, magenta = flee, orange = pursuit, etc.) to inspect decision-making in real time
- **Dynamic spawning & census** — spawn up to 110 Nemo, 20 Sharks, 50 Starfish, 30 Eels, and 20 Orcas; live population counter updates as predators hunt prey
- **Screen-wrap world** with static rock obstacles that all fish pathfind around using urgency-scaled repulsion forces

🔗 [View Repository](https://github.com/HuuBen5334/FishSchooling)
## 🛠️ Tech Stack

![C#](https://img.shields.io/badge/C%23-239120?style=for-the-badge&logo=c-sharp&logoColor=white)
![Godot](https://img.shields.io/badge/Godot-478CBF?style=for-the-badge&logo=godot-engine&logoColor=white)
![.NET](https://img.shields.io/badge/.NET-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)


## 📫 Connect with Me!
Phone: (239) 888-0663

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/nguyenhuuben)
[![Email](https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:benn5334@gmail.com)
<!--- [![Portfolio](https://img.shields.io/badge/Portfolio-000000?style=for-the-badge&logo=github&logoColor=white)](https://your-site.com)
<!---
HuuBen5334/HuuBen5334 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
