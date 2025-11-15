---
name: api-design
description: Design consistent, maintainable REST and GraphQL APIs with proper versioning, error handling, and documentation. Apply when creating new APIs or refactoring existing ones.
---

# API Design Skill

Design robust, consistent, and developer-friendly APIs that follow industry best practices and stand the test of time.

## Core Principles

### 1. **Consistency**
- Predictable naming conventions
- Uniform response structures
- Standard error formats
- Consistent authentication patterns

### 2. **Developer Experience**
- Self-documenting endpoints
- Clear error messages
- Comprehensive documentation
- Easy to explore and test

### 3. **Future-Proof**
- Versioning strategy from day one
- Backwards compatibility
- Graceful deprecation
- Extensible design

### 4. **Security First**
- Authentication and authorization
- Input validation
- Rate limiting
- HTTPS only

## REST API Design

### URL Structure

**Pattern**: `/api/version/resource/identifier/sub-resource`

**Best Practices**:
```
✅ Good:
GET    /api/v1/users
GET    /api/v1/users/123
GET    /api/v1/users/123/orders
POST   /api/v1/users
PUT    /api/v1/users/123
DELETE /api/v1/users/123

❌ Bad:
GET    /api/getUsers
POST   /api/user/create
GET    /api/users/getById?id=123
DELETE /api/deleteUser/123
```

**Rules**:
- Use **plural nouns** for resources (`/users`, not `/user`)
- Use **kebab-case** for multi-word resources (`/shipping-addresses`)
- Avoid verbs in URLs (use HTTP methods instead)
- Keep URLs **short and readable**
- Use **path parameters** for IDs (`/users/123`)
- Use **query parameters** for filtering/pagination (`/users?role=admin&page=2`)

### HTTP Methods

| Method | Purpose | Idempotent | Request Body | Response Body |
|--------|---------|------------|--------------|---------------|
| GET | Retrieve resource(s) | Yes | No | Yes |
| POST | Create new resource | No | Yes | Yes (created resource) |
| PUT | Replace entire resource | Yes | Yes | Yes (updated resource) |
| PATCH | Partial update | No | Yes | Yes (updated resource) |
| DELETE | Remove resource | Yes | Optional | Optional |

**Usage Examples**:
```
GET /api/v1/users/123
→ Returns user with ID 123

POST /api/v1/users
Body: { "name": "Alice", "email": "alice@example.com" }
→ Creates new user, returns created user with ID

PUT /api/v1/users/123
Body: { "name": "Alice Smith", "email": "alice@example.com", "role": "admin" }
→ Replaces entire user object

PATCH /api/v1/users/123
Body: { "email": "newemail@example.com" }
→ Updates only email field

DELETE /api/v1/users/123
→ Deletes user, returns 204 No Content
```

### Response Formats

**Successful Response Structure**:
```json
{
  "data": {
    "id": "123",
    "type": "user",
    "attributes": {
      "name": "Alice",
      "email": "alice@example.com"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

**Collection Response with Pagination**:
```json
{
  "data": [
    { "id": "1", "name": "Alice" },
    { "id": "2", "name": "Bob" }
  ],
  "meta": {
    "total": 100,
    "page": 1,
    "perPage": 20,
    "totalPages": 5
  },
  "links": {
    "self": "/api/v1/users?page=1",
    "next": "/api/v1/users?page=2",
    "last": "/api/v1/users?page=5"
  }
}
```

### Error Response Format

**Standard Error Structure**:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email address is invalid",
    "details": [
      {
        "field": "email",
        "issue": "Must be a valid email format"
      }
    ],
    "requestId": "req_abc123",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

**HTTP Status Codes**:
```
2xx Success:
  200 OK              - Request succeeded
  201 Created         - Resource created successfully
  204 No Content      - Success but no response body

4xx Client Errors:
  400 Bad Request     - Invalid input/malformed request
  401 Unauthorized    - Authentication required
  403 Forbidden       - Authenticated but not authorized
  404 Not Found       - Resource doesn't exist
  409 Conflict        - Resource state conflict
  422 Unprocessable   - Validation failed
  429 Too Many Reqs   - Rate limit exceeded

