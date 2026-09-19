# To-Do List REST API — Full Project Overview & Demo

A clean, modular, and production-ready **To-Do List REST API** built with **Node.js, Express.js, MongoDB, and Mongoose**. It implements full **CRUD** (Create, Read, Update, Delete) functionality with defensive validation, centralized error handling, custom DNS resolution for MongoDB Atlas, and strict adherence to RESTful principles.

---

## Table of Contents

- [1. Project Overview](#1-project-overview)
- [2. Technology Stack](#2-technology-stack)
- [3. Architecture & Directory Structure](#3-architecture--directory-structure)
- [4. Data Model & Schema](#4-data-model--schema)
- [5. Environment Configuration](#5-environment-configuration)
- [6. Installation & How to Run](#6-installation--how-to-run)
- [7. API Endpoints & Response Contracts](#7-api-endpoints--response-contracts)
- [8. Full End-to-End Live Demo (cURL & Postman)](#8-full-end-to-end-live-demo-curl--postman)
  - [Demo 1: Health Check](#demo-1-health-check)
  - [Demo 2: Create Tasks (POST)](#demo-2-create-tasks-post)
  - [Demo 3: Get All Tasks (GET)](#demo-3-get-all-tasks-get)
  - [Demo 4: Get Single Task by ID (GET)](#demo-4-get-single-task-by-id-get)
  - [Demo 5: Update Task (PUT)](#demo-5-update-task-put)
  - [Demo 6: Delete Task (DELETE)](#demo-6-delete-task-delete)
  - [Demo 7: Undefined Route 404](#demo-7-undefined-route-404)
- [9. Error Handling Architecture](#9-error-handling-architecture)
- [10. Key Engineering Highlights](#10-key-engineering-highlights)
- [11. Verification Checklist](#11-verification-checklist)

---

## 1. Project Overview

This API manages a collection of tasks (To-Do items), allowing client applications (web apps, mobile apps, or API testing clients) to perform standard task management operations:
- **Create** tasks with a required `title` and an optional `description`.
- **Read** all stored tasks (sorted by creation date, newest first) or retrieve an individual task by its unique ID.
- **Update** task fields (`title`, `description`, `completed` status).
- **Delete** tasks permanently from the database.

### Key Architectural Strengths
- **Separation of Concerns**: Clean separation between routing, controllers, data models, middleware, and database configurations.
- **Defensive Validation**: Guards against invalid MongoDB `ObjectId` formats before hitting the database, preventing unhandled exceptions.
- **Custom DNS Resolution**: Overrides DNS servers to Google & Cloudflare (`1.1.1.1`, `8.8.8.8`) to eliminate MongoDB Atlas SRV lookup failures common on Windows and restricted networks.
- **Uniform Response Structure**: Every response strictly adheres to predictable JSON envelopes (`success`, `message`, `data`).

---

## 2. Technology Stack

| Component | Technology | Version | Purpose |
| :--- | :--- | :--- | :--- |
| **Runtime** | Node.js | `>= 18.x` | Asynchronous, event-driven JavaScript server environment |
| **Framework** | Express.js | `^4.19.2` | HTTP request routing, middleware pipeline, and JSON parsing |
| **Database** | MongoDB Atlas | Cloud Cluster | Document-oriented NoSQL storage |
| **ODM** | Mongoose | `^8.5.0` | Schema definitions, type casting, validation, and database queries |
| **Environment**| dotenv | `^16.4.5` | Decouples configuration (port, connection string) from source code |
| **Dev Tooling**| nodemon | `^3.1.4` | Development auto-reload server |

---

## 3. Architecture & Directory Structure

```text
TO-DO-List/
├── config/
│   └── db.js               # MongoDB connection setup with custom DNS resolver
├── controllers/
│   └── taskController.js   # Business logic & request handling for tasks
├── middleware/
│   └── errorHandler.js     # Centralized global error handling middleware
├── models/
│   └── Task.js             # Mongoose Task schema definition
├── routes/
│   └── taskRoutes.js       # RESTful route definitions mapped to controller methods
├── .env                    # Environment variables (excluded from git)
├── .env.example            # Sample environment variables template
├── .gitignore              # Git ignore rules (node_modules, .env)
├── package.json            # Dependencies and npm scripts
├── Server.js               # Main application bootstrap and Express server entry point
└── README.md               # Project documentation and demo guide
```

### Component Breakdown
- **`Server.js`**: Application entry point. Boots up the database connection, configures `express.json()` middleware, registers route handlers, mounts the 404 fallback, and attaches the global error handler.
- **`config/db.js`**: Connects Mongoose to the MongoDB cluster specified by `MONGO_URI`. Pre-configures Cloudflare (`1.1.1.1`) and Google (`8.8.8.8`) DNS servers.
- **`models/Task.js`**: Defines the data model, including required fields, string trimming, defaults, and timestamps.
- **`controllers/taskController.js`**: Pure business logic layer. Handles input validation, queries Mongoose, and formats responses.
- **`routes/taskRoutes.js`**: Connects URL paths and HTTP verbs directly to controller actions.
- **`middleware/errorHandler.js`**: Catches all forwarded errors (`next(error)`), sanitizes error messages, and maps them to appropriate HTTP status codes.

---

## 4. Data Model & Schema

The Task entity is defined in `models/Task.js`:

```javascript
const mongoose = require('mongoose');

const taskSchema = new mongoose.Schema(
  {
    title: {
      type: String,
      required: [true, 'Title is required'],
      trim: true,
    },
    description: {
      type: String,
      trim: true,
      default: '',
    },
    completed: {
      type: Boolean,
      default: false,
    },
  },
  {
    timestamps: true, // Automatically manages createdAt and updatedAt
  }
);

module.exports = mongoose.model('Task', taskSchema);
```

### Field Definitions
| Field | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `_id` | ObjectId | Auto | Auto | Unique MongoDB document identifier |
| `title` | String | Yes | N/A | Title of the task (trimmed, non-empty) |
| `description` | String | No | `""` | Additional notes or details regarding the task |
| `completed` | Boolean | No | `false` | Status indicating if the task is done |
| `createdAt` | Date | Auto | Current Time | Timestamp when task was created |
| `updatedAt` | Date | Auto | Current Time | Timestamp when task was last modified |

---

## 5. Environment Configuration

The application requires configuration through a `.env` file in the root directory.

A template is provided in `.env.example`:

```env
MONGO_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/todoapp?retryWrites=true&w=majority
PORT=7000
```

> **Note**: Never commit your active `.env` file to version control. It is protected by `.gitignore`.

---

## 6. Installation & How to Run

### Prerequisites
- [Node.js](https://nodejs.org/) (v18 or higher recommended)
- A MongoDB Atlas database cluster (or local MongoDB running on `mongodb://127.0.0.1:27017/todoapp`)

### 1. Clone & Install Dependencies
```bash
# Navigate to the project directory
cd "c:/Users/garai/OneDrive/Documents/Backend Task/TO-DO-List"

# Install dependencies
npm install
```

### 2. Configure Environment
Create a `.env` file (or copy `.env.example`):
```bash
# Set your MongoDB connection string and port
MONGO_URI=your_mongodb_atlas_connection_string
PORT=7000
```

### 3. Run the Server

#### Development Mode (with automatic restart on file edits):
```bash
npm run dev
```

#### Production Mode:
```bash
npm start
```

Upon successful startup, the terminal outputs:
```text
MongoDB Connected: <cluster-host>
Server running on http://localhost:7000
```

---

## 7. API Endpoints & Response Contracts

Base URL: `http://localhost:7000`

| Method | Endpoint | Description | Status Codes |
| :--- | :--- | :--- | :--- |
| `GET` | `/` | Health check route | `200` |
| `POST` | `/api/tasks` | Create a new task | `201`, `400` |
| `GET` | `/api/tasks` | Get all tasks (newest first) | `200` |
| `GET` | `/api/tasks/:id` | Get single task by ID | `200`, `400`, `404` |
| `PUT` | `/api/tasks/:id` | Update an existing task | `200`, `400`, `404` |
| `DELETE` | `/api/tasks/:id` | Remove a task | `200`, `400`, `404` |

### Standard Response Formats

#### Success Response Envelope:
```json
{
  "success": true,
  "message": "Optional descriptive message",
  "data": { }
}
```

#### Error Response Envelope:
```json
{
  "success": false,
  "message": "Descriptive reason for error"
}
```

---

## 8. Full End-to-End Live Demo (cURL & Postman)

Below are the complete requests, cURL commands, and responses to demo all operations and edge cases.

### Demo 1: Health Check

Verify the API server is up and accepting incoming requests.

- **Method**: `GET`
- **URL**: `http://localhost:7000/`

```bash
curl -X GET http://localhost:7000/
```

**Expected Response (`200 OK`):**
```json
{
  "message": "To-Do API is running"
}
```

---

### Demo 2: Create Tasks (POST)

#### 2.1 Successful Task Creation
- **Method**: `POST`
- **URL**: `http://localhost:7000/api/tasks`
- **Header**: `Content-Type: application/json`

```bash
curl -X POST http://localhost:7000/api/tasks \
  -H "Content-Type: application/json" \
  -d "{\"title\":\"Build Backend API\",\"description\":\"Implement CRUD endpoints with Express & MongoDB\"}"
```

**Expected Response (`201 Created`):**
```json
{
  "success": true,
  "message": "Task created successfully",
  "data": {
    "_id": "665f1c2e8b1e4a0012a3f9c1",
    "title": "Build Backend API",
    "description": "Implement CRUD endpoints with Express & MongoDB",
    "completed": false,
    "createdAt": "2026-09-19T16:25:00.000Z",
    "updatedAt": "2026-09-19T16:25:00.000Z",
    "__v": 0
  }
}
```

#### 2.2 Validation Error (Missing Title)
```bash
curl -X POST http://localhost:7000/api/tasks \
  -H "Content-Type: application/json" \
  -d "{\"description\":\"Missing title test\"}"
```

**Expected Response (`400 Bad Request`):**
```json
{
  "success": false,
  "message": "Title is required"
}
```

---

### Demo 3: Get All Tasks (GET)

Fetches all tasks sorted in descending order by `createdAt` (newest first).

- **Method**: `GET`
- **URL**: `http://localhost:7000/api/tasks`

```bash
curl -X GET http://localhost:7000/api/tasks
```

**Expected Response (`200 OK`):**
```json
{
  "success": true,
  "count": 1,
  "data": [
    {
      "_id": "665f1c2e8b1e4a0012a3f9c1",
      "title": "Build Backend API",
      "description": "Implement CRUD endpoints with Express & MongoDB",
      "completed": false,
      "createdAt": "2026-09-19T16:25:00.000Z",
      "updatedAt": "2026-09-19T16:25:00.000Z",
      "__v": 0
    }
  ]
}
```

---

### Demo 4: Get Single Task by ID (GET)

#### 4.1 Valid Existing Task
- **Method**: `GET`
- **URL**: `http://localhost:7000/api/tasks/665f1c2e8b1e4a0012a3f9c1`

```bash
curl -X GET http://localhost:7000/api/tasks/665f1c2e8b1e4a0012a3f9c1
```

**Expected Response (`200 OK`):**
```json
{
  "success": true,
  "data": {
    "_id": "665f1c2e8b1e4a0012a3f9c1",
    "title": "Build Backend API",
    "description": "Implement CRUD endpoints with Express & MongoDB",
    "completed": false,
    "createdAt": "2026-09-19T16:25:00.000Z",
    "updatedAt": "2026-09-19T16:25:00.000Z",
    "__v": 0
  }
}
```

#### 4.2 Malformed Task ID
```bash
curl -X GET http://localhost:7000/api/tasks/invalid-id-xyz
```

**Expected Response (`400 Bad Request`):**
```json
{
  "success": false,
  "message": "Invalid task ID"
}
```

#### 4.3 Non-Existent Task ID (Valid ObjectId format)
```bash
curl -X GET http://localhost:7000/api/tasks/665f1c2e8b1e4a0012a3f999
```

**Expected Response (`404 Not Found`):**
```json
{
  "success": false,
  "message": "Task not found"
}
```

---

### Demo 5: Update Task (PUT)

Update any combination of `title`, `description`, or `completed`.

- **Method**: `PUT`
- **URL**: `http://localhost:7000/api/tasks/665f1c2e8b1e4a0012a3f9c1`
- **Header**: `Content-Type: application/json`

```bash
curl -X PUT http://localhost:7000/api/tasks/665f1c2e8b1e4a0012a3f9c1 \
  -H "Content-Type: application/json" \
  -d "{\"title\":\"Build Backend API (Completed)\",\"completed\":true}"
```

**Expected Response (`200 OK`):**
```json
{
  "success": true,
  "message": "Task updated successfully",
  "data": {
    "_id": "665f1c2e8b1e4a0012a3f9c1",
    "title": "Build Backend API (Completed)",
    "description": "Implement CRUD endpoints with Express & MongoDB",
    "completed": true,
    "createdAt": "2026-09-19T16:25:00.000Z",
    "updatedAt": "2026-09-19T16:30:12.000Z",
    "__v": 0
  }
}
```

---

### Demo 6: Delete Task (DELETE)

Permanently removes the task document from the database.

- **Method**: `DELETE`
- **URL**: `http://localhost:7000/api/tasks/665f1c2e8b1e4a0012a3f9c1`

```bash
curl -X DELETE http://localhost:7000/api/tasks/665f1c2e8b1e4a0012a3f9c1
```

**Expected Response (`200 OK`):**
```json
{
  "success": true,
  "message": "Task deleted successfully",
  "data": {
    "_id": "665f1c2e8b1e4a0012a3f9c1",
    "title": "Build Backend API (Completed)",
    "description": "Implement CRUD endpoints with Express & MongoDB",
    "completed": true
  }
}
```

---

### Demo 7: Undefined Route 404

Attempts to query a route that does not exist.

- **Method**: `GET`
- **URL**: `http://localhost:7000/api/unknown-route`

```bash
curl -X GET http://localhost:7000/api/unknown-route
```

**Expected Response (`404 Not Found`):**
```json
{
  "success": false,
  "message": "Route not found: /api/unknown-route"
}
```

---

## 9. Error Handling Architecture

The application implements a centralized error handling pipeline via `middleware/errorHandler.js`. Any error triggered inside an async controller function is caught and passed to `next(error)`.

```text
Incoming Request
      │
      ▼
Controller (try / catch)
      │ (on error: next(error))
      ▼
errorHandler Middleware
      ├── ValidationError (Missing field) ───► 400 Bad Request
      ├── CastError (Malformed ObjectId)  ───► 400 Bad Request
      ├── Duplicate Key (Code 11000)      ───► 400 Bad Request
      └── Uncaught Exception              ───► 500 Server Error
```

This guarantees that:
- Raw server stack traces are logged internally but never leaked to client responses.
- Every client response maintains the `{ success: false, message }` JSON contract.

---

## 10. Key Engineering Highlights

1. **Atlas DNS Resolution (`config/db.js`)**:
   MongoDB Atlas uses `mongodb+srv://` connection strings which query DNS TXT records. Many local Windows DNS resolvers time out during this lookup. We explicitly bind Node's DNS engine to Cloudflare (`1.1.1.1`) and Google (`8.8.8.8`) to ensure 100% reliable cluster discovery.
2. **Defensive Validation (`taskController.js`)**:
   Instead of letting invalid IDs trigger database-level cast errors, `mongoose.Types.ObjectId.isValid(id)` checks every ID parameter proactively.
3. **Mongoose Schema Lifecycle**:
   Enabled `runValidators: true` on `findByIdAndUpdate` to ensure updates still adhere to validation rules.
4. **Environment Encapsulation**:
   All sensitive database credentials, usernames, passwords, and server ports are externalized into `.env`.

---

## 11. Verification Checklist

- [x] Node.js + Express.js + MongoDB + Mongoose stack
- [x] Clean modular directory structure (config, controllers, models, routes, middleware)
- [x] MongoDB Atlas connection via Mongoose using `.env`
- [x] `.env.example` template provided
- [x] Task model with `title`, `description`, `completed`, `createdAt`, `updatedAt`
- [x] `title` required & trimmed; `completed` defaults to `false`
- [x] Mongoose `timestamps: true`
- [x] `POST /api/tasks` — Create task
- [x] `GET /api/tasks` — Get all tasks (sorted newest first)
- [x] `GET /api/tasks/:id` — Get single task (400 for invalid ID, 404 for not found)
- [x] `PUT /api/tasks/:id` — Update task with schema validation
- [x] `DELETE /api/tasks/:id` — Delete task
- [x] Centralized error middleware handling validation errors, cast errors, and server faults
- [x] Proper HTTP status codes (`200`, `201`, `400`, `404`, `500`)
- [x] Consistent JSON envelope responses (`success`, `message`, `data`)
- [x] Fallback 404 handler for undefined routes
- [x] `npm start` and `npm run dev` scripts
- [x] Full cURL and Postman live demo test suite
