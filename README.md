# SonoCode-Coding-Platform-Backend
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white)
![Fastify](https://img.shields.io/badge/Fastify-000000?style=for-the-badge&logo=fastify&logoColor=white)
![Express.js](https://img.shields.io/badge/Express.js-000000?style=for-the-badge&logo=express&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![BullMQ](https://img.shields.io/badge/BullMQ-D83A3A?style=for-the-badge)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Socket.io](https://img.shields.io/badge/Socket.io-010101?style=for-the-badge&logo=socketdotio&logoColor=white)
![Axios](https://img.shields.io/badge/Axios-5A29E4?style=for-the-badge&logo=axios&logoColor=white)
![Zod](https://img.shields.io/badge/Zod-3E67B6?style=for-the-badge&logo=zod&logoColor=white)
---
SonoCode is a high-performance, distributed coding platform inspired by LeetCode, built with advanced microservices architecture principles. This project demonstrates enterprise-level backend development concepts including event-driven architecture, asynchronous processing, real-time communication, and tackles advance system design problems.

# System Design & Data-Flow
The architecture is a decoupled, event-driven system where the Submission Service acts as the primary ingress point. Upon receiving a request, it persists an initial state and publishes a job to a Submission Redis Queue. A separate Evaluator Service worker consumes this job, spins up a secure, isolated Docker container for sandboxed code execution, and upon completion, publishes the results to an Evaluation Redis Queue. The Submission Service is also subscribed to this queue, and upon receiving the result, it updates the database and pushes the final payload to the WebSocket Service, which relays it to the client for asynchronous, real-time feedback. This design ensures non-blocking I/O, high throughput, and resilience by decoupling the API from the resource-intensive evaluation logic.

![Testing png 1](https://github.com/Himu336/SonoCode-Coding-Platform-Backend/blob/main/SonoCode-HLD.excalidraw.png)

# 🔑 Key Highlights:
| Feature | Description |
| :--- | :--- |
| **Distributed Microservices** | The platform is built as a collection of independent, domain-driven services (Problems, Submissions, Evaluation, Sockets). |
| **High-Performance API Framework** | The Submission Service is built using Fastify, a low-overhead web framework, to handle a high volume of incoming code submissions efficiently. |
| **Redis-Powered Message Queues** | Utilizes Redis as a message broker for asynchronous task handling between the submission and evaluation services. |
| **Async Pub/Sub Model** | Implements a publisher/subscriber pattern using Redis for broadcasting events and state changes across the system. |
| **WebSocket for Real-Time IO** | Employs WebSockets for persistent, bidirectional communication, providing instant feedback on submission status to the user. |
| **Scalable Session Management** | Uses Redis as a distributed cache to store user-to-socket-ID mappings, enabling stateful connection management across multiple Socket Service instances. |
| **Dynamic Docker Container Factory** | The evaluation service dynamically creates and manages isolated Docker containers for executing user-submitted code securely. |
| **Secure Execution Sandbox** | Leverages Docker to create a completely isolated environment for each submission, restricting file system, network, and process access. |
| **Resource Exhaustion Mitigation** | Imposes strict resource limits (CPU, memory, process count) on each Docker container to prevent attacks like **fork bombs**. |
| **CI/CD & Shell Automation** | Leverages shell scripting for service configuration, build processes, and deployment automation. |
| **Resilient & Fault-Tolerant Design** | Asynchronous communication and service decoupling inherently make the system more resilient to individual service failures. |

## 📦 Repository and Service Links
This section maps out each core microservice and its corresponding repository.

| Service Name | Description | Repository Link |
| :--- | :--- | :--- |
| **Submission Service** | Acts as the primary API ingress, handling all incoming code submissions and orchestrating the workflow. | [SonoCode-Submission-Service](https://github.com/Himu336/SonoCode-Submission-Service) |
| **Evaluator Service** | A background worker that consumes jobs from the queue and evaluates code in a secure, sandboxed Docker environment. | [SonoCode-Evaluator-Service](https://github.com/Himu336/SonoCode-Evaluator-Service) |
| **Socket Service** | Manages persistent WebSocket connections for pushing real-time evaluation results and status updates to clients. | [SonoCode-Socket-Service](https://github.com/Himu336/SonoCode-Socket-Service) |
| **Problem Admin Service** | Handles all CRUD (Create, Read, Update, Delete) operations for managing coding problems and their test cases. | [SonoCode-Problem-Service](https://github.com/Himu336/SonoCode-Problem-Service) |

## 🗃️ Database Schema

The project uses two main MongoDB collections: `problems` and `submissions`.

### 1. `problems` Collection
Stores all data for a single coding problem.
| Field | Type | Description |
| :--- | :--- | :--- |
| `_id` | `ObjectId` | Unique problem ID. |
| `title` | `String` | Problem title. |
| `description` | `String` | Full problem statement. |
| `difficulty` | `String` | Difficulty level (e.g., "easy"). |
| `testCases` | `Array<Object>` | Array of `{input, output}` objects for validation. |
| `codeStubs` | `Array<Object>` | Language-specific boilerplate code. |
<details>
<summary>View example document</summary>

```json
{
  "_id": { "$oid": "68ac63e7823a7a68037ca1e0" },
  "title": "Valid Palindrome 3",
  "description": "A phrase is a palindrome if...",
  "difficulty": "easy",
  "testCases": [ { "input": "race a car", "output": "true" } ],
  "codeStubs": [ { "language": "CPP", "startSnippet": "...", "endSnippet": "...", "userSnippet": "..." } ]
}
```
</details>

### 2. `submissions` Collection
Stores a user's code submission for a specific problem.
| Field | Type | Description |
| :--- | :--- | :--- |
| _id | ObjectId | Unique submission ID. |
| userId | String | ID of the user who made the submission. |
| problemId| ObjectId | Foreign key referencing the problems collection. |
| code | String | The user's submitted source code. |
| language | String | Language of the submission (e.g., "CPP"). |
| status | String | Current status (e.g., "Pending", "Success"). |

<details>
<summary>View example document</summary>

```json

{
  "_id": { "$oid": "68ac61b251ed0b4484ff9a20" },
  "userId": "1",
  "problemId": "68ac2d20c2911d94596215cb",
  "code": "class Solution { ... };",
  "language": "CPP",
  "status": "Success"
}
```
</details>

## 🚀 Test My Backend
You can test the entire code submission and evaluation workflow — from submitting a solution to receiving real-time results — using the Postman collection linked below. This collection includes all available API endpoints across the SonoCode microservices.

🔗 **Postman Collection:** [Insert your Postman link here]
