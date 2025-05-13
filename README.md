# Pet Adoption Platform API Design
## Introduction

This document outlines the API design and project structure for a Pet Adoption Platform. The design emphasizes clarity, maintainability, and scalability, drawing inspiration from SOLID principles and layered architecture patterns commonly seen in frameworks like Laravel, adapted for a Node.js/Express context. This is a "No Code" task focusing on design and analysis.

## 1. Project Structure & Conceptual Libraries

### Our proposed project structure promotes separation of concerns:

```
pet-adoption-api/
├── src/
│ ├── app.js # Express app setup, global middlewares
│ ├── server.js # Server initialization and startup
│ ├── config/ # Configuration files (e.g., database, environment variables loading)
│ ├── controllers/ # Request handlers, orchestrate service calls
│ │ ├── pet.controller.js
│ │ ├── clinic.controller.js
│ │ └── article.controller.js
│ ├── services/ # Business logic layer
│ │ ├── pet.service.js
│ │ ├── clinic.service.js
│ │ └── article.service.js
│ ├── middlewares/ # Custom middlewares
│ │ ├── auth.middleware.js # (Conceptual: for authentication if user roles are introduced)
│ │ ├── error.handler.js # Global error handler
│ │ └── validators/ # Request validation middlewares (akin to Form Requests)
│ │ ├── pet.validator.js
│ │ ├── clinic.validator.js
│ │ └── article.validator.js
│ ├── models/ # Data schemas/interfaces and Data Access Objects (DAOs)
│ │ ├── Pet.model.js # (e.g., Mongoose Schema & Model or Sequelize Model)
│ │ ├── Clinic.model.js
│ │ └── Article.model.js
│ ├── routes/ # API route definitions
│ │ ├── index.js # Main router to delegate to resource-specific routers
│ │ ├── pet.routes.js
│ │ ├── clinic.routes.js
│ │ └── article.routes.js
│ └── utils/ # Utility functions (e.g., date formatters, custom error classes)
│ ├── httpErrors.js # Custom HTTP error classes for standardized error responses
│ └── helpers.js # General helper functions
├── .env # Environment variables (e.g., DB_URI, PORT, JWT_SECRET)
├── .gitignore
├── package.json # Project dependencies and scripts
└── README.md # This design document
```

### Conceptual Libraries to be Used:

    Framework: Express.js (Minimalist and flexible Node.js web application framework).

    Database & ORM/ODM: (See section below: "Database Choice Justification" - Mongoose for MongoDB is proposed).

    Validation: A library like Joi or express-validator (For robust and declarative request data validation).

    Authentication (Future): jsonwebtoken (For implementing JWT-based authentication if user accounts are added).

    Utilities:

        dotenv: For loading environment variables from a .env file.

        uuid: For generating unique identifiers if not provided by the database (MongoDB ObjectIds are typically used).

        morgan: HTTP request logger middleware for Node.js.

        bcryptjs (Future): For hashing passwords if user authentication is implemented.

## 2. Database Choice Justification

Choosing the right database is crucial for performance, scalability, and development ease. For the Pet Adoption Platform, we will analyze the suitability of both SQL (specifically MySQL with Sequelize as ORM) and NoSQL (specifically MongoDB with Mongoose as ODM).

### Data Characteristics & Relationships:

    Entities: The core entities are Pets, Clinics, and Articles.

    Relationships:

        A pet is associated with one city (can be a simple string field or a reference if cities become managed entities).

        Articles can be tagged by pet type (e.g., 'dog', 'cat'). This could be an array of strings.

        Future considerations: Users might "favorite" pets (many-to-many), and adoption records would link users to pets.

    Data Structure:

        Pets, Clinics, and Articles have relatively well-defined schemas.

        Some fields are naturally arrays (e.g., services in Clinic, petTypeTags in Article).

        Clinic operatingHours and location (coordinates) are well-suited for nested object structures.

    Query Patterns:

        Filtering pets by city, type, adoption status.

        Searching clinics based on isOpenNow, isEmergency, city, or geo-location (proximity search).

        Filtering articles by petType and keywords.

