---
name: database-design
description: Design efficient database schemas with proper normalization, indexing, and query optimization. Apply when creating new databases, optimizing queries, or planning data migrations.
---

# Database Design Skill

Design efficient, scalable, and maintainable database schemas that support application needs while ensuring data integrity and query performance.

## Core Principles

### 1. **Data Integrity First**
- Enforce constraints at database level
- Use appropriate data types
- Define relationships explicitly
- Plan for edge cases

### 2. **Design for Query Patterns**
- Understand how data will be accessed
- Optimize for common queries
- Balance normalization with performance
- Index strategically

### 3. **Plan for Scale**
- Consider data growth
- Anticipate load patterns
- Design for horizontal scaling
- Think about sharding strategies

### 4. **Keep It Simple**
- Start normalized, denormalize if needed
- Avoid premature optimization
- Document design decisions
- Prefer clarity over cleverness

## Data Modeling

### Entity-Relationship Design

**Process**:
```
1. Identify entities (nouns)
2. Define attributes (properties)
3. Establish relationships
4. Determine cardinality
5. Add constraints
```

**Example - E-Commerce**:
```
Entities:
- User
- Product
- Order
- OrderItem
- Category

Relationships:
- User has many Orders (1:N)
- Order has many OrderItems (1:N)
- Product has many OrderItems (1:N)
- Product belongs to many Categories (M:N)
```

### Cardinality

**One-to-One (1:1)**:
```
User ─── UserProfile

Use when:
- Separating rarely accessed data
- Security isolation
- Optional data
```

**One-to-Many (1:N)**:
```
User ───< Orders

Most common relationship
Parent can have multiple children
Child belongs to one parent
```

**Many-to-Many (M:N)**:
```
Products >──< Categories

Requires junction/join table
Can store relationship attributes
```

## Normalization

### Normal Forms

**First Normal Form (1NF)**:
```
- Atomic values (no arrays/lists in columns)
- No repeating groups
- Each row uniquely identifiable

Bad:
| id | name  | phones                    |
|----|-------|---------------------------|
| 1  | Alice | 555-1234, 555-5678        |

Good:
| id | name  | phone    |
|----|-------|----------|
| 1  | Alice | 555-1234 |
| 1  | Alice | 555-5678 |

Or use separate Phone table
```

**Second Normal Form (2NF)**:
```
- Be in 1NF
- No partial dependencies (all non-key depends on whole key)

Bad (composite key: order_id, product_id):
| order_id | product_id | quantity | product_name |
|----------|------------|----------|--------------|
product_name depends only on product_id, not full key

Good: Move product_name to Products table
```

**Third Normal Form (3NF)**:
```
- Be in 2NF
- No transitive dependencies (non-key → non-key)

Bad:
| id | name  | dept_id | dept_name   |
|----|-------|---------|-------------|
dept_name depends on dept_id, not id

Good: Move dept_name to Departments table
```

### When to Denormalize

**Consider denormalization when**:
```
- Read performance is critical
- Data rarely changes
- Joins are expensive (many tables)
- Reporting/analytics queries
- Data aggregations are frequent
```

**Common denormalization patterns**:
```
- Calculated/derived columns
- Materialized views
- Summary tables
- Redundant columns to avoid joins
- Document-style embedding
```

## Schema Design Patterns

### Common Patterns

**Soft Delete**:
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) NOT NULL,
  deleted_at TIMESTAMP,  -- NULL = active

  INDEX idx_active_users (email) WHERE deleted_at IS NULL
);

-- Query active users
SELECT * FROM users WHERE deleted_at IS NULL;
```

**Audit Trail**:
```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  status VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  created_by INT REFERENCES users(id),
  updated_by INT REFERENCES users(id)
);

-- Or separate audit table
CREATE TABLE order_history (
  id SERIAL PRIMARY KEY,
  order_id INT REFERENCES orders(id),
  changed_at TIMESTAMP DEFAULT NOW(),
  changed_by INT REFERENCES users(id),
  old_values JSONB,
  new_values JSONB
);
```

**Polymorphic Associations**:
```sql
-- Option 1: Separate tables (preferred)
CREATE TABLE post_comments (
  id SERIAL PRIMARY KEY,
  post_id INT REFERENCES posts(id),
  content TEXT
);

CREATE TABLE photo_comments (
  id SERIAL PRIMARY KEY,
  photo_id INT REFERENCES photos(id),
  content TEXT
);

-- Option 2: Single table with type
CREATE TABLE comments (
  id SERIAL PRIMARY KEY,
  commentable_type VARCHAR(50),  -- 'Post', 'Photo'
  commentable_id INT,
  content TEXT
);
-- Note: Can't enforce foreign key constraint
```

**Self-Referencing (Hierarchy)**:
```sql
-- Adjacency list
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  parent_id INT REFERENCES categories(id)
);

