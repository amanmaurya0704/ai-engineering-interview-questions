# REST APIs - Interview Question & Answer Cheat Sheet (MLOps)

---

# 1. What is an API?

**Answer:**

An **API (Application Programming Interface)** is a communication interface that enables two or more software applications to communicate and exchange data without knowing each other's internal implementation.

---

# 2. What is a REST API?

**Answer:**

A **REST (Representational State Transfer) API** is a stateless architectural style that enables communication between clients and servers over the HTTP protocol using standard HTTP methods such as GET, POST, PUT, PATCH, and DELETE.

---

# 3. Why is REST called stateless?

**Answer:**

REST is called **stateless** because the server does not store client session information between requests. Every request must contain all the information required to process it independently.

---

# 4. What are the advantages of REST APIs?

**Answer:**

- Stateless communication
- Scalable
- Lightweight
- Language-independent
- Easy to integrate
- Supports multiple clients (web, mobile, backend)
- Uses standard HTTP protocol

---

# 5. Explain Client-Server Architecture.

**Answer:**

Client-Server Architecture is a software architecture in which the client sends requests to the server, and the server processes the request, performs business logic, accesses databases or machine learning models, and returns a response.

---

# 6. What is HTTP?

**Answer:**

HTTP (HyperText Transfer Protocol) is the standard communication protocol used for sending requests and receiving responses between clients and servers over the web.

An HTTP request generally contains:

- HTTP Method
- URL
- Headers
- Request Body (optional)

An HTTP response contains:

- Status Code
- Headers
- Response Body

---

# 7. What are HTTP Methods?

**Answer:**

HTTP methods define the action to be performed on a resource.

| Method | Purpose |
|---------|---------|
| GET | Retrieve data |
| POST | Create resource or send data for processing |
| PUT | Replace an entire resource |
| PATCH | Update specific fields of a resource |
| DELETE | Remove a resource |

---

# 8. Why is POST generally used for ML prediction APIs?

**Answer:**

Prediction APIs require clients to send input features such as age, salary, images, or text to the server. Since POST is designed to send data in the request body for processing, it is the preferred HTTP method for prediction endpoints.

---

# 9. What are HTTP Status Codes?

**Answer:**

HTTP Status Codes are three-digit codes returned by the server to indicate whether a request was successful, failed, or requires further action.

---

# 10. Common HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK (Request successful) |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized (Authentication failed) |
| 403 | Forbidden (Permission denied) |
| 404 | Not Found |
| 422 | Unprocessable Entity (Validation failed) |
| 500 | Internal Server Error |
| 503 | Service Unavailable |

---

# 11. Difference between 401 and 403

**401 Unauthorized**

- Authentication failed
- Missing token
- Invalid token
- Expired token

**403 Forbidden**

- Authentication successful
- User lacks permission to perform the requested action

---

# 12. What is JSON?

**Answer:**

JSON (JavaScript Object Notation) is a lightweight, human-readable, text-based, language-independent data interchange format used to exchange structured data between clients and servers. It stores data as key-value pairs and is the most common data format used in REST APIs.

---

# 13. Why is JSON used in REST APIs?

**Answer:**

JSON is used because it is:

- Lightweight
- Human-readable
- Language-independent
- Easy to parse
- Widely supported

---

# 14. What is a Key-Value Pair?

**Answer:**

A key-value pair stores data where each unique key is associated with a corresponding value.

Example:

```json
{
    "age": 25,
    "salary": 50000
}
```

- age → Key
- 25 → Value

---

# 15. Difference between Python Dictionary and JSON

| Python Dictionary | JSON |
|-------------------|------|
| True | true |
| False | false |
| None | null |
| Python object | Text format |

---

# 16. What is the Request–Response Lifecycle?

**Answer:**

The Request–Response Lifecycle is the sequence of steps that occurs from the moment a client sends an HTTP request until it receives an HTTP response from the server.

Typical flow:

Client

↓

Uvicorn

↓

Middleware

↓

Route Matching

↓

Pydantic Validation

↓

Dependency Injection

↓

Business Logic / ML Model

↓