#### Option 1: NoSQL (MongoDB with Mongoose)

    Pros:

        Schema Flexibility: MongoDB's document model excels at handling arrays, nested objects, and evolving data structures without requiring strict schema migrations for every minor change. This is highly beneficial for a startup where requirements can change rapidly.

        Scalability: MongoDB is designed for horizontal scalability (sharding), which is advantageous for applications expecting high traffic and data growth.

        Geo-spatial Queries: MongoDB has robust built-in support for geospatial indexing and queries, which is a key requirement for finding nearby clinics.

        Development Speed: For applications with document-centric data, Mongoose (an ODM for MongoDB) can lead to faster development cycles, especially for teams proficient in JavaScript/JSON.

    Cons:

        Transactions: While MongoDB supports multi-document ACID transactions, they can be more complex to reason about and manage for highly interdependent relational operations compared to traditional SQL databases.

        Complex Joins: Performing operations equivalent to complex SQL JOINs (e.g., for advanced analytics or reporting not typical for application-level queries) can be less direct, often relying on application-level joins or aggregation framework features.

#### Option 2: SQL (MySQL with Sequelize)

    Pros:

        ACID Transactions: MySQL (typically with the InnoDB storage engine) provides strong ACID compliance, ensuring high data consistency and reliability for transactional operations.

        Mature Relational Model: Ideal for highly structured data with well-defined relationships where data integrity enforced by schemas and foreign keys is paramount. MySQL is a widely adopted and well-understood RDBMS.

        Powerful Query Language (SQL): SQL is extremely powerful for complex queries, JOINs, and data aggregation.

    Cons:

        Schema Rigidity: Requires schema migrations (e.g., managed by Sequelize migrations) for any structural changes, which can add development overhead, especially in early stages with frequent iterations.

        Scalability: MySQL primarily scales vertically (increasing resources of a single server). While solutions like read replicas and clustering exist, implementing and managing horizontal sharding is generally more complex than with MongoDB.

        Handling Semi-structured Data: While newer versions of MySQL support a JSON data type, storing and querying arrays or deeply nested JSON-like structures can feel less native and might not offer the same performance or ease of use as MongoDB.

        Geospatial Support: MySQL has spatial extensions, but MongoDB's geospatial capabilities are often considered more comprehensive and developer-friendly for common location-based application queries.

### Recommendation & Justification:

For this Pet Adoption Platform, MongoDB (with Mongoose as ODM) is the recommended database.

    Justification:

        Geospatial Needs: The critical requirement to find nearby clinics (based on isOpenNow, isEmergency, and geographical proximity) is a strong fit for MongoDB's native and efficient geospatial indexing and query capabilities.

        Flexible Data Model & Rapid Iteration: The nature of data like clinic services (array), article tags (array), and potentially evolving pet attributes (e.g., array of medical notes) aligns well with MongoDB's flexible document model. This facilitates quicker iterations and adaptation to new features, which is vital for a startup.

        Development Agility: Mongoose and the document-centric approach often lead to faster initial development and easier integration for teams already working extensively with JavaScript and JSON.

        Scalability for Growth: MongoDB's architecture is inherently designed for horizontal scaling, providing a good path for handling future growth in user base and data volume.

        Simplicity for Core Relationships: The primary relationships in the current scope (e.g., pet to city, article to tags) can be effectively managed within a document model using embedding or simple references, without the immediate need for the full complexity of SQL JOINs for most common application queries.

Therefore, the src/models/ directory would contain Mongoose schemas and models (e.g., Pet.model.js, Clinic.model.js, Article.model.js). These models will serve as the Data Access Objects (DAOs) for interacting with the MongoDB database. 3. Conceptual Data Models (Mongoose-like Schemas)

#### Pet (Pet.model.js):

```js
    // Conceptual Mongoose Schema for Pet
    const petSchema = new Schema({
        name: { type: String, required: true, trim: true, index: true },
        type: { type: String, enum: ['dog', 'cat'], required: true, index: true },
        breed: { type: String, trim: true },
        age: { type: Number, min: 0 }, // In years or months, needs clarification
        gender: { type: String, enum: ['male', 'female'] },
        city: { type: String, required: true, trim: true, index: true },
        description: { type: String, trim: true },
        imageUrl: { type: String, trim: true }, // URL to the image
        isAdopted: { type: Boolean, default: false, index: true },
        // Additional fields: size, color, healthNotes (array of strings), etc.
    }, { timestamps: true }); // Adds createdAt and updatedAt automatically



 ```

#### Clinic (Clinic.model.js):

```js
// Conceptual Mongoose Schema for Clinic
const clinicSchema = new Schema(
  {
    name: { type: String, required: true, trim: true, index: true },
    address: { type: String, required: true, trim: true },
    city: { type: String, required: true, trim: true, index: true },
    phone: { type: String, trim: true },
    services: [String], // Array of services offered
    operatingHours: {
      // Example: { mon: "9am-5pm", tue: "9am-5pm", ... }
      type: Map,
      of: String,
    },
    isOpenNow: { type: Boolean, default: false }, // This might be dynamically calculated or updated by a job
    isEmergency: { type: Boolean, default: false, index: true },
    location: {
      // For geospatial queries
      type: { type: String, enum: ["Point"], default: "Point", required: true },
      coordinates: { type: [Number], required: true }, // [longitude, latitude]
    },
    // Additional fields: website, email, etc.
  },
  { timestamps: true }
);
clinicSchema.index({ location: "2dsphere" }); // Geospatial index for location-based searches
```

#### Article (Article.model.js):

```js
// Conceptual Mongoose Schema for Article
const articleSchema = new Schema(
  {
    title: { type: String, required: true, trim: true, index: true },
    content: { type: String, required: true }, // Could be Markdown or HTML
    author: { type: String, trim: true, default: "Platform Team" },
    petTypeTags: [
      { type: String, enum: ["dog", "cat", "general"], index: true },
    ],
    category: { type: String, trim: true, index: true }, // e.g., 'care', 'training', 'health'
    summary: { type: String, trim: true }, // Short summary for list views
    // Additional fields: views, likes, coverImageUrl, etc.
  },
  { timestamps: true }
); // `createdAt` can serve as publishedDate
```

## 4. API Endpoint Definitions

This section details the required APIs based on the user stories.
### User Story 1: "As a user, I want to see adoptable pets in my city."

#### API 1.1: List Adoptable Pets

Method: GET

Endpoint: /api/pets

Description: Retrieves a paginated list of pets available for adoption, filterable by city, type, and adoption status.

Request Details:

Query Params:

    city: string (Required) - e.g., "Nablus". Validation: pet.validator.js will check for presence.

    type: string (Optional, enum: 'dog', 'cat') - e.g., "dog". Validation: pet.validator.js.

    isAdopted: boolean (Optional, default: false).

    page: number (Optional, default: 1, min: 1).

    limit: number (Optional, default: 10, min: 1, max: 100).

##### Successful Response (200 OK):

```json
        {
          "message": "Pets retrieved successfully.",
          "data": [
            {
              "id": "60c72b2f9b1e8a001c8e4d8e", // MongoDB ObjectId
              "name": "Buddy",
              "type": "dog",
              "breed": "Golden Retriever",
              "age": 2,
              "city": "Nablus",
              "imageUrl": "http://example.com/images/buddy.jpg",
              "isAdopted": false,
              "createdAt": "2023-01-15T10:00:00.000Z"
            }
            // ... more pets
          ],
          "pagination": {
            "currentPage": 1,
            "totalPages": 5,
            "totalItems": 50,
            "limit": 10
          }
        }



 ```

##### Potential Errors & Responses:

400 Bad Request: If city is missing or other query params are invalid (e.g., non-enum type, non-numeric page). Handled by pet.validator.js via error.handler.js.

```json
    { "status": "error", "statusCode": 400, "message": "Query parameter 'city' is required." }



```

404 Not Found: If no pets match the criteria. The service layer might return an empty array, and the controller will still send a 200 OK with empty data, or a specific 404 if preferred. (Convention: usually 200 OK with empty data for list endpoints).

500 Internal Server Error: For unexpected server issues. Handled by error.handler.js.

#### API 1.2: Get Specific Pet Details

Method: GET

Endpoint: /api/pets/:id

Description: Retrieves detailed information for a specific pet by its ID.

Request Details:

Params: id: string (MongoDB ObjectId). Validation: ID format check in pet.validator.js.

 ##### Successful Response (200 OK):

```json
    {
      "message": "Pet details retrieved successfully.",
      "data": {
        "id": "60c72b2f9b1e8a001c8e4d8e",
        "name": "Buddy",
        "type": "dog",
        "breed": "Golden Retriever",
        "age": 2,
        "gender": "male",
        "city": "Nablus",
        "description": "A very friendly and playful dog, loves walks and cuddles.",
        "imageUrl": "http://example.com/images/buddy.jpg",
        "isAdopted": false,
        "createdAt": "2023-01-15T10:00:00.000Z",
        "updatedAt": "2023-01-16T12:30:00.000Z"
      }
    }



```

##### Potential Errors & Responses:

400 Bad Request: If id parameter is not a valid MongoDB ObjectId format. Handled by pet.validator.js.

404 Not Found: If no pet with the given ID exists. Service layer throws NotFoundError, handled by error.handler.js.

```json
    { "status": "error", "statusCode": 404, "message": "Pet with ID '...' not found." }
```

500 Internal Server Error.

### User Story 2: "As a user, I need to find currently open emergency vet clinics."

#### API 2.1: List Vet Clinics

    Method: GET

    Endpoint: /api/clinics

    Description: Retrieves a list of vet clinics, filterable by current open status (conceptual), emergency services, and city or proximity.

    Request Details:

        Query Params:

            isOpenNow: boolean (Required, value must be true for this user story). Validation: clinic.validator.js.

            isEmergency: boolean (Required, value must be true for this user story). Validation: clinic.validator.js.

            city: string (Optional, if not providing lat/lng) - e.g., "Ramallah".

            lat: number (Optional, for geo-search, requires lng).

            lng: number (Optional, for geo-search, requires lat).

            radius: number (Optional, in kilometers, for geo-search, default e.g., 10km).

##### Successful Response (200 OK):

```json
        {
          "message": "Clinics retrieved successfully.",
          "data": [
            {
              "id": "60c72b2f9b1e8a001c8e4d9f",
              "name": "City Vet Emergency Care",
              "address": "123 Main St, Ramallah",
              "phone": "02-295-1234",
              "isOpenNow": true, // This value would ideally be dynamic or frequently updated
              "isEmergency": true,
              "location": { "type": "Point", "coordinates": [35.20, 31.90] }, // [lng, lat]
              "operatingHours": { "mon": "24/7 Emergency", "tue": "24/7 Emergency" }
            }
            // ... more clinics
          ]
        }



```

##### Potential Errors & Responses:

    400 Bad Request: If isOpenNow or isEmergency are missing or not true for this specific search. Invalid geo-params. Handled by clinic.validator.js.

    404 Not Found: If no clinics match the criteria. (Usually 200 OK with empty data for list endpoints).

    500 Internal Server Error.

### User Story 3: "As a user, I want to search for articles by pet type."

#### API 3.1: List Articles

    Method: GET

    Endpoint: /api/articles

    Description: Retrieves a paginated list of articles, filterable by pet type tags, category, and keywords.

    Request Details:

        Query Params:

            petType: string (Required, enum: 'dog', 'cat', 'general') - e.g., "dog". Validation: article.validator.js.

            category: string (Optional) - e.g., "health".

            keyword: string (Optional, for searching in title/summary) - e.g., "training".

            page: number (Optional, default: 1).

            limit: number (Optional, default: 10).

##### Successful Response (200 OK):