5xx Server Errors:
  500 Internal Error  - Server encountered error
  502 Bad Gateway     - Upstream service error
  503 Service Unavail - Server temporarily down
```

### Versioning Strategies

**Option 1: URL Versioning (Recommended)**
```
/api/v1/users
/api/v2/users
```
✅ Clear and explicit
✅ Easy to route
✅ Cacheable

**Option 2: Header Versioning**
```
GET /api/users
Accept: application/vnd.myapi.v1+json
```
✅ Clean URLs
❌ Less visible, harder to test

**Option 3: Query Parameter**
```
/api/users?version=1
```
❌ Not recommended (mixes versioning with filtering)

**Version Management**:
- Start with v1 from day one
- Maintain at least 2 versions
- Deprecate old versions with warnings
- Document breaking changes clearly
- Give clients 6-12 months to migrate

### Pagination

**Offset-Based Pagination**:
```
GET /api/v1/users?page=2&perPage=20
GET /api/v1/users?offset=20&limit=20
```
✅ Simple to implement
❌ Performance degrades with large offsets
❌ Inconsistent results if data changes

**Cursor-Based Pagination** (Recommended for large datasets):
```
GET /api/v1/users?cursor=abc123&limit=20
```
✅ Consistent results
✅ Better performance
✅ Works well with real-time data

**Response includes**:
- Current page/cursor
- Total count
- Links to next/previous pages

### Filtering and Sorting

**Filtering**:
```
GET /api/v1/users?role=admin&status=active
GET /api/v1/users?createdAfter=2024-01-01
GET /api/v1/users?name[contains]=alice
```

**Sorting**:
```
GET /api/v1/users?sort=createdAt        // ascending
GET /api/v1/users?sort=-createdAt       // descending (- prefix)
GET /api/v1/users?sort=lastName,firstName  // multiple fields
```

**Field Selection** (sparse fieldsets):
```
GET /api/v1/users?fields=id,name,email
```
Reduces payload size and improves performance.

### Nested Resources

**Pattern**:
```
GET /api/v1/users/123/orders           // User's orders
GET /api/v1/users/123/orders/456       // Specific order
POST /api/v1/users/123/orders          // Create order for user
```

**Limit Nesting**: Avoid more than 2 levels deep
```
❌ /api/v1/users/123/orders/456/items/789/reviews
✅ /api/v1/order-items/789/reviews
```

### Authentication & Authorization

**Authentication Methods**:
1. **API Keys** (simple, low security)
   ```
   Authorization: ApiKey your-api-key-here
   ```

2. **Bearer Tokens** (JWT, OAuth)
   ```
   Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
   ```

3. **OAuth 2.0** (delegated access)
   - Use for third-party integrations
   - Support refresh tokens

**Authorization Patterns**:
- Check permissions on every request
- Return 403 Forbidden for unauthorized actions
- Don't leak info about resources user can't access (404 vs 403)

### Rate Limiting

**Headers**:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1704897600
```

**Response when exceeded**:
```
HTTP/1.1 429 Too Many Requests
Retry-After: 3600

{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Try again in 1 hour."
  }
}
```

**Strategies**:
- Per user/API key limits
- Different limits for different endpoints
- Sliding window or token bucket algorithms

### Idempotency

**Idempotency Keys** (for POST requests):
```
POST /api/v1/payments
Idempotency-Key: unique-key-123
```

Prevents duplicate actions (e.g., double charges) on retries.

**GET, PUT, DELETE** should be naturally idempotent.

## GraphQL API Design

### Schema Design Principles

**Type-First Approach**:
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  published: Boolean!
}
```

**Best Practices**:
- Use **non-null (`!`)** appropriately
- Provide **clear field names** (no abbreviations)
- Use **custom scalars** (DateTime, Email, URL)
- Keep types **focused and cohesive**

### Queries

```graphql
type Query {
  # Single resource
  user(id: ID!): User

  # Collections with pagination
  users(
    first: Int
    after: String
    filter: UserFilter
    sort: UserSort
  ): UserConnection!

  # Search
  searchUsers(query: String!): [User!]!
}

# Pagination types
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

### Mutations