-- Materialized path
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  path VARCHAR(255)  -- '/1/5/12/'
);

-- Nested sets
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  lft INT,
  rgt INT
);
```

**Tagging**:
```sql
CREATE TABLE tags (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) UNIQUE
);

CREATE TABLE post_tags (
  post_id INT REFERENCES posts(id),
  tag_id INT REFERENCES tags(id),
  PRIMARY KEY (post_id, tag_id)
);

-- Query posts with specific tags
SELECT p.* FROM posts p
JOIN post_tags pt ON p.id = pt.post_id
JOIN tags t ON pt.tag_id = t.id
WHERE t.name IN ('tech', 'database');
```

### Temporal Data

**Type 1: Overwrite**:
```sql
-- Just update the value
UPDATE products SET price = 29.99 WHERE id = 1;
-- History is lost
```

**Type 2: Add Row**:
```sql
CREATE TABLE product_prices (
  id SERIAL PRIMARY KEY,
  product_id INT REFERENCES products(id),
  price DECIMAL(10,2),
  effective_from TIMESTAMP,
  effective_to TIMESTAMP  -- NULL = current
);

-- Get current price
SELECT price FROM product_prices
WHERE product_id = 1 AND effective_to IS NULL;

-- Get price at specific time
SELECT price FROM product_prices
WHERE product_id = 1
  AND effective_from <= '2024-01-15'
  AND (effective_to IS NULL OR effective_to > '2024-01-15');
```

## Indexing Strategy

### Index Types

**B-Tree (Default)**:
```
Good for:
- Equality and range queries
- Sorting
- Most common use cases

Example:
CREATE INDEX idx_users_email ON users(email);
```

**Hash**:
```
Good for:
- Equality comparisons only
- Not range queries

Example:
CREATE INDEX idx_users_email ON users USING hash(email);
```

**GIN (Generalized Inverted)**:
```
Good for:
- Full-text search
- Array contains
- JSONB queries

Example:
CREATE INDEX idx_posts_tags ON posts USING gin(tags);
```

**GiST (Generalized Search Tree)**:
```
Good for:
- Geometric data
- Range types
- Full-text search

Example:
CREATE INDEX idx_locations ON places USING gist(coordinates);
```

### Index Best Practices

**When to Create Indexes**:
```
✅ Columns in WHERE clauses
✅ Columns in JOIN conditions
✅ Columns in ORDER BY
✅ Columns in GROUP BY
✅ Foreign keys
✅ Unique constraints
```

**When NOT to Index**:
```
❌ Small tables
❌ Columns with low cardinality (few unique values)
❌ Frequently updated columns
❌ Columns rarely used in queries
❌ Very wide columns
```

**Composite Indexes**:
```sql
-- Column order matters!
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- This index helps with:
WHERE user_id = 1                           -- Yes
WHERE user_id = 1 AND created_at > '2024'   -- Yes
WHERE created_at > '2024'                   -- No (wrong column first)
ORDER BY user_id, created_at                -- Yes
ORDER BY created_at, user_id                -- No (wrong order)
```

**Partial Indexes**:
```sql
-- Index only subset of data
CREATE INDEX idx_orders_pending ON orders(created_at)
WHERE status = 'pending';

-- Smaller index, faster for specific queries
SELECT * FROM orders
WHERE status = 'pending' AND created_at > '2024-01-01';
```

**Covering Indexes**:
```sql
-- Include all columns needed by query
CREATE INDEX idx_orders_user_total ON orders(user_id, status)
INCLUDE (total);

-- Query satisfied entirely from index (no table lookup)
SELECT status, total FROM orders WHERE user_id = 1;
```

## Query Optimization

### EXPLAIN Analysis

**Reading EXPLAIN Output**:
```sql
EXPLAIN ANALYZE
SELECT * FROM orders
WHERE user_id = 123 AND status = 'pending';

-- Key things to look for:
-- - Seq Scan: Full table scan (usually bad for large tables)
-- - Index Scan: Using index (good)
-- - Index Only Scan: Even better
-- - Nested Loop: Watch out for large tables
-- - Hash Join: Often good for larger datasets
-- - Sort: Check if can use index
-- - Actual rows vs estimated: Statistics accurate?
```

**Common Issues**:
```
Problem: Sequential scan on large table
Solution: Add appropriate index

Problem: Many rows estimated, few actual
Solution: Update statistics (ANALYZE)

Problem: Nested loop with many rows
Solution: Consider hash join, add index

Problem: Sort operation
Solution: Add index for ORDER BY columns
```

### Query Patterns

**Pagination**:
```sql
-- Offset pagination (slow for large offsets)
SELECT * FROM posts
ORDER BY created_at DESC
LIMIT 20 OFFSET 10000;  -- Must skip 10000 rows!

-- Keyset pagination (fast, consistent)
SELECT * FROM posts
WHERE created_at < '2024-01-15T10:30:00'
ORDER BY created_at DESC
LIMIT 20;
```

**Avoiding N+1 Queries**:
```sql
-- Bad: N+1
SELECT * FROM posts WHERE user_id = 1;
-- Then for each post:
SELECT * FROM comments WHERE post_id = ?;

-- Good: JOIN
SELECT p.*, c.*
FROM posts p
LEFT JOIN comments c ON p.id = c.post_id
WHERE p.user_id = 1;

-- Good: IN clause
SELECT * FROM comments
WHERE post_id IN (1, 2, 3, 4, 5);
```

**Batch Operations**:
```sql
-- Bad: Individual inserts
INSERT INTO logs (message) VALUES ('log1');
INSERT INTO logs (message) VALUES ('log2');
-- ... 1000 times

-- Good: Batch insert
INSERT INTO logs (message) VALUES
  ('log1'), ('log2'), ('log3'), ...;

-- Good: COPY (PostgreSQL)
COPY logs (message) FROM '/path/to/data.csv';
```

**Selective Columns**:
```sql
-- Bad: Select all columns
SELECT * FROM users WHERE id = 1;

-- Good: Select only needed columns
SELECT id, name, email FROM users WHERE id = 1;
```

## Data Types

### Choosing Data Types

**Strings**:
```
CHAR(n):      Fixed length, padded (rarely used)
VARCHAR(n):   Variable length with limit
TEXT:         Unlimited length

Use VARCHAR with reasonable limit for validated input
Use TEXT for unbounded user content
```

**Numbers**:
```
SMALLINT:     2 bytes, -32768 to 32767
INTEGER:      4 bytes, -2B to 2B
BIGINT:       8 bytes, very large range
DECIMAL(p,s): Exact, for money
REAL/DOUBLE:  Floating point, approximate

Use DECIMAL for money (never float!)
Use INTEGER for most IDs
Use BIGINT for large sequences
```

**Date/Time**:
```
DATE:         Date only
TIME:         Time only
TIMESTAMP:    Date and time
TIMESTAMPTZ:  With timezone (preferred!)

Always use TIMESTAMPTZ for timestamps
Store in UTC, convert on display
```

**Other**:
```
BOOLEAN:      true/false
UUID:         Universally unique identifier
JSONB:        Binary JSON (indexed, preferred over JSON)
ARRAY:        Array of values
ENUM:         Enumerated type
```

### UUID vs Serial IDs

**Serial/Identity**:
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY
);

Pros:
- Simple, sequential
- Smaller storage
- Better index locality

Cons:
- Predictable (security)
- Hard to merge databases
- Centralized generation
```

**UUID**:
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid()
);

Pros:
- Globally unique
- Decentralized generation
- Secure (non-guessable)
- Easy to merge data

Cons:
- Larger storage (16 bytes)
- Worse index performance
- Harder to debug
```

## Constraints

### Types of Constraints

**NOT NULL**:
```sql
CREATE TABLE users (
  email VARCHAR(255) NOT NULL
);
```

**UNIQUE**:
```sql
CREATE TABLE users (
  email VARCHAR(255) UNIQUE
);

-- Composite unique
ALTER TABLE user_roles
ADD CONSTRAINT uq_user_role UNIQUE (user_id, role_id);
```

**PRIMARY KEY**:
```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY
);

-- Composite primary key
CREATE TABLE order_items (
  order_id INT,
  product_id INT,
  PRIMARY KEY (order_id, product_id)
);
```

**FOREIGN KEY**:
```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INT REFERENCES users(id)
    ON DELETE CASCADE
    ON UPDATE CASCADE
);

-- Options:
-- ON DELETE CASCADE:    Delete children
-- ON DELETE SET NULL:   Set FK to NULL
-- ON DELETE RESTRICT:   Prevent deletion
-- ON DELETE NO ACTION:  Same as RESTRICT (default)
```

**CHECK**:
```sql
CREATE TABLE products (
  price DECIMAL(10,2) CHECK (price > 0),
  status VARCHAR(20) CHECK (status IN ('active', 'inactive', 'archived'))
);
```

## Migrations

### Migration Best Practices

**Safe Migration Pattern**:
```
1. Add new column (nullable or with default)
2. Deploy code that writes to both old and new
3. Migrate existing data
4. Deploy code that reads from new
5. Remove old column

