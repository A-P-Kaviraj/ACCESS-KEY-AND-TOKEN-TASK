  Access Key and Token Information Management Services

Access Key and Token Information Management Services
====================================================

This repository contains two microservices: Access Key Management Service and Token Information Management Service. These services work together to handle access key generation, rate limiting, and token information retrieval using RabbitMQ for communication.

Project Structure
-----------------

This repository includes the following submodules:

*   Access Key Management Service
*   Token Information Management Service

To initialize the submodules, use the following command:

    git submodule update --init --recursive

Overview
--------

*   **Access Key Management Service**: Manages access keys, including key generation, rate limiting, TTL (Time-to-Live), and key validation. It communicates with the Token Information Service to verify access keys.
*   **Token Information Management Service**: Provides token information (Web3 tokens) and verifies the access key provided by the user. It uses RabbitMQ to communicate with the Access Key Management Service for access key validation and throttling.

Getting Started
---------------

### Prerequisites

Ensure you have the following installed:

*   Node.js
*   Docker (for RabbitMQ)
*   RabbitMQ (running locally or in a container)

### Setting Up RabbitMQ

You can start RabbitMQ using Docker with the following command:

    docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management

This will expose RabbitMQ on port 5672 and the management interface on 15672. You can access the management dashboard at [http://localhost:15672/](http://localhost:15672/) with default credentials (guest/guest).

Installation
------------

1.  **Clone the Repository**  
    Clone the main repository that includes submodules:

    ```
    git clone --recurse-submodules https://github.com/A-P-Kaviraj/Access-Key-And-Token-Information-Management-Services.git
    ```
    ```
    cd Access-Key-And-Token-Information-Management-Services
    ```

1.  **Install Dependencies**  
    Both services have their own dependencies. You need to navigate into each submodule and install dependencies.

**Access Key Management Service:**

    cd access-key-management-service
    npm install
    

**Token Information Management Service:**
  
    cd ../token-information-management-service
    npm install
  

3.  **Prisma Setup for Access Key Management Service**  
    For the Access Key Management Service, you need to configure Prisma.

Set up the database connection in `prisma/schema.prisma` file (e.g., SQLite or PostgreSQL):

    datasource db {
      provider = "postgresql"
      url      = env("DATABASE_URL")
    }

Migrate the database:

    npx prisma migrate dev

Running the Services
--------------------

1.  **Access Key Management Service**  
    Start the Access Key Management Service:

    ```
    cd access-key-management-service
    ```
    ```
    npm run start
    ```
It will run on [http://localhost:3001/](http://localhost:3001/) and listen for RabbitMQ messages on the access-key-queue.

2.  **Token Information Management Service**  
    Start the Token Information Management Service:

    ```
    cd ../token-information-management-service
    ```
    ```
    npm run start
    ```

It will run on [http://localhost:3002/](http://localhost:3002/) and also listen to RabbitMQ messages for access key validation.

How the Services Work
---------------------

### Access Key Management Service

This service is responsible for:

*   Generating unique access keys.
*   Storing keys with their rate limits and TTL.
*   Validating access keys received from the Token Information Management Service.
*   Providing a REST API for managing access keys.

### Endpoints

**Headers:** All requests must include the following header:

    Content-Type: application/json

**Example JSON for Generating Keys:**

    {
      "rateLimit": 100,
      "ttl": 3600
    }

*   **POST /keys:** Create a new access key.

    POST http://localhost:3001/keys

Request Body (JSON):

    {
      "rateLimit": 100,
      "ttl": 3600
    }

*   **GET /keys:** List all access keys.

    GET http://localhost:3001/keys

*   **GET /keys/:key:** Get a specific access key by its value.

    GET http://localhost:3001/keys/:key

*   **PUT /keys/:id:** Update an access key's rate limit or TTL.

    PUT http://localhost:3001/keys/:id

Request Body (JSON):

    {
      "rateLimit": 200,
      "ttl": 7200
    }

*   **DELETE /keys/:id:** Delete an access key.

    DELETE http://localhost:3001/keys/:id


### Token Information Management Service

This service is responsible for:

*   Providing Web3 token information (currently static data).
*   Verifying the access key provided by the user by communicating with the Access Key Management Service via RabbitMQ.
*   Implements rate limiting and TTL checks to ensure that the access key is valid and within its usage limits.

### Endpoints

*   **GET /token-info:** Returns Web3 token information after verifying the access key.

    GET http://localhost:3002/token-info

**Headers:**

    Content-Type: application/json
    x-access-key: {your-access-key}

This service uses a guard (`AccessGuard`) that intercepts requests and validates the provided access key against the Access Key Management Service through RabbitMQ.

Dependencies
------------

### Common Dependencies

*   Node.js: Runtime environment for the services.
*   RabbitMQ: Message broker for communication between services.
*   Docker: For containerizing RabbitMQ.

### Service-Specific Dependencies

**Access Key Management Service**

*   NestJS: Backend framework.
*   Prisma: ORM for database management.
*   @nestjs/microservices: For RabbitMQ communication.
*   PostgreSQL: As the database (or any other supported by Prisma).

**Token Information Management Service**

*   NestJS: Backend framework.
*   @nestjs/microservices: For RabbitMQ communication.
*   `firstValueFrom`: For handling observables in NestJS.

Notes
-----

*   Ensure that RabbitMQ is running before starting the services.
*   Both services communicate via RabbitMQ queues for validating access keys.
*   The Access Key Management Service is responsible for the lifecycle and limits of access keys, while the Token Information Service only provides token data after validating keys.
