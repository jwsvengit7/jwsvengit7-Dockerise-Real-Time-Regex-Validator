# 🧪 Real-Time Regex Validator (Fullstack App)

A real-time fullstack web app that validates user-submitted strings against a configurable regular expression. Designed to demonstrate scalable, event-driven architecture using **NestJS**, **React**, **Kafka**, **Redis**, and **MongoDB/PostgreSQL** — fully containerized with Docker Compose.

---

## 🧠 Purpose & Workflow

1. A user enters a text string in the frontend.
2. The backend accepts the request and queues it via Kafka.
3. A background service asynchronously validates the string using a regex pattern from environment config.
4. The result (`Validating`,`Valid`, `Invalid`) is pushed to the frontend in real-time via WebSocket.
5. A persistent history of submissions is shown on the UI.

---

## 🗂️ Project Structure

```
real-time-regex-validator/
├── docker-compose.yml
├── titans-backend/
│   ├── src/
│   │   ├── jobs/
│   │   ├── kafka/
│   │   ├── redis/
│   │   ├── common/
│   │   ├── database/
│   │   ├── app.module.ts
│   │   └── main.ts
│   └── Dockerfile
├── titans-frontend/
│   ├── src/
│   │   ├── App.tsx
│   │   ├── index.tsx
│   │   ├── App.css
│   │   └── ...
│   └── Dockerfile
```

---

## ⚙️ Tech Stack

| Layer     | Technology |
|-----------|------------|
| Frontend  | React + Vite + WebSocket |
| Backend   | NestJS (Node.js) |
| Messaging | Kafka |
| Pub/Sub   | Redis |
| Database  | MongoDB |
| Orchestration | Docker Compose |

---

## 🏗️ Architecture Overview

### ASCII Diagram

```
+-----------+        HTTP        +----------+     Kafka     +-------------+
|  Frontend |  +---------------> |  Backend | +-----------> | Kafka Topic |
|  (React)  |                   | (NestJS) |               | "regex-jobs"|
+-----------+                  /+----------+\              +-------------+
   ↑         WebSocket        /     ↑       \   WebSocket Broadcast
   |  <----------------------+  Job Gateway  +------------> Clients
   |                         \               /
   |                          \             /   Redis PubSub
   |                           \           /  +---------------+
   |                            \         /   | Redis Channel |
   |                             \       /    +---------------+
   |                              \     /
   |                             +---------+     MongoDB 
   +-----------------------------+ Processor +--------------------> Store Results
                                 +---------+
```

---

## ⚡ Real-Time Update Mechanism

- Kafka queues jobs for validation.
- A NestJS worker consumes Kafka messages and validates strings.
- Redis is used to **publish job status** to the WebSocket gateway.
- Frontend listens on WebSocket for `job-status` updates.

---

## 🛠 How to Run Locally

### 🔧 Prerequisites

- Node.js (v18 recommended)
- Docker & Docker Compose

### 🐳 Run with Docker (recommended)

```bash
cp titans-backend/.env.example titans-backend/.env
docker compose up --build
```

Frontend will be available at:  
👉 `http://localhost:61234`

---

### 🧪 Run Without Docker (Dev Mode)

In separate terminals:

#### 🟢 Backend

```bash
cd titans-backend
npm install
cp .env.example .env
npm run start:dev
```

#### 🟠 Frontend

```bash
cd titans-frontend
npm install
npm run dev
```

Then visit: `http://localhost:5173`

---

## 🌍 Environment Variables (Backend)

```env
REGEX_PATTERN=^[a-zA-Z0-9]+$
MONGODB_URI=mongodb://root:root@mongo:27017/regex-validator?authSource=admin
REDIS_HOST=redis
KAFKA_BROKER=kafka:9093
PROCESS_DELAY_MS=2000
```

---

## 📦 Component Responsibilities

- `JobsService`: handles job creation, DB persistence (Mongo), Kafka publishing.
- `JobsProcessor`: listens to Kafka, processes validation, updates status.
- `JobsGateway`: emits WebSocket events to frontend.
- `RedisService`: handles publish of job status over Redis pub/sub.
- `KafkaService`: wraps `kafkajs` for producer/consumer logic.
- `React App`: submits strings, shows real-time status, fetches history.

---

## 🔁 Fault Tolerance & Reliability

- Kafka ensures job durability (even if backend restarts).
- Redis decouples communication between services.
- Docker Compose can auto-restart failed containers.
- MongoDB ensures persistence even under process failures.
- Input validation and error handling via NestJS decorators.

---

## ☁️ Hypothetical AWS Deployment Strategy

> While this project is not yet deployed on AWS, here is how it could be:

| Component | AWS Service |
|----------|-------------|
| Kafka    | Amazon MSK |
| Redis    | Amazon ElastiCache |
| MongoDB  | MongoDB Atlas or self-hosted on EC2 |
| Backend  | ECS Fargate or EKS |
| Frontend | S3 + CloudFront or ECS |
| Secrets  | AWS Secrets Manager or SSM Parameter Store |

### Config Management

- Use `.env` files in local, Secrets Manager in production.
- Inject via ECS task definitions or EKS environment config.

### Monitoring & Scaling

- Use **CloudWatch Logs** and **CloudWatch Alarms**.
- Auto-scale with ECS Service Auto Scaling.
- Consider managed services to reduce ops overhead.

---

## 🧪 Testing

```bash
# Inside the running backend container
docker exec -it nestjs_backend npm run test
```

---

## 📄 License

MIT – feel free to reuse this architecture for other event-driven systems!!

---

Thank you SavannahTech for this great opportunity to build this project.