Never:
- Drop column while code uses it
- Rename column without transition
- Add NOT NULL without default
```

**Backward Compatible Changes**:
```
✅ Safe:
- Add new table
- Add nullable column
- Add column with default
- Add index (CONCURRENTLY)
- Add constraint (NOT VALID + VALIDATE)

❌ Dangerous:
- Drop column
- Rename column
- Change column type
- Add NOT NULL to existing column
```

**Zero-Downtime Migrations**:
```sql
-- Adding NOT NULL column safely

-- Step 1: Add nullable column
ALTER TABLE users ADD COLUMN new_col VARCHAR(255);

-- Step 2: Backfill data
UPDATE users SET new_col = 'default' WHERE new_col IS NULL;

-- Step 3: Add constraint
ALTER TABLE users
ADD CONSTRAINT users_new_col_not_null
CHECK (new_col IS NOT NULL) NOT VALID;

-- Step 4: Validate (can run concurrently)
ALTER TABLE users
VALIDATE CONSTRAINT users_new_col_not_null;

-- Step 5: Add actual NOT NULL
ALTER TABLE users ALTER COLUMN new_col SET NOT NULL;
ALTER TABLE users DROP CONSTRAINT users_new_col_not_null;
```

**Adding Index Without Locking**:
```sql
-- PostgreSQL
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- Note: Takes longer but doesn't block writes
```

## Scaling Strategies

### Vertical Scaling
```
- More CPU, RAM, faster disks
- Simple, but has limits
- Good first step
```

### Read Replicas
```
         Writes
            │
        ┌───┴───┐
        │Primary│
        └───┬───┘
   Replication│
    ┌─────┬──┴──┬─────┐
    ▼     ▼     ▼     ▼
 Replica Replica Replica
    ▲     ▲     ▲
    └─────┼─────┘
        Reads

Good for: Read-heavy workloads
Challenge: Replication lag
```

### Sharding
```
         ┌─────────┐
         │ Router  │
         └────┬────┘
    ┌─────────┼─────────┐
    ▼         ▼         ▼
┌───────┐ ┌───────┐ ┌───────┐
│Shard 1│ │Shard 2│ │Shard 3│
│ A-H   │ │ I-P   │ │ Q-Z   │
└───────┘ └───────┘ └───────┘

Good for: Write-heavy, large datasets
Challenge: Cross-shard queries, rebalancing
```

**Shard Key Selection**:
```
Good shard keys:
- Even distribution
- Query locality (most queries hit one shard)
- Cardinality (many unique values)

Examples:
- user_id (for user-centric apps)
- tenant_id (for multi-tenant)
- geographic region

Avoid:
- Timestamps (hot spots)
- Low cardinality (uneven distribution)
```

## Database Design Checklist

### Schema Design
- [ ] Tables represent clear entities
- [ ] Appropriate normalization level
- [ ] Relationships properly defined
- [ ] Constraints enforce data integrity
- [ ] Appropriate data types used
- [ ] Naming conventions consistent

### Performance
- [ ] Indexes on frequently queried columns
- [ ] Indexes on foreign keys
- [ ] Composite indexes match query patterns
- [ ] No over-indexing
- [ ] Large text/blob columns separated
- [ ] Partitioning for large tables

### Data Integrity
- [ ] Primary keys on all tables
- [ ] Foreign keys for relationships
- [ ] NOT NULL where required
- [ ] CHECK constraints for validation
- [ ] UNIQUE constraints where needed
- [ ] Default values set

### Operations
- [ ] Backup strategy defined
- [ ] Migration plan documented
- [ ] Monitoring queries identified
- [ ] Growth projections considered
- [ ] Archival strategy planned

## Anti-Patterns

**Entity-Attribute-Value (EAV)**:
```
❌ Stores attributes as rows
   Hard to query, no type safety

✅ Use JSONB for flexible schemas
   Or proper normalized tables
```

**One Table to Rule Them All**:
```
❌ Huge table with many nullable columns
   Different entity types mixed together

✅ Separate tables for different entities
   Use inheritance or composition
```

**Storing Formatted Data**:
```
❌ Storing "$1,234.56" instead of 1234.56
   Storing "Jan 15, 2024" instead of timestamp

✅ Store raw data, format on display
```

**No Foreign Keys**:
```
❌ Relying on application to enforce relationships
   Orphaned records accumulate

✅ Use foreign keys with appropriate ON DELETE
```

---

**Remember**: Good database design is about understanding your data and how it will be used. Start with a normalized design, add indexes based on query patterns, and denormalize only when you have evidence it's needed.