Response Validation

↓

JSON Response

↓

Client

---

# 17. Does a request directly reach the ML model?

**Answer:**

No.

A request first passes through:

- Uvicorn
- Middleware
- Route Matching
- Request Validation
- Dependency Injection

Only then does it reach the endpoint where the ML model is executed.

---

# 18. Where does FastAPI perform request validation?

**Answer:**

FastAPI performs request validation using Pydantic before the endpoint function executes.

If validation fails, FastAPI immediately returns:

422 Unprocessable Entity

---

# 19. What are HTTP Headers?

**Answer:**

HTTP Headers are key-value pairs that contain metadata about an HTTP request or response. They provide additional information such as content type, authentication credentials, accepted response format, caching policies, and client information.

---

# 20. Why are HTTP Headers used?

**Answer:**

Headers are used for:

- Authentication
- Security
- Logging
- Caching
- API Versioning
- Content Negotiation
- Debugging

---

# 21. Difference between Headers and Body

| Headers | Body |
|----------|------|
| Metadata | Actual data |
| Authentication | Customer features |
| Content-Type | JSON input |
| API Version | Images/Files |
| Client Information | Prediction request |

---

# 22. What does Content-Type specify?

**Answer:**

The `Content-Type` header specifies the format of the request body so that the server knows how to interpret the incoming data.

Example:

```text
Content-Type: application/json
```

---

# 23. What is the Authorization Header?

**Answer:**

The Authorization header carries authentication credentials such as JWT tokens or API keys. It allows the server to verify the identity of the client before granting access to protected resources.

Example:

```text
Authorization: Bearer eyJhbGc...
```

---

# 24. Why aren't JWT tokens sent in the request body?

**Answer:**

JWT tokens are authentication credentials and, by HTTP convention, are transmitted using the Authorization header. This allows authentication middleware and libraries to process credentials consistently before the request reaches the application logic. When HTTPS is used, both headers and the request body are encrypted during transmission.

---

# 25. What is Authentication?

**Answer:**

Authentication is the process of verifying the identity of a user or client before allowing access to protected resources.

It answers:

**Who are you?**

---

# 26. What is Authorization?

**Answer:**

Authorization is the process of determining what actions an authenticated user is allowed to perform.

It answers:

**What are you allowed to do?**

---

# 27. Difference between Authentication and Authorization

| Authentication | Authorization |
|----------------|---------------|
| Verifies identity | Verifies permissions |
| Who are you? | What can you do? |
| Happens first | Happens after authentication |

---

# 28. What is JWT?

**Answer:**

JWT (JSON Web Token) is a compact, self-contained token used to securely transmit authentication information between a client and a server. It is commonly sent using the Authorization header as a Bearer token.

Example:

```text
Authorization: Bearer <JWT_TOKEN>
```

---

# 29. Why should Basic Authentication be used with HTTPS?

**Answer:**

Basic Authentication transmits usernames and passwords using Base64 encoding, which is not encryption. HTTPS encrypts the entire HTTP request during transmission, preventing attackers from intercepting and reading authentication credentials.

---

# 30. What is API Versioning?

**Answer:**

API Versioning is the practice of maintaining multiple versions of an API to ensure backward compatibility, allowing new features or changes to be introduced without breaking existing client applications.

---

# 31. Why is API Versioning important?

**Answer:**

API Versioning is important because it:

- Maintains backward compatibility
- Prevents breaking changes
- Enables gradual client migration
- Reduces production failures
- Simplifies long-term maintenance

---

# 32. What is a Breaking Change?

**Answer:**

A breaking change is any modification to an API that causes existing client applications to stop working without requiring changes on the client side.

Example:

Old API:

```json
{
    "age":25,
    "income":50000
}
```

New API:

```json
{
    "age":25,
    "income":50000,
    "credit_score":720
}
```

Older clients fail because they do not provide the new required field.

---

# 33. Which API Versioning strategy is most commonly used?

**Answer:**

The most commonly used strategy is URL Versioning.

Example:

```text
/api/v1/predict

/api/v2/predict
```

This approach is simple, explicit, and easy to maintain.