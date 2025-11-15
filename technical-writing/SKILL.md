---
name: technical-writing
description: Write clear, comprehensive technical documentation including READMEs, API docs, architecture decision records, and onboarding guides. Apply when creating or updating any project documentation.
---

# Technical Writing Skill

Create clear, comprehensive technical documentation that empowers developers to understand, use, and contribute to your project effectively.

## Core Principles

### 1. **Know Your Audience**
- Who will read this?
- What do they already know?
- What do they need to accomplish?
- What's their experience level?

### 2. **Write for Scanning**
- Developers scan first, read later
- Use headers, lists, and formatting
- Put important info first
- Make structure visible

### 3. **Show, Don't Just Tell**
- Include code examples
- Provide working samples
- Use diagrams where helpful
- Give concrete scenarios

### 4. **Keep It Updated**
- Outdated docs worse than no docs
- Update docs with code changes
- Review docs periodically
- Mark deprecated sections clearly

### 5. **Make It Actionable**
- Clear next steps
- Runnable commands
- Copy-paste friendly
- Links to related resources

## README Structure

A great README answers these questions in order:

### 1. What is this?
**One-sentence description** (clear, specific)

```markdown
# Project Name

A lightweight REST API framework for Node.js with built-in validation and authentication.
```

### 2. Why should I care?
**Key benefits** (3-5 bullet points)

```markdown
## Features

- Zero-config authentication with JWT
- Automatic request validation using JSON schemas
- Built-in rate limiting and CORS handling
- Comprehensive error handling
- TypeScript support with full type definitions
```

### 3. Quick start (for impatient developers)

```markdown
## Quick Start

\`\`\`bash
npm install your-package

# Create app.js
echo 'const app = require("your-package")
app.listen(3000)' > app.js

# Run it
node app.js
\`\`\`

Visit http://localhost:3000
```

### 4. Installation

```markdown
## Installation

### Prerequisites

- Node.js 18+
- PostgreSQL 14+
- Redis (optional, for caching)

### Install

\`\`\`bash
npm install your-package
# or
yarn add your-package
# or
pnpm add your-package
\`\`\`
```

### 5. Basic Usage

**Show the 80% use case** first

```markdown
## Usage

\`\`\`javascript
const { createServer, route } = require('your-package')

const app = createServer()

// Define a route
route.get('/users/:id', async (req, res) => {
  const user = await db.users.find(req.params.id)
  return res.json(user)
})

app.listen(3000)
\`\`\`
```

### 6. Configuration

```markdown
## Configuration

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PORT` | No | `3000` | Server port |
| `DATABASE_URL` | Yes | - | PostgreSQL connection string |
| `JWT_SECRET` | Yes | - | Secret for JWT signing |
| `REDIS_URL` | No | - | Redis connection (optional) |

### Config File

Create `config.json`:

\`\`\`json
{
  "port": 3000,
  "database": {
    "host": "localhost",
    "port": 5432
  }
}
\`\`\`
```

### 7. Advanced Topics

```markdown
## Advanced Usage

### Custom Middleware

\`\`\`javascript
app.use(async (req, res, next) => {
  req.startTime = Date.now()
  await next()
  console.log(`Request took ${Date.now() - req.startTime}ms`)
})
\`\`\`

### Authentication

See [Authentication Guide](docs/authentication.md)
```

### 8. API Reference (if applicable)

Link to full API docs or provide summary

### 9. Contributing

```markdown
## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md)

Quick version:
1. Fork the repo
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Commit changes: `git commit -am 'Add feature'`
4. Push: `git push origin feature/my-feature`
5. Open a Pull Request
```

### 10. License

```markdown
## License

MIT © [Your Name]
```

### Optional Sections

```markdown
## FAQ

**Q: Does this work with Express?**
A: Yes, it's fully compatible with Express middleware.

## Troubleshooting

**Problem: Port already in use**
Solution: Change the PORT environment variable or kill the process using the port.

## Examples

See [examples/](examples/) for complete working examples:
- [Basic REST API](examples/basic-api)
- [Auth with JWT](examples/jwt-auth)
- [File uploads](examples/file-upload)

## Roadmap

- [x] JWT authentication
- [x] Request validation
- [ ] GraphQL support (coming Q2 2024)
- [ ] WebSocket support

## Acknowledgments

Built with inspiration from [project] and [project].
```

## API Documentation

### Endpoint Documentation Format

```markdown
### GET /api/users/:id

Retrieve a user by ID.

**Parameters**

| Name | Type | In | Required | Description |
|------|------|-----|----------|-------------|
| `id` | string | path | Yes | User ID |

**Query Parameters**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `include` | string | No | - | Comma-separated list of relations to include (e.g., `posts,comments`) |

**Request Example**

\`\`\`bash
curl -X GET https://api.example.com/api/users/123?include=posts \
  -H "Authorization: Bearer YOUR_TOKEN"
\`\`\`

**Response**

Status: `200 OK`

\`\`\`json
{
  "data": {
    "id": "123",
    "name": "Alice",
    "email": "alice@example.com",
    "posts": [
      { "id": "1", "title": "Hello World" }
    ]
  }
}
\`\`\`

**Error Responses**

| Status | Code | Description |
|--------|------|-------------|
| 404 | `USER_NOT_FOUND` | User with given ID doesn't exist |
| 401 | `UNAUTHORIZED` | Missing or invalid authentication token |
```

### Code Examples Best Practices

```markdown
## Examples

### Basic Example

\`\`\`javascript
// Initialize the client
const client = new ApiClient({
  apiKey: 'your-api-key'
})

// Fetch data
const users = await client.users.list()
console.log(users)
\`\`\`

### With Error Handling

\`\`\`javascript
try {
  const user = await client.users.get('123')
  console.log(user.name)
} catch (error) {
  if (error.code === 'USER_NOT_FOUND') {
    console.log('User not found')
  } else {
    console.error('Unexpected error:', error)
  }
}
\`\`\`

### Full Example

See [complete example](examples/full-example.js) for a working application.
```

## Architecture Decision Records (ADRs)

Document **why** you made architectural choices.

### ADR Template

```markdown
# ADR-001: Use PostgreSQL as Primary Database

## Status

Accepted

## Context

We need a database to store user data, posts, and relationships. Requirements:
- ACID transactions for data consistency
- Support for complex queries and joins
- Strong community support
- Horizontal scalability path

Options considered:
- PostgreSQL
- MySQL
- MongoDB

## Decision

We will use PostgreSQL as our primary database.

## Rationale

**Why PostgreSQL**:
- Strong ACID guarantees for financial transactions
- Excellent support for JSON data (for flexibility)
- Powerful query optimizer for complex joins
- pg_partitioning for future scaling
- Rich ecosystem (extensions, tools)
- Team has experience with PostgreSQL

**Why not MySQL**:
- Less feature-rich JSON support
- Team less familiar

**Why not MongoDB**:
- Eventual consistency model risky for financial data
- Complex transactions added only recently
- Team lacks NoSQL expertise

## Consequences

**Positive**:
- Strong data consistency guarantees
- Flexible with JSONB columns
- Can use full-text search, geospatial, etc.

**Negative**:
- Vertical scaling limits (will need sharding eventually)
- More complex operations than simple key-value store
- Requires connection pooling management

**Neutral**:
- Need to manage migrations carefully
- Monitor query performance

## Follow-up Actions

- [ ] Set up connection pooling (pg_bouncer)
- [ ] Implement migration strategy (Flyway/Liquibase)
- [ ] Configure replication for HA
- [ ] Set up monitoring (pg_stat_statements)

## References

- [PostgreSQL Documentation](https://postgresql.org/docs)
- [Comparing PostgreSQL and MongoDB](link)
```

### When to Write ADRs

