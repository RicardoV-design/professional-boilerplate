# Professional API Boilerplate

## Project Description

It's a production-ready backend boilerplate built with Node.js, Express, TypeScript, and TypeORM. This template is designed for the development of scalable RESTful APIs by providing a pre-configured architecture featuring strict request validation, centralized error management, structured logging, and database migration workflows out of the box.

## Features List

* **TypeScript & `tsx` Execution:** Native ECMAScript module support and fast runtime execution with hot-reloading capabilities.

* **TypeORM Integration:** Full database management with MySQL, including explicit entity typing and automated schema generation.

* **Centralized Error Handling:** Global error catching that formats and standardizes HTTP responses, preventing server crashes and duplicate try-catch blocks.

* **Strict Payload Validation:** Joi middleware to sanitize and validate incoming `body`, `query`, and `params` data.

* **Structured Logging:** Winston integration with customized transports for combined request tracking and isolated error logs stored in JSON format.

* **Database Version Control:** Configured TypeORM CLI for generating, running, and reverting SQL migrations, plus seed execution.

* **Security & Performance:** Integrated standard middleware including Helmet, CORS, and Express rate limiting.

## Getting Started

### 1. Setup Instructions

Clone the repository and navigate into the project directory, then follow these steps:

Install the required project dependencies:

```bash
npm install
```

Copy the environment variables template to create your local config:

```bash
cp .env.example .env
```

Update the `.env` file with your local MySQL credentials. Then, generate and execute the initial database schema:

```bash
npm run migration:generate -- src/migrations/InitialSchema
npm run migration:run
```

Start the development server with hot-reloading:

```bash
npm run dev
```

### 2. Environment Variables Documentation

The following environment variables are required in your `.env` file to run the application:

* `PORT`: The port number the Express server will listen on (default: `3000`).
* `NODE_ENV`: The current deployment environment (`development`, `production`, `test`).
* `DB_HOST`: The hostname or IP address of your MySQL database server.
* `DB_PORT`: The port number for your MySQL database (default: `3306`).
* `DB_USERNAME`: The authorized username for your MySQL database.
* `DB_PASSWORD`: The password for your MySQL database user.
* `DB_DATABASE`: The specific name of the MySQL database schema to connect to.

## API Endpoints Documentation

### General

* **`GET /health`**
  * **Description:** Health check endpoint to verify the server is active.
  * **Response:** `{ "status": "ok", "timestamp": "..." }`

### Users

* **`GET /api/users`**
  * **Description:** Retrieves a paginated list of users.
  * **Query Params:** `page`, `limit`, `sort`, `order`

* **`GET /api/users/:id`**
  * **Description:** Retrieves a single user's details by their ID.

* **`POST /api/users`**
  * **Description:** Creates a new user. Requires `name`, `email`, and `password` in the request body.

* **`PUT /api/users/:id`**
  * **Description:** Updates an existing user's information.

* **`DELETE /api/users/:id`**
  * **Description:** Soft or hard deletes a user by their ID.

## Database Management (Migrations & Seeders)

The project uses `tsx` to safely execute TypeORM CLI commands and internal scripts without Node.js file-existence errors.

**Generate a new migration (after modifying entities):**

```bash
npm run migration:generate -- src/migrations/YourMigrationName
```

**Apply pending migrations to the database:**

```bash
npm run migration:run
```

**Revert the last applied migration:**

```bash
npm run migration:revert
```

**Populate the database with initial seed data:**

```bash
npm run seed
```

## Project Reflection

**1. What was the most challenging part of building this boilerplate? How did you overcome it?**

- The most challenging part was executing the initial schema migration, as I encountered errors while installing dependencies like `reflect-metadata`, `mysql2`, `joi`, and `winston`, and while making a few modifications to the code. I overcame this by carefully checking the error messages and tracing the source of the problem.

**2. How does this boilerplate differ from your Midterm project? What specific improvements did you make?**

- I find this as a template for making projects including the User only for the entity and there are certain codes/dependencies that can run in just a single command.

**3. Why is centralized error handling better than try-catch in every controller?**

- It's where all of the errors will be validated and much better if it will be centralized so that each failed request or problem will be handled together in 1 file. Also, it's much cleaner.

**4. How does request validation with Joi improve security and code quality?**

- Request validation with Joi acts as a strict perimeter gateway for an API, filtering out malformed or malicious payloads before they ever reach the business logic or database layer.

**5. What's the difference between `synchronize: true` and migrations? Why are migrations essential for production?**

- The `synchronize: true` setting is strictly for development because it automatically alters tables, columns, and database schemas every time the server restarts to perfectly match current code entities, which can aggressively drop and replace elements resulting in severe data loss. In contrast, migrations are explicitly generated, sequential SQL scripts used for making incremental changes to the tables in a controlled and version-tracked manner without destroying existing data.

Migrations are essential for production because they safely preserve live database records while altering structures and act as a permanent, ordered ledger to ensure your local environment, staging server, and production server always share the exact same schema state.

**6. How does structured logging (Winston) help in production debugging compared to console.log?**

- Structured logging with Winston outputs log entries as machine-readable data structures, typically JSON, whereas standard `console.log` only outputs raw, unformatted text strings.

**7. If you had to add a new feature (e.g., Product CRUD), how would you use this boilerplate? What would you need to add?**

- Depends on the project I am making. To give an example, if I need to add categories, services or products I need to add new entity, controllers, routing following the same flow.

**8. What additional features would you add to make this boilerplate even more production-ready?**

- Implementing Role-Based Access Control (RBAC) alongside short-lived JWTs and HTTP-only refresh tokens establishes a strict permissions matrix, which is critical for workflows. Also, transitioning from manual EchoAPI checks to a comprehensive automated testing suite using Postman, guarantees that future code updates will not silently break your validation middleware or TypeORM queries.