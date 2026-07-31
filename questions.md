# FastAPI Interview Questions & Answers (Part 1)
## FastAPI Fundamentals, ASGI, Uvicorn, Routing, HTTP Methods, Path Parameters & Query Parameters

---

# 1. What is FastAPI?

### Answer

FastAPI is a modern, high-performance Python web framework used to build RESTful APIs. It provides automatic request validation, interactive API documentation, asynchronous support, and excellent performance, making it ideal for production applications and machine learning model serving.

---

# 2. Why is FastAPI popular in MLOps?

### Answer

FastAPI is widely used in MLOps because it allows trained machine learning models to be exposed as REST APIs. It also provides:

- Automatic request validation
- Interactive Swagger documentation
- Asynchronous request handling
- High performance
- Easy integration with Docker and Kubernetes

---

# 3. What are the advantages of FastAPI over Flask?

### Answer

Compared to Flask, FastAPI offers:

- Automatic request validation
- Automatic API documentation
- Async support
- Better performance
- Built-in type checking using Python type hints
- Cleaner development experience

---

# 4. What is FastAPI mainly used for in MLOps?

### Answer

FastAPI is primarily used to expose trained machine learning models as REST APIs so that external applications can send data and receive predictions over HTTP.

Example:

Client

↓

POST /predict

↓

FastAPI

↓

Machine Learning Model

↓

Prediction

---

# 5. Is FastAPI a web server?

### Answer

No.

FastAPI is a web framework.

It defines API routes and business logic but does not listen for incoming HTTP requests itself.

A separate ASGI server such as Uvicorn is required to run a FastAPI application.

---

# 6. What is REST API?

### Answer

A REST API is an interface that allows applications to communicate over HTTP using standard methods such as GET, POST, PUT, and DELETE.

FastAPI is commonly used to build REST APIs.

---

# 7. Why is FastAPI considered high performance?

### Answer

FastAPI achieves high performance because:

- It uses ASGI instead of WSGI.
- It supports asynchronous programming.
- It is built on Starlette.
- It uses efficient data validation through Pydantic.

---

# 8. What does FastAPI automatically generate?

### Answer

FastAPI automatically generates:

- Swagger UI documentation (/docs)
- ReDoc documentation (/redoc)
- OpenAPI specification
- Request validation
- Response validation

---

# 9. What is asynchronous programming?

### Answer

Asynchronous programming allows a server to handle multiple requests concurrently without waiting for one request to finish before processing another.

This improves scalability for I/O-bound operations such as database queries and API calls.

---

# 10. Why is FastAPI suitable for production?

### Answer

FastAPI is production-ready because it provides:

- Automatic validation
- Automatic documentation
- Async support
- Excellent performance
- Strong typing
- Easy deployment with Docker and Kubernetes

---

# ASGI & UVICORN

---

# 11. What is ASGI?

### Answer

ASGI (Asynchronous Server Gateway Interface) is a standard interface between asynchronous Python web applications and web servers.

It enables concurrent request handling and asynchronous programming.

---

# 12. Why does FastAPI use ASGI instead of WSGI?

### Answer

FastAPI uses ASGI because it supports asynchronous request handling, allowing multiple requests to be processed concurrently.

This improves scalability and performance for modern web applications.

---

# 13. What is WSGI?

### Answer

WSGI (Web Server Gateway Interface) is a standard interface used by traditional synchronous Python web frameworks such as Flask and Django.

It processes requests synchronously.

---

# 14. Difference between ASGI and WSGI?

| ASGI | WSGI |
|------|------|
| Asynchronous | Synchronous |
| Handles concurrent requests | One request at a time |
| Used by FastAPI | Used by Flask |
| Supports WebSockets | No WebSocket support |
| Better for modern APIs | Better for traditional web apps |

---

# 15. What is Uvicorn?

### Answer

Uvicorn is a lightweight, high-performance ASGI server used to run FastAPI applications.

It receives HTTP requests, passes them to FastAPI, and returns responses to clients.

---

# 16. Why do we need Uvicorn?

### Answer

FastAPI cannot directly receive network requests.

Uvicorn:

- Opens a network port
- Accepts HTTP requests
- Sends requests to FastAPI
- Returns responses

Without Uvicorn (or another ASGI server), FastAPI cannot serve requests.

---

# 17. Can FastAPI run without Uvicorn?

### Answer

No.

FastAPI requires an ASGI server such as:

- Uvicorn
- Hypercorn
- Daphne

to receive HTTP requests.

---

# 18. Explain the FastAPI request flow.

### Answer

Client

↓

HTTP Request

↓

Uvicorn

↓

FastAPI

↓

Business Logic / ML Model

↓

Response

↓

Client

---

# 19. What is an ASGI server?

### Answer

An ASGI server implements the ASGI protocol and runs asynchronous Python applications.

Examples include:

- Uvicorn
- Hypercorn
- Daphne

---

# 20. Give a real-world analogy for Uvicorn.

### Answer

FastAPI is the chef.

Uvicorn is the waiter.

The waiter receives customer orders and delivers them to the chef.

Without the waiter, customers cannot interact with the chef.

---

# ROUTING

---

# 21. What is a route?

### Answer

A route is a URL endpoint that maps an incoming HTTP request to a Python function responsible for processing the request and returning a response.

---

# 22. Why do we need routing?

### Answer

Routing directs incoming requests to the appropriate function based on the requested URL and HTTP method.

It allows different endpoints to perform different operations.

---

# 23. Give common routes in an MLOps API.

### Answer

- /predict
- /health
- /metrics
- /train
- /docs

---

# 24. What happens when a client calls /predict?

### Answer

Client

↓

POST /predict

↓

Route matches

↓

Prediction function executes

↓

Machine learning model predicts

↓

JSON response returned

---

# 25. Can multiple routes exist in one FastAPI application?

### Answer

Yes.

A single FastAPI application commonly contains many routes such as:

- /predict
- /metrics
- /health
- /train
- /upload

---

# HTTP METHODS

---

# 26. What is GET?

### Answer

GET is an HTTP method used to retrieve data from the server without modifying its state.

Examples:

- GET /health
- GET /metrics
- GET /docs

---

# 27. What is POST?

### Answer

POST is an HTTP method used to send data to the server for processing or creating a resource.

Examples:

- POST /predict
- POST /train
- POST /upload

---

# 28. Difference between GET and POST?

| GET | POST |
|------|------|
| Retrieve data | Send data |
| Usually no request body | Usually contains request body |
| Does not modify server state | Processes or creates resources |
| Safe operation | Processing operation |

---

# 29. Why is /predict implemented using POST?

### Answer

Because prediction requires sending input features to the server.

POST allows structured request bodies containing model input data.

---

# 30. Which HTTP method should be used?

| Endpoint | Method |
|-----------|--------|
| /health | GET |
| /metrics | GET |
| /predict | POST |
| /upload | POST |

---

# PATH PARAMETERS

---

# 31. What is a path parameter?

### Answer

A path parameter is a variable embedded in the URL path used to identify a specific resource.

Example:

/models/5

Here,

5

is the path parameter.

---

# 32. Give examples of path parameters.

### Answer

/users/10

/models/4

/customer/101

/files/15

---

# 33. Why are path parameters usually required?

### Answer

Because they identify the exact resource requested by the client.

Without them, the server cannot determine which resource should be returned.

---

# QUERY PARAMETERS

---

# 34. What is a query parameter?

### Answer

A query parameter is an optional key-value pair added after the ? symbol in the URL.

It is commonly used for:

- Filtering
- Searching
- Sorting
- Pagination

---

# 35. Give examples of query parameters.

### Answer

/users?page=2

/products?category=laptop

/models?version=3

/predictions?limit=20

---

# 36. Difference between path parameters and query parameters?

| Path Parameter | Query Parameter |
|----------------|-----------------|
| Inside URL path | After ? |
| Usually required | Usually optional |
| Identifies a resource | Filters or modifies response |

---

# 37. When should you use path parameters?

### Answer

Use path parameters when identifying a specific resource.

Examples:

- /users/15
- /models/2

---

# 38. When should you use query parameters?

### Answer

Use query parameters when filtering, searching, sorting, or paginating results.

Examples:

- /users?page=3
- /products?category=phone

---

# 39. Which is a path parameter?

```
/customer/100
```

### Answer

100

---

# 40. Which is a query parameter?

```
/customer?page=2
```

### Answer

page=2

---

# Quick Revision

✔ FastAPI is a high-performance Python API framework.

✔ FastAPI requires an ASGI server such as Uvicorn.

✔ ASGI supports asynchronous request handling.

✔ Uvicorn receives HTTP requests and runs FastAPI.

✔ Routes map URLs to Python functions.

✔ GET retrieves data.

✔ POST sends data for processing.

✔ Path parameters identify specific resources.

✔ Query parameters filter or modify responses.

# FastAPI Interview Questions & Answers (Part 2)
## Pydantic, Request Models, Response Models, Validation & Serialization

---

# PYDANTIC

---

# 41. What is Pydantic?

### Answer

Pydantic is a Python data validation library used by FastAPI to validate, parse, and serialize request and response data using Python type hints.

It ensures that data follows the expected structure before entering or leaving the application.

---

# 42. Why does FastAPI use Pydantic?

### Answer

FastAPI uses Pydantic because it automatically:

- Validates incoming requests
- Converts compatible data types
- Validates outgoing responses
- Generates API documentation
- Prevents invalid data from reaching application logic

---

# 43. What problems does Pydantic solve?

### Answer

Without Pydantic:

- Invalid data may crash the application.
- Developers manually validate every request.
- Documentation becomes difficult to maintain.

With Pydantic:

- Validation is automatic.
- Code becomes cleaner.
- APIs become more reliable.

---

# 44. Why is Pydantic important in MLOps?

### Answer

Machine learning models expect input features to have the same schema and data types used during training.

Pydantic ensures:

- Required features are present.
- Data types are correct.
- Invalid requests are rejected.
- Models receive clean and consistent input.

---

# 45. Does Pydantic only validate requests?

### Answer

No.

Pydantic validates both:

- Incoming request data
- Outgoing response data

---

# 46. What is BaseModel?

### Answer

BaseModel is the parent class provided by Pydantic.

Every request model and response model typically inherits from BaseModel.

Example:

```python
class Customer(BaseModel):
    age: int
    salary: float
```

---

# 47. What is automatic type conversion?

### Answer

Pydantic automatically converts compatible data types.

Example:

Client sends:

```json
{
    "age":"25"
}
```

Pydantic converts:

```
"25"
```

↓

```
25
```

The request succeeds.

---

# 48. Will Pydantic reject every string sent for an integer?

### Answer

No.

Compatible strings such as:

```
"25"
```

are automatically converted to integers.

Only incompatible values such as:

```
"hello"
```

are rejected.

---

# 49. What happens if conversion is impossible?

### Answer

FastAPI immediately returns:

```
422 Unprocessable Entity
```

The request never reaches the application logic.

---

# 50. Give examples of values that fail validation.

### Answer

Expected:

```
age: int
```

Invalid:

```
age = "hello"
```

Expected:

```
salary: float
```

Invalid:

```
salary = "abc"
```

---

# REQUEST MODELS

---

# 51. What is a Request Model?

### Answer

A Request Model is a Pydantic model that defines the structure, data types, and validation rules for incoming request data.

---

# 52. Why do we use Request Models?

### Answer

Request Models provide:

- Automatic validation
- Type checking
- Cleaner code
- Better documentation
- Consistent request structure

---

# 53. Why not manually parse JSON?

### Answer

Instead of writing:

```python
age=request["age"]
salary=request["salary"]
balance=request["balance"]
```

FastAPI automatically parses and validates the request using a Pydantic model.

This reduces boilerplate code and prevents many runtime errors.

---

# 54. What information does a Request Model contain?

### Answer

A Request Model defines:

- Field names
- Data types
- Required fields
- Optional fields
- Validation rules

---

# 55. Give an MLOps example of a Request Model.

### Answer

Suppose a churn prediction model requires:

- Age
- Salary
- Balance
- Credit Score

The Request Model ensures all these features are present and correctly typed before prediction begins.

---

# 56. When is a Request Model validated?

### Answer

Before the endpoint function executes.

If validation fails, FastAPI returns an error immediately.

---

# RESPONSE MODELS

---

# 57. What is a Response Model?

### Answer

A Response Model is a Pydantic model that defines and validates the structure of the API response before it is sent to the client.

---

# 58. Why do we use Response Models?

### Answer

Response Models provide:

- Consistent API responses
- Output validation
- Automatic documentation
- Type safety

---

# 59. Why are Response Models useful in production?

### Answer

They ensure every client receives responses in a predictable format regardless of internal implementation changes.

---

# 60. Give an example of a Response Model.

### Answer

Prediction API

Response:

```json
{
    "prediction":"Churn"
}
```

The Response Model guarantees this structure for every successful prediction.

---

# 61. Difference between Request Model and Response Model?

| Request Model | Response Model |
|--------------|----------------|
| Validates input | Validates output |
| Client → API | API → Client |
| Before business logic | After business logic |

---

# VALIDATION

---

# 62. What is validation?

### Answer

Validation is the process of checking whether incoming data matches the expected schema before processing it.

---

# 63. Why is validation important?

### Answer

Validation:

- Prevents application crashes
- Protects ML models
- Ensures consistent data
- Improves API reliability
- Reduces debugging effort

---

# 64. What happens if a required field is missing?

### Answer

FastAPI rejects the request and returns:

```
422 Unprocessable Entity
```

---

# 65. What happens if the wrong datatype is provided?

### Answer

If automatic conversion is impossible,

FastAPI rejects the request with:

```
422 Unprocessable Entity
```

---

# 66. What happens if the client sends:

```json
{
    "age":"25"
}
```

Expected:

```
age:int
```

### Answer

Pydantic converts:

```
"25"
```

↓

```
25
```

The request succeeds.

---

# 67. What happens if the client sends:

```json
{
    "age":"hello"
}
```

Expected:

```
age:int
```

### Answer

Validation fails.

FastAPI returns:

```
422 Unprocessable Entity
```

---

# 68. What happens if salary is missing?

Expected:

```
age:int
salary:float
```

Received:

```json
{
    "age":25
}
```

### Answer

Validation fails because salary is required.

FastAPI returns:

```
422 Unprocessable Entity
```

---

# 69. Does validation happen before or after prediction?

### Answer

Before.

Invalid requests never reach the machine learning model.

---

# 70. Why is validation especially important for machine learning?

### Answer

ML models are trained on a specific feature schema.

Incorrect data types or missing features may:

- Produce incorrect predictions
- Cause runtime errors
- Reduce model reliability

Validation prevents these problems.

---

# SERIALIZATION

---

# 71. What is serialization?

### Answer

Serialization is the process of converting Python objects into a format suitable for transmission, such as JSON.

---

# 72. Why is serialization needed?

### Answer

Browsers and APIs communicate using JSON, not Python objects.

Serialization converts Python data into JSON before sending it to clients.

---

# 73. What is deserialization?

### Answer

Deserialization is the process of converting incoming JSON into Python objects.

---

# 74. When does deserialization occur?

### Answer

During request processing.

Client

↓

JSON

↓

Python object

↓

Request Model

---

# 75. When does serialization occur?

### Answer

During response processing.

Python object

↓

JSON

↓

Client

---

# 76. Who performs serialization and deserialization in FastAPI?

### Answer

FastAPI performs these operations automatically using Pydantic.

---

# ERROR HANDLING DURING VALIDATION

---

# 77. What HTTP status code is returned when validation fails?

### Answer

```
422 Unprocessable Entity
```

---

# 78. Why is 422 returned instead of 500?

### Answer

Because the request itself is invalid.

The application has not crashed.

The client must correct the input before retrying.

---

# 79. Does invalid input reach the endpoint function?

### Answer

No.

FastAPI stops processing before the endpoint executes.

---

# 80. Explain the validation flow.

### Answer

Client

↓

JSON Request

↓

Pydantic Validation

↓

Valid?

├── Yes → Endpoint Executes → Prediction

└── No → 422 Unprocessable Entity

---

# Quick Revision

✔ Pydantic validates request and response data.

✔ Request Models validate incoming data.

✔ Response Models validate outgoing data.

✔ Validation occurs before endpoint execution.

✔ Missing fields return 422.

✔ Invalid datatypes return 422.

✔ Compatible datatypes are automatically converted.

✔ Pydantic performs serialization and deserialization automatically.

✔ ML models should never receive unvalidated input.

# FastAPI Interview Questions & Answers (Part 3)
## Dependency Injection, File Uploads, Exception Handling, Middleware, Swagger & Health Endpoints

---

# DEPENDENCY INJECTION

---

# 81. What is Dependency Injection (DI)?

### Answer

Dependency Injection (DI) is a design pattern where FastAPI automatically provides reusable dependencies to endpoint functions instead of creating them manually.

Examples of dependencies include:

- Machine learning models
- Database connections
- Authentication services
- Configuration objects
- Redis clients

---

# 82. Why do we use Dependency Injection?

### Answer

Dependency Injection helps:

- Reduce duplicate code
- Improve code reusability
- Simplify maintenance
- Improve testability
- Share expensive resources efficiently

---

# 83. What is Depends()?

### Answer

`Depends()` is FastAPI's built-in mechanism for dependency injection.

It tells FastAPI to automatically provide a required dependency to an endpoint function.

---

# 84. Why is Depends() useful in MLOps?

### Answer

Machine learning models are expensive to load.

Instead of loading the model for every request, the model can be loaded once and reused through dependency injection.

This improves response time and reduces resource consumption.

---

# 85. Give examples of dependencies in production APIs.

### Answer

Common dependencies include:

- Machine Learning Model
- Database Connection
- Redis Cache
- JWT Authentication
- API Key Validation
- Configuration Settings

---

# 86. Why shouldn't we load the ML model inside every endpoint?

### Answer

Loading a model repeatedly:

- Increases latency
- Wastes CPU and memory
- Slows predictions
- Makes the code repetitive

Instead, load it once and reuse it.

---

# 87. What are the advantages of Dependency Injection?

### Answer

Advantages include:

- Reusable code
- Better organization
- Easier testing
- Lower memory usage
- Faster API responses
- Cleaner architecture

---

# 88. Explain Dependency Injection with an example.

### Answer

Without Dependency Injection:

```
/predict

↓

Load Model

↓

Predict
```

Every request loads the model again.

With Dependency Injection:

```
Application Starts

↓

Load Model Once

↓

Depends()

↓

Predict

↓

Reuse Same Model
```

---

# FILE UPLOADS

---

# 89. Why do machine learning APIs often require file uploads?

### Answer

Many ML applications receive files instead of JSON.

Examples include:

- Computer Vision → Images
- OCR → PDFs
- Speech Recognition → Audio
- Video Analytics → Videos
- NLP → Documents

---

# 90. Which FastAPI components handle file uploads?

### Answer

FastAPI uses:

- UploadFile
- File()

for handling uploaded files.

---

# 91. What is UploadFile?

### Answer

UploadFile is a FastAPI class that efficiently handles uploaded files by streaming them instead of loading the entire file into memory.

---

# 92. Why is UploadFile preferred over bytes?

### Answer

UploadFile is preferred because it:

- Uses less memory
- Streams large files efficiently
- Is suitable for production
- Supports large uploads

Reading everything as bytes loads the complete file into RAM.

---

# 93. Give examples of APIs that use file uploads.

### Answer

Examples include:

- Image Classification
- OCR Systems
- Face Recognition
- Speech-to-Text
- Video Processing
- Medical Image Analysis

---

# 94. What happens after a file is uploaded?

### Answer

Typical flow:

Client

↓

Upload File

↓

FastAPI

↓

Preprocessing

↓

Machine Learning Model

↓

Prediction

↓

JSON Response

---

# EXCEPTION HANDLING

---

# 95. What is exception handling?

### Answer

Exception handling is the process of catching runtime errors and returning meaningful HTTP responses instead of allowing the application to crash.

---

# 96. Why is exception handling important?

### Answer

Exception handling:

- Prevents crashes
- Improves debugging
- Returns useful error messages
- Improves user experience
- Makes APIs more reliable

---

# 97. What is HTTPException?

### Answer

HTTPException is FastAPI's built-in exception class used to return HTTP status codes and error messages.

---

# 98. Give common HTTP status codes.

### Answer

| Status Code | Meaning |
|-------------|---------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 422 | Validation Error |
| 500 | Internal Server Error |

---

# 99. What happens if the model file is missing?

### Answer

Instead of crashing,

FastAPI should return an appropriate error such as:

```
404 Not Found
```

or

```
500 Internal Server Error
```

along with a meaningful message explaining the problem.

---

# 100. Why should APIs return meaningful error messages?

### Answer

Meaningful error messages:

- Help clients understand the problem
- Improve debugging
- Reduce support effort
- Improve developer experience

---

# MIDDLEWARE

---

# 101. What is Middleware?

### Answer

Middleware is software that executes before and/or after every HTTP request and response.

It performs common processing without modifying individual endpoint functions.

---

# 102. Why do we use Middleware?

### Answer

Middleware avoids repeating the same code in every endpoint.

It centralizes common functionality.

---

# 103. What are common uses of Middleware?

### Answer

Middleware is commonly used for:

- Logging
- Authentication
- Measuring latency
- CORS
- Security headers
- Request tracing

---

# 104. How does Middleware work?

### Answer

Client

↓

Middleware

↓

Endpoint

↓

Middleware

↓

Client

Middleware can inspect or modify both the request and the response.

---

# 105. Why is Middleware useful in MLOps?

### Answer

Middleware can automatically:

- Log every prediction request
- Measure prediction latency
- Authenticate users
- Attach request IDs
- Record metrics for monitoring

---

# SWAGGER & OPENAPI

---

# 106. What is Swagger?

### Answer

Swagger is an interactive API documentation interface automatically generated by FastAPI.

It allows developers to explore and test APIs directly from the browser.

---

# 107. Where is Swagger available?

### Answer

Swagger UI is available at:

```
/docs
```

---

# 108. What is ReDoc?

### Answer

ReDoc is another documentation interface generated by FastAPI.

It is available at:

```
/redoc
```

---

# 109. What is OpenAPI?

### Answer

OpenAPI is a standard specification for describing REST APIs.

FastAPI automatically generates an OpenAPI specification from your routes and Pydantic models.

---

# 110. Why is Swagger useful?

### Answer

Swagger allows developers to:

- Read API documentation
- Test endpoints
- View request schemas
- View response schemas
- Explore APIs without Postman

---

# 111. Why doesn't FastAPI require manual API documentation?

### Answer

FastAPI reads:

- Route definitions
- Type hints
- Pydantic models

and automatically generates documentation.

---

# HEALTH ENDPOINTS

---

# 112. What is a Health Endpoint?

### Answer

A health endpoint is a lightweight API endpoint that reports whether the application is running and ready to serve requests.

---

# 113. What is the most common health endpoint?

### Answer

```
GET /health
```

---

# 114. What does a health endpoint typically return?

### Answer

Example:

```json
{
    "status":"healthy"
}
```

Production systems may also report:

- Database status
- Redis status
- Model status
- GPU availability

---

# 115. Why are health endpoints important?

### Answer

Health endpoints allow external systems to determine whether the application is healthy.

They are commonly used by:

- Docker
- Kubernetes
- Load Balancers
- Monitoring Systems

---

# 116. How does Kubernetes use health endpoints?

### Answer

Kubernetes periodically calls the health endpoint using:

- Liveness Probes
- Readiness Probes

If the application becomes unhealthy,

Kubernetes can restart the container automatically.

---

# 117. What is the difference between Liveness and Readiness?

### Answer

**Liveness Probe**

Checks whether the application is still running.

If it fails, Kubernetes restarts the container.

**Readiness Probe**

Checks whether the application is ready to receive traffic.

If it fails, Kubernetes temporarily stops sending requests.

---

# 118. What should a production health endpoint check?

### Answer

A robust health endpoint may verify:

- Application running
- Model loaded
- Database connected
- Redis connected
- Disk space available
- External services reachable

---

# 119. Why should health endpoints be lightweight?

### Answer

Health checks are executed frequently.

Heavy computations would:

- Increase latency
- Waste resources
- Slow Kubernetes health checks

---

# 120. Explain a complete FastAPI request lifecycle.

### Answer

```
Client
    │
HTTP Request
    │
    ▼
Uvicorn (ASGI Server)
    │
    ▼
Middleware
    │
    ▼
Routing
    │
    ▼
Request Model (Pydantic)
    │
Validation
    │
Dependency Injection
    │
Business Logic / ML Model
    │
Response Model
    │
Middleware
    │
JSON Response
    │
Client
```

---

# Quick Revision

✅ Dependency Injection provides reusable resources using `Depends()`.

✅ ML models should be loaded once and reused.

✅ UploadFile streams files efficiently for large uploads.

✅ Exception handling prevents application crashes.

✅ `HTTPException` returns meaningful HTTP errors.

✅ Middleware runs before and/or after every request.

✅ Swagger (`/docs`) provides interactive API documentation.

✅ ReDoc (`/redoc`) provides an alternative documentation UI.

✅ Health endpoints monitor application availability.

✅ Kubernetes uses liveness and readiness probes to monitor FastAPI applications.

---
# FastAPI Interview Questions & Answers (Part 4A)
## Async, Sync, Background Tasks, CORS & Authentication

---

# ASYNCHRONOUS PROGRAMMING

---

# 121. What is synchronous programming?

### Answer

Synchronous programming executes tasks one after another.

The next task starts only after the previous task has finished.

Example:

```
Request 1
↓

Finish

↓

Request 2

↓

Finish
```

If one request takes a long time, all subsequent requests must wait.

---

# 122. What is asynchronous programming?

### Answer

Asynchronous programming allows multiple requests to be processed concurrently.

Instead of waiting for one request to finish, the server can switch to handling another request while waiting for I/O operations such as database queries or API calls.

---

# 123. Why does FastAPI support asynchronous programming?

### Answer

FastAPI supports asynchronous programming to improve performance and scalability by efficiently handling multiple concurrent requests, especially for I/O-bound operations.

---

# 124. Difference between synchronous and asynchronous programming?

| Synchronous | Asynchronous |
|-------------|--------------|
| Executes one task at a time | Handles multiple tasks concurrently |
| Blocking | Non-blocking |
| Slower for I/O | Faster for I/O |
| Easier to understand | More scalable |

---

# 125. What is a blocking operation?

### Answer

A blocking operation prevents the program from executing other tasks until the current task finishes.

Examples:

- Reading large files
- Database queries
- Calling external APIs

---

# 126. What is a non-blocking operation?

### Answer

A non-blocking operation allows the application to continue processing other requests while waiting for an operation to complete.

---

# 127. What does async mean?

### Answer

The `async` keyword defines an asynchronous function that can pause while waiting for I/O operations without blocking the entire application.

---

# 128. What does await mean?

### Answer

`await` pauses the execution of an asynchronous function until the awaited operation completes.

While waiting, FastAPI can process other requests.

---

# 129. When should you use async in FastAPI?

### Answer

Use async when performing I/O-bound operations such as:

- Database queries
- API calls
- Reading files
- Cloud storage access
- Redis operations

---

# 130. Should CPU-intensive ML prediction be async?

### Answer

Usually **No**.

Machine learning inference is CPU-bound rather than I/O-bound.

Making prediction functions async generally does not improve performance.

Async mainly benefits operations waiting on external resources.

---

# 131. Give examples of async operations in MLOps.

### Answer

Examples include:

- Downloading a model
- Reading from Redis
- Fetching feature data
- Querying a database
- Calling another API

---

# 132. What happens if you make everything async?

### Answer

Not everything benefits from async.

CPU-intensive work still occupies the processor.

Using async unnecessarily may increase code complexity without improving performance.

---

# 133. Is FastAPI automatically asynchronous?

### Answer

FastAPI supports both synchronous and asynchronous endpoints.

Developers choose whether to use `def` or `async def` based on the workload.

---

# BACKGROUND TASKS

---

# 134. What are Background Tasks?

### Answer

Background Tasks allow FastAPI to execute work after returning a response to the client.

The client does not need to wait for the background operation to finish.

---

# 135. Why are Background Tasks useful?

### Answer

They improve response time by moving non-critical work outside the request-response cycle.

---

# 136. Give examples of Background Tasks in MLOps.

### Answer

Examples include:

- Logging predictions
- Saving audit records
- Sending emails
- Updating dashboards
- Retraining models
- Uploading files
- Triggering notifications

---

# 137. Explain Background Tasks with an example.

### Answer

Without Background Tasks:

```
Prediction

↓

Save Logs

↓

Send Email

↓

Return Response
```

Client waits for everything.

With Background Tasks:

```
Prediction

↓

Return Response Immediately

↓

Background

↓

Save Logs

↓

Send Email
```

The client receives the prediction much faster.

---

# 138. Should model prediction itself be a Background Task?

### Answer

No.

Prediction is the main purpose of the request.

The client expects the prediction immediately.

Only secondary work should be executed in the background.

---

# CORS

---

# 139. What is CORS?

### Answer

CORS (Cross-Origin Resource Sharing) is a browser security mechanism that controls whether a web application can access resources hosted on another origin.

---

# 140. What is an origin?

### Answer

An origin is defined by:

- Protocol
- Domain
- Port

Example:

```
http://localhost:3000
```

and

```
http://localhost:8000
```

have different origins because their ports differ.

---

# 141. Why does CORS exist?

### Answer

CORS protects users by preventing malicious websites from making unauthorized requests to another website using the user's browser.

---

# 142. Why do React applications often get CORS errors?

### Answer

During development:

Frontend:

```
localhost:3000
```

Backend:

```
localhost:8000
```

These are different origins.

Unless FastAPI explicitly allows the frontend origin, the browser blocks the request.

---

# 143. How do we solve CORS errors in FastAPI?

### Answer

By adding CORS middleware and allowing trusted origins.

This tells browsers which websites are permitted to access the API.

---

# AUTHENTICATION

---

# 144. What is authentication?

### Answer

Authentication is the process of verifying the identity of a user or application before allowing access to protected resources.

---

# 145. Why is authentication important in MLOps APIs?

### Answer

Machine learning APIs should not be publicly accessible.

Authentication helps:

- Protect models
- Prevent unauthorized access
- Prevent misuse
- Control API usage
- Protect sensitive data

---

# 146. What are common authentication methods?

### Answer

Common methods include:

- API Keys
- JWT Tokens
- OAuth2
- Basic Authentication

---

# 147. What is an API Key?

### Answer

An API Key is a unique secret value sent with each request to identify and authorize the client.

---

# 148. What is JWT?

### Answer

JWT (JSON Web Token) is a secure token used to authenticate users without storing session information on the server.

---

# 149. Difference between Authentication and Authorization?

| Authentication | Authorization |
|----------------|---------------|
| Verifies identity | Determines permissions |
| "Who are you?" | "What can you access?" |

---

# 150. Explain the authentication flow.

### Answer

```
Client

↓

Login

↓

Server verifies credentials

↓

Generate JWT/API Key

↓

Client stores token

↓

Future Requests

↓

Authorization Header

↓

Server validates token

↓

Access Granted
```

---

# Quick Revision

✅ FastAPI supports both synchronous and asynchronous programming.

✅ Async is useful for I/O-bound operations.

✅ CPU-intensive ML inference usually does not benefit from async.

✅ Background Tasks execute work after sending the response.

✅ CORS controls cross-origin browser requests.

✅ React and FastAPI often require CORS configuration during development.

✅ Authentication verifies identity before allowing API access.

✅ API Keys and JWT are common authentication mechanisms.

✅ Authentication answers "Who are you?"

✅ Authorization answers "What are you allowed to do?"

---
# FastAPI Interview Questions & Answers (Part 4B)
## Gunicorn, Uvicorn, Rate Limiting, API Versioning, Deployment & Advanced MLOps

---

# GUNICORN & UVICORN

---

# 151. Why shouldn't we use `uvicorn main:app` directly in production?

### Answer

Although Uvicorn is an excellent ASGI server, running a single Uvicorn process means only one worker is handling requests.

If that worker crashes, the application becomes unavailable.

Production deployments typically require:

- Multiple worker processes
- Automatic worker restart
- Better process management

This is why Gunicorn is commonly used together with Uvicorn.

---

# 152. What is Gunicorn?

### Answer

Gunicorn (Green Unicorn) is a production-grade Python application server that manages multiple worker processes.

It improves:

- Reliability
- Scalability
- Fault tolerance

---

# 153. How do Gunicorn and Uvicorn work together?

### Answer

Gunicorn manages multiple worker processes.

Each worker runs a Uvicorn ASGI server.

Architecture:

```
Users

↓

Gunicorn

↓

Worker 1 (Uvicorn)

Worker 2 (Uvicorn)

Worker 3 (Uvicorn)

↓

FastAPI
```

---

# 154. Why are multiple workers useful?

### Answer

Multiple workers:

- Handle more concurrent requests
- Improve availability
- Continue serving traffic if one worker crashes
- Better utilize multiple CPU cores

---

# 155. What happens if one worker crashes?

### Answer

Gunicorn automatically starts a new worker.

This improves application reliability.

---

# API VERSIONING

---

# 156. What is API Versioning?

### Answer

API Versioning is the practice of maintaining multiple versions of an API without breaking existing clients.

Example:

```
/v1/predict

/v2/predict
```

---

# 157. Why is API Versioning important in MLOps?

### Answer

Machine learning models evolve.

A new model may:

- Require different input features
- Return different outputs
- Use a different prediction pipeline

Versioning allows old clients to continue working.

---

# 158. What are common API versioning strategies?

### Answer

Common approaches include:

- URL Versioning
- Header Versioning
- Query Parameter Versioning

URL versioning is the most common.

Example:

```
/v1/predict

/v2/predict
```

---

# RATE LIMITING

---

# 159. What is Rate Limiting?

### Answer

Rate Limiting restricts how many requests a client can send within a specific time period.

Example:

```
100 requests/minute
```

---

# 160. Why is Rate Limiting important?

### Answer

Rate Limiting protects APIs from:

- Abuse
- Accidental overload
- Brute-force attacks
- Excessive resource consumption

---

# 161. Why is Rate Limiting especially useful for ML APIs?

### Answer

Machine learning inference is computationally expensive.

Without rate limiting, a malicious user could send thousands of prediction requests and exhaust CPU or GPU resources.

---

# 162. Give examples of Rate Limits.

### Answer

Examples:

- 10 requests/second
- 100 requests/minute
- 1000 requests/day

---

# DEPLOYMENT

---

# 163. How is FastAPI typically deployed in production?

### Answer

Typical deployment:

```
Client

↓

Load Balancer

↓

Kubernetes

↓

Docker Container

↓

Gunicorn

↓

Uvicorn

↓

FastAPI

↓

Machine Learning Model
```

---

# 164. Why do we containerize FastAPI?

### Answer

Docker provides:

- Consistent environments
- Easy deployment
- Portability
- Dependency isolation
- Reproducible builds

---

# 165. Why deploy FastAPI on Kubernetes?

### Answer

Kubernetes provides:

- Auto scaling
- Self-healing
- Rolling updates
- Load balancing
- High availability

---

# 166. Why is a Health Endpoint important for Kubernetes?

### Answer

Kubernetes periodically checks:

```
GET /health
```

using:

- Liveness Probe
- Readiness Probe

If health checks fail, Kubernetes restarts or removes the unhealthy pod.

---

# ADVANCED MLOPS QUESTIONS

---

# 167. Where should an ML model be loaded?

### Answer

The model should be loaded once during application startup or provided through Dependency Injection.

Avoid loading it inside every prediction request.

---

# 168. Why shouldn't a model be loaded for every request?

### Answer

Repeated loading:

- Increases latency
- Wastes memory
- Reduces throughput
- Slows the API

Loading once significantly improves performance.

---

# 169. How would you deploy a production ML API?

### Answer

Typical architecture:

```
Client

↓

FastAPI

↓

Pydantic Validation

↓

Authentication

↓

Dependency Injection

↓

ML Model

↓

Logging

↓

Prometheus Metrics

↓

Grafana Dashboard

↓

Response
```

Containerized using Docker and deployed on Kubernetes.

---

# 170. Explain the complete MLOps inference pipeline.

### Answer

```
Client

↓

HTTP Request

↓

Load Balancer

↓

Kubernetes

↓

FastAPI

↓

Middleware

↓

Authentication

↓

Pydantic Validation

↓

Dependency Injection

↓

Machine Learning Model

↓

Prediction

↓

Logging

↓

Prometheus Metrics

↓

Grafana Dashboard

↓

JSON Response

↓

Client
```

---

# BONUS INTERVIEW QUESTIONS

---

# 171. Why shouldn't prediction endpoints trust client input?

### Answer

Clients may send:

- Missing fields
- Incorrect datatypes
- Malicious inputs
- Unexpected values

Validation protects the model and application.

---

# 172. Why are Request Models important?

### Answer

They guarantee that incoming data matches the schema expected by the machine learning model.

---

# 173. Why are Response Models important?

### Answer

They ensure every client receives a consistent and validated response format.

---

# 174. Why are health endpoints important in production?

### Answer

They allow Docker, Kubernetes, monitoring systems, and load balancers to verify whether the application is alive and ready to serve traffic.

---

# 175. If your prediction API suddenly becomes slow, what would you check?

### Answer

I would investigate:

- CPU usage
- Memory usage
- GPU utilization
- Request latency
- Model loading time
- Database latency
- Network latency
- Prometheus metrics
- Application logs
- Kubernetes pod health

---

# 176. Which monitoring metrics would you expose?

### Answer

Infrastructure:

- CPU usage
- Memory usage
- Disk usage
- Network I/O

Application:

- Total requests
- Error count
- Request latency

ML-specific:

- Prediction count
- Prediction latency
- Model drift
- Data drift
- Feature statistics

---

# 177. How do Prometheus and FastAPI work together?

### Answer

FastAPI exposes metrics through an endpoint such as:

```
/metrics
```

Prometheus periodically scrapes this endpoint and stores the metrics.

Grafana then visualizes those metrics.

---

# 178. Why is structured logging preferred?

### Answer

Structured logs:

- Are machine-readable
- Are easier to search
- Work well with ELK, Loki, Splunk
- Simplify production debugging

---

# 179. What would you log for every prediction request?

### Answer

Typical fields include:

- Timestamp
- Request ID
- Endpoint
- User ID (if applicable)
- Prediction latency
- HTTP status code
- Model version
- Error message (if any)

Sensitive information should not be logged.

---

# 180. Explain your ideal production MLOps architecture.

### Answer

```
User

↓

Load Balancer

↓

Kubernetes

↓

FastAPI

↓

Authentication

↓

Middleware

↓

Validation (Pydantic)

↓

Dependency Injection

↓

Machine Learning Model

↓

Prediction

↓

Logging

↓

Prometheus

↓

Grafana

↓

Monitoring & Alerts
```

This architecture provides:

- Scalability
- Reliability
- Monitoring
- Observability
- Security
- Maintainability

---

# Final FastAPI Revision

✅ FastAPI builds REST APIs.

✅ Uvicorn runs FastAPI.

✅ Gunicorn manages multiple Uvicorn workers.

✅ Pydantic validates request and response data.

✅ Dependency Injection loads reusable resources.

✅ UploadFile efficiently handles file uploads.

✅ Middleware executes before and after every request.

✅ Swagger automatically documents APIs.

✅ Health endpoints support Docker and Kubernetes.

✅ Async improves I/O-bound workloads.

✅ Background Tasks execute non-critical work after responding.

✅ CORS enables safe cross-origin communication.

✅ JWT and API Keys authenticate users.

✅ Rate Limiting protects APIs from abuse.

✅ Versioning allows APIs to evolve safely.

✅ Docker packages the application.

✅ Kubernetes deploys and scales the application.

✅ Prometheus monitors metrics.

✅ Grafana visualizes metrics.

✅ Logging records application events.

✅ Together, these components form a production-ready MLOps inference system.