- Major technology choices (database, framework, language)
- Architecture patterns (microservices, monolith)
- Security decisions (auth strategy)
- Data modeling approaches
- Third-party service selections
- Refactoring strategies

## Onboarding Documentation

### New Developer Guide Template

```markdown
# Developer Onboarding

Welcome! This guide will get you from zero to productive in ~2 hours.

## Prerequisites

Install these first:
- [Node.js 18+](https://nodejs.org)
- [Docker Desktop](https://docker.com)
- [PostgreSQL 14+](https://postgresql.org)

## Setup (30 minutes)

### 1. Clone the Repository

\`\`\`bash
git clone https://github.com/your-org/your-repo.git
cd your-repo
\`\`\`

### 2. Install Dependencies

\`\`\`bash
npm install
\`\`\`

### 3. Environment Setup

Copy the example environment file:

\`\`\`bash
cp .env.example .env
\`\`\`

Edit `.env` and set:
- `DATABASE_URL`: Ask #engineering on Slack for dev DB credentials
- `JWT_SECRET`: Use any random string for local dev

### 4. Database Setup

\`\`\`bash
npm run db:migrate
npm run db:seed
\`\`\`

### 5. Run the Application

\`\`\`bash
npm run dev
\`\`\`

Visit http://localhost:3000

**Expected output**:
\`\`\`
Server running on port 3000
Database connected
Ready to accept requests
\`\`\`

### 6. Verify Everything Works

\`\`\`bash
npm test
\`\`\`

All tests should pass ✅

## Project Structure (15 minutes)

\`\`\`
src/
├── api/           # API routes and controllers
├── models/        # Database models
├── services/      # Business logic
├── utils/         # Helper functions
└── config/        # Configuration files

tests/
├── unit/          # Unit tests
├── integration/   # Integration tests
└── e2e/           # End-to-end tests
\`\`\`

**Key files**:
- `src/server.js` - Entry point
- `src/config/database.js` - DB configuration
- `package.json` - Dependencies and scripts

## Development Workflow (30 minutes)

### Running Tests

\`\`\`bash
npm test              # All tests
npm run test:unit     # Unit tests only
npm run test:watch    # Watch mode
\`\`\`

### Code Quality

\`\`\`bash
npm run lint          # Check code style
npm run lint:fix      # Auto-fix issues
npm run format        # Format code
\`\`\`

### Git Workflow

1. Create a branch:
   \`\`\`bash
   git checkout -b feature/your-feature-name
   \`\`\`

2. Make changes and commit:
   \`\`\`bash
   git add .
   git commit -m "Add feature: description"
   \`\`\`

3. Push and create PR:
   \`\`\`bash
   git push origin feature/your-feature-name
   \`\`\`

4. Open Pull Request on GitHub
5. Wait for CI to pass
6. Request review from team
7. Merge after approval

### Making Your First Contribution

Try this starter task:
1. Add a new API endpoint: `GET /api/health`
2. It should return: `{ "status": "ok" }`
3. Add a test for the endpoint
4. Create a PR

This will familiarize you with the codebase!

## Common Tasks (30 minutes)

### Adding a New API Endpoint

1. Create route in `src/api/routes/`
2. Create controller in `src/api/controllers/`
3. Add business logic in `src/services/`
4. Add tests in `tests/integration/`

Example: See `src/api/routes/users.js`

### Database Migrations

Create migration:
\`\`\`bash
npm run migration:create add_email_to_users
\`\`\`

Run migrations:
\`\`\`bash
npm run db:migrate
\`\`\`

Rollback:
\`\`\`bash
npm run db:migrate:rollback
\`\`\`

### Debugging

Use VS Code debugger (F5) or:

\`\`\`bash
node --inspect-brk src/server.js
\`\`\`

Then open `chrome://inspect` in Chrome.

## Resources (15 minutes)

### Documentation
- [Architecture Overview](docs/architecture.md)
- [API Documentation](docs/api.md)
- [Database Schema](docs/database.md)

### Team Communication
- Slack: #engineering (general), #help (questions)
- Meetings: Daily standup at 10am, Sprint planning Mondays

### Getting Help
1. Check docs first (here and in `docs/`)
2. Search Slack history
3. Ask in #help channel
4. Pair with a buddy (ask your manager to assign one)

## Next Steps

Now that you're set up:
- [ ] Read [Architecture Overview](docs/architecture.md)
- [ ] Make your first contribution (see above)
- [ ] Attend next team meeting
- [ ] Schedule 1:1 with your manager
- [ ] Join #engineering on Slack

Welcome to the team! 🎉
```

## Writing Best Practices

### Use Active Voice

```markdown
❌ "The server will be started by running `npm start`"
✅ "Run `npm start` to start the server"

❌ "The user object is returned by the API"
✅ "The API returns the user object"
```

### Be Concise

```markdown
❌ "In order to install the dependencies, you should run the npm install command"
✅ "Install dependencies: `npm install`"
```

### Use Consistent Terminology

```markdown
❌ Mixed terms: "user", "account", "member" for same concept
✅ Pick one: Always use "user"
```

### Format Code Consistently

```markdown
✅ Inline code: Use `backticks` for commands, variables, file names
✅ Code blocks: Use triple backticks with language
✅ File paths: Use `code/style/paths.js`
✅ Commands: `npm install`, `git commit`
```

### Structure with Headers

```markdown
# H1: Document Title (only one per doc)
## H2: Main Sections
### H3: Subsections
#### H4: Rarely needed
```

### Use Lists Effectively

```markdown
**Ordered lists** (when sequence matters):
1. Clone repository
2. Install dependencies
3. Run migrations

**Unordered lists** (when order doesn't matter):
- Fast performance
- Easy to use
- Well documented
```

### Tables for Structured Data

```markdown
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | User ID |
| name | string | No | User name |
```

### Link to Related Content

```markdown
For authentication, see [Authentication Guide](./authentication.md).

Learn more about our [API Design](https://docs.example.com/api).
```

## Documentation Types

### README.md
- **Purpose**: Project overview and quick start
- **Audience**: New users, contributors
- **Update**: With major changes

### API Documentation
- **Purpose**: Complete API reference
- **Audience**: API consumers
- **Update**: With every API change

### Architecture Docs
- **Purpose**: System design and decisions
- **Audience**: Developers, architects
- **Update**: With major architecture changes

### Onboarding Guides
- **Purpose**: Get new developers productive
- **Audience**: New team members
- **Update**: Quarterly or when setup changes

### Runbooks
- **Purpose**: Operational procedures
- **Audience**: DevOps, on-call engineers
- **Update**: After incidents or process changes

### Changelog
- **Purpose**: Track changes between versions
- **Audience**: All users
- **Update**: With every release

## Documentation Checklist

Before considering docs complete:

- [ ] Clear purpose stated upfront
- [ ] Audience identified
- [ ] Prerequisites listed
- [ ] Installation steps are complete and tested
- [ ] Code examples are working and copy-paste ready
- [ ] Common use cases covered
- [ ] Error handling shown
- [ ] Links work (no 404s)
- [ ] Formatting is consistent
- [ ] No typos or grammar errors
- [ ] Updated with latest code changes
- [ ] Reviewed by someone unfamiliar with the project

## Common Documentation Mistakes

❌ **Assuming knowledge**: Don't assume readers know your context
❌ **Outdated examples**: Code examples that don't work
❌ **Missing prerequisites**: Forgetting to list what's needed first
❌ **Too much detail**: Overwhelming with unnecessary information
❌ **No examples**: All theory, no practical code
❌ **Buried important info**: Key details hidden deep in text
❌ **Broken links**: Links to non-existent pages
❌ **Inconsistent style**: Mixing formats and terminology

---

**Remember**: Documentation is a first-class deliverable. Good docs reduce support burden, accelerate onboarding, and improve developer experience. Invest in documentation—it pays dividends.