```json
        {
          "message": "Articles retrieved successfully.",
          "data": [
            {
              "id": "60c72b2f9b1e8a001c8e4e0a",
              "title": "Top 5 Training Tips for Your New Dog",
              "author": "Platform Team",
              "petTypeTags": ["dog", "general"],
              "category": "training",
              "summary": "Essential training advice for new dog owners to ensure a happy companion.",
              "createdAt": "2023-03-10T09:00:00.000Z"
            }
            // ... more articles
          ],
          "pagination": { /* ... pagination details ... */ }
        }



 ```

##### Potential Errors & Responses:

    400 Bad Request: If petType is missing or invalid. Handled by article.validator.js.

    404 Not Found: (Usually 200 OK with empty data for list endpoints).

    500 Internal Server Error.

#### API 3.2: Get Specific Article Content

    Method: GET

    Endpoint: /api/articles/:id

    Description: Retrieves the full content of a specific article by its ID.

    Request Details:

        Params: id: string (MongoDB ObjectId). Validation: ID format check in article.validator.js.

##### Successful Response (200 OK):

```json
    {
      "message": "Article content retrieved successfully.",
      "data": {
        "id": "60c72b2f9b1e8a001c8e4e0a",
        "title": "Top 5 Training Tips for Your New Dog",
        "content": "<p>Full article content formatted in HTML or Markdown goes here...</p>",
        "author": "Platform Team",
        "petTypeTags": ["dog", "general"],
        "category": "training",
        "createdAt": "2023-03-10T09:00:00.000Z",
        "updatedAt": "2023-03-11T14:20:00.000Z"
      }
    }
```

##### Potential Errors & Responses:

    400 Bad Request: Invalid article ID format.

    404 Not Found: Article with the given ID not found. Service throws NotFoundError.

    500 Internal Server Error.

## 5. Adherence to SOLID Principles (Conceptual)

The proposed architecture aims to adhere to SOLID principles:

Single Responsibility Principle (SRP):

    Controllers: Responsible for handling the HTTP request/response cycle, input sanitation (delegated to validators), and invoking appropriate service methods. They do not contain business logic.

    Services: Encapsulate all business logic related to a specific domain (e.g., PetService handles all operations for pets). They interact with models for data persistence.

    Models (Mongoose): Define the data schema and provide an abstraction layer for database interactions (CRUD operations, queries).

    Validators (*.validator.js): Solely responsible for validating incoming request data against predefined rules.

    Error Handler Middleware: Centralized place for handling all errors and formatting consistent error responses.

Open/Closed Principle (OCP): The system is open for extension (e.g., adding new features, routes, or services) but closed for modification of existing, stable components. Middleware architecture (validators, error handlers) naturally supports this.

Liskov Substitution Principle (LSP): More relevant with class inheritance, but conceptually, different service implementations (if we were to use interfaces) should be substitutable.

Interface Segregation Principle (ISP): Clients (e.g., controllers) should not be forced to depend on methods they do not use. Services should expose focused interfaces.

Dependency Inversion Principle (DIP): High-level modules (Controllers) depend on abstractions (Services), and Services depend on abstractions (Models provided by ORM/ODM). This promotes loose coupling and testability. Actual dependency injection can be achieved manually or with DI containers.

## 6. Future Considerations (Out of Scope for this Task, but good for design thinking)

   User Authentication & Authorization: Implementing user accounts (adopters, clinic owners, admins) with JWTs or sessions. Role-based access control (RBAC).

   Admin Panel: APIs for admins to manage pets, clinics, articles, and users. This would involve POST, PUT, DELETE endpoints for these resources.

   Image Uploads: A robust solution for uploading and serving images for pets and articles.

   Adoption Process Workflow: APIs to manage adoption applications, status updates, etc.

   User Profiles & Favorites: Allowing users to save favorite pets or search criteria.

   Reviews and Ratings: For clinics or even pets (post-adoption).

   Real-time Notifications: (e.g., new pet matching saved search, adoption status updates).

   Advanced Search & Filtering: More complex filtering options, full-text search capabilities (MongoDB Atlas Search).

   Testing Strategy: Defining unit, integration, and end-to-end testing approaches.

   Deployment & CI/CD: Considerations for deploying the application and setting up a CI/CD pipeline.