```graphql
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(input: UpdateUserInput!): UpdateUserPayload!
  deleteUser(id: ID!): DeleteUserPayload!
}

input CreateUserInput {
  name: String!
  email: String!
}

type CreateUserPayload {
  user: User
  errors: [UserError!]!
}

type UserError {
  field: String
  message: String!
  code: String!
}
```

**Mutation Best Practices**:
- Use **Input types** for complex arguments
- Return **payload types** with both data and errors
- Make mutations **specific** (not generic CRUD)
- Include **client mutation ID** for request tracking

### Error Handling

**Field-Level Errors**:
```graphql
type CreateUserPayload {
  user: User
  errors: [UserError!]!
}
```

**Top-Level Errors** (for critical failures):
```json
{
  "data": null,
  "errors": [
    {
      "message": "Database connection failed",
      "extensions": {
        "code": "INTERNAL_SERVER_ERROR"
      }
    }
  ]
}
```

### Performance Optimization

**N+1 Query Problem**: Use DataLoader
```
Without batching:
- Fetch 10 posts
- Fetch author for post 1
- Fetch author for post 2
- ... (11 queries!)

With DataLoader:
- Fetch 10 posts
- Fetch all authors in one query (2 queries)
```

**Query Complexity Limits**:
- Prevent malicious deeply nested queries
- Set max depth and complexity scores

**Field-Level Caching**:
- Cache resolver results
- Use cache control directives

## API Documentation

### Essential Documentation

1. **Getting Started Guide**
   - Authentication setup
   - First API call example
   - Common use cases

2. **API Reference**
   - All endpoints/types
   - Request/response examples
   - Parameter descriptions
   - Error codes

3. **Changelog**
   - Version history
   - Breaking changes
   - Deprecations
   - Migration guides

### OpenAPI/Swagger (REST)

**Auto-generate from code** or write spec first:
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /users:
    get:
      summary: List users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
```

### GraphQL Schema as Documentation

GraphQL schema **is** the documentation:
```graphql
"""
Represents a user in the system
"""
type User {
  """
  Unique identifier for the user
  """
  id: ID!

  """
  User's full name (required)
  """
  name: String!
}
```

Use schema introspection tools (GraphiQL, GraphQL Playground).

## API Design Checklist

Before releasing an API:

- [ ] Consistent naming conventions
- [ ] RESTful URL structure or well-designed GraphQL schema
- [ ] Proper HTTP status codes
- [ ] Standardized error responses
- [ ] Versioning strategy implemented
- [ ] Authentication and authorization
- [ ] Rate limiting configured
- [ ] Input validation on all endpoints
- [ ] Pagination for collections
- [ ] Filtering and sorting options
- [ ] HTTPS enforced
- [ ] CORS configured properly
- [ ] Comprehensive documentation
- [ ] Examples for all endpoints
- [ ] Changelog maintained
- [ ] Backwards compatibility guaranteed
- [ ] Deprecation warnings for old features

## Common API Anti-Patterns

❌ **Chatty APIs**: Too many requests to accomplish one task
→ Use batching or composite endpoints

❌ **God endpoints**: One endpoint does everything
→ Keep endpoints focused on single resources

❌ **Leaky abstractions**: Exposing database structure directly
→ Design API around use cases, not tables

❌ **Ignoring HTTP semantics**: Using POST for everything
→ Use proper HTTP methods

❌ **Poor error messages**: "Error 500" with no details
→ Provide actionable error information

❌ **No versioning**: Breaking changes without version bumps
→ Version from day one

❌ **Undocumented**: No examples or unclear parameter descriptions
→ Treat documentation as first-class deliverable

## API Evolution Strategy

### Adding Features (Non-Breaking)
✅ New endpoints
✅ New optional parameters
✅ New response fields
✅ New resource types

### Breaking Changes (Require New Version)
- Removing endpoints
- Removing fields from responses
- Making optional parameters required
- Changing response structure
- Changing authentication

### Deprecation Process
1. Announce deprecation with timeline
2. Add deprecation warnings to API responses
3. Provide migration guide
4. Give clients 6-12 months
5. Remove deprecated features in next major version

---

**Remember**: Good API design is about empathy for developers. Make your API predictable, well-documented, and pleasant to use—it will pay dividends in adoption and satisfaction.
