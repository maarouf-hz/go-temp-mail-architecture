# Building a High-Performance Disposable Email Service in Go 🚀

> **Live Demo:** You can see this architecture running in production at **[Box-Mail.org](https://box-mail.org)**.

## 📖 Introduction

"Temp Mail" services are essential privacy tools that allow users to receive emails without exposing their primary inbox to spam. While many exist, I wanted to build one that was **exceptionally fast**, **lightweight**, and **scalable**.

I chose **Go (Golang)** for the backend because of its superior handling of concurrency and low memory footprint.

This repository documents the **System Architecture** and **Design Decisions** behind [Box-Mail.org](https://box-mail.org).

---

## 🏗️ System Architecture

The system is built on a micro-service architecture designed to handle high-throughput SMTP traffic while maintaining real-time updates for the user.

### 1. The SMTP Server (The Core)
Instead of using a standard mail server like Postfix (which is heavy), I wrote a custom SMTP server in Go.
* **Listening:** It listens on port 25 for incoming TCP connections.
* **Filtering:** Unlike a normal server that checks if a user exists, this server accepts **wildcard recipients**. Anything ending in `@box-mail.org` is accepted.
* **Parsing:** Incoming data streams are parsed in real-time to extract the Sender, Subject, and Body (HTML/Text).

### 2. In-Memory Storage (The Speed)
Since the emails are "disposable" and have a short Time-To-Live (TTL), persisting them to a traditional SQL database would be too slow and unnecessary.
* **Data Structure:** I use thread-safe maps with `sync.RWMutex` to store active sessions and emails.
* **Auto-Cleanup:** A background Goroutine (worker) runs every minute to clean up expired sessions, ensuring zero data retention after the user leaves.

### 3. Real-Time Delivery (The UX)
The most critical feature is **instant feedback**. Users shouldn't have to refresh the page.
* **Mechanism:** The frontend establishes a connection to the backend API.
* **Push:** When the SMTP service receives an email, it pushes the data immediately to the specific active client session via a secure channel.

---

## 🛠️ Technology Stack

* **Language:** Go (Golang) 1.21+
* **Containerization:** Docker & Docker Compose
* **Web Server:** Nginx (Alpine Linux)
* **Frontend:** Vanilla JavaScript (No heavy frameworks for maximum load speed)
* **Protocol:** SMTP, HTTP/2

---

## ⚡ Performance Optimization

To ensure [Box-Mail.org](https://box-mail.org) loads instantly globally:
1.  **Single Binary:** The entire Go backend compiles into a single static binary (~10MB).
2.  **Goroutines:** Each incoming email connection is handled by a lightweight Goroutine, allowing the server to process thousands of concurrent emails with minimal CPU usage.
3.  **Minimal Docker Image:** The frontend is served via an optimized Nginx Alpine image, stripping unused modules.

---

## 🔒 Security & Privacy

Privacy is the main product. The architecture enforces this by design:
* **No Logs:** The custom Go logger is configured to discard sender metadata.
* **RAM-Only:** Data lives in RAM. If the server restarts, all data is wiped instantly.
* **TLS/SSL:** All connections are secured via Let's Encrypt.

---

## 🔗 Try It Live

I have deployed this architecture to a production environment. You can test the speed and responsiveness yourself:

👉 **[Visit Box-Mail.org](https://box-mail.org)**

---

*Note: This repository contains the architectural documentation. The source code is currently private as it is part of a commercial project.*
