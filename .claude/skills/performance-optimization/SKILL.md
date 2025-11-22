---
name: performance-optimization
description: Optimize application performance through profiling, benchmarking, caching, and resource management. Apply when addressing slow response times, high memory usage, or scalability concerns.
---

# Performance Optimization Skill

Systematically identify and resolve performance bottlenecks to build fast, efficient, and scalable applications.

## Core Principles

### 1. **Measure First**
- Profile before optimizing
- Establish baseline metrics
- Use data, not assumptions
- Focus on actual bottlenecks

### 2. **Optimize the Right Things**
- 80/20 rule: 20% of code causes 80% of issues
- Premature optimization is the root of all evil
- Optimize hot paths first
- Consider trade-offs (readability vs speed)

### 3. **Validate Improvements**
- Benchmark before and after
- Test under realistic conditions
- Monitor in production
- Watch for regressions

## Performance Profiling

### Types of Profiling

**CPU Profiling**
- Identify functions consuming most CPU time
- Find hot spots and tight loops
- Detect unnecessary computations

**Memory Profiling**
- Track heap allocations
- Find memory leaks
- Identify excessive object creation

**I/O Profiling**
- Database query times
- Network latency
- File system operations
- External API calls

### Profiling Process

```
1. Identify the symptom (slow page, high CPU, memory growth)
2. Reproduce under controlled conditions
3. Profile to find bottleneck
4. Hypothesize root cause
5. Implement fix
6. Measure improvement
7. Verify no regressions
```

### Key Metrics to Track

**Response Time Percentiles**
```
p50 (median): 200ms  - Typical user experience
p95:          500ms  - Most users' worst experience
p99:          1000ms - Outlier experience
```

**Resource Utilization**
- CPU usage (user, system, idle)
- Memory (heap, RSS, available)
- Disk I/O (reads/writes, latency)
- Network (bandwidth, connections)

**Application Metrics**
- Requests per second (throughput)
- Error rate
- Queue depth
- Cache hit ratio

## Common Bottlenecks & Solutions

### 1. Database Performance

**Symptoms**: Slow queries, high database CPU

**Common Issues**:
- Missing indexes
- N+1 query problem
- Full table scans
- Lock contention
- Inefficient queries

**Solutions**:

**Add Appropriate Indexes**
```sql
-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 123;

-- Add index for frequently filtered columns
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Composite index for multiple conditions
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Partial index for specific values
CREATE INDEX idx_orders_pending ON orders(created_at)
WHERE status = 'pending';
```

**Fix N+1 Queries**
```
Problem:
  SELECT * FROM posts LIMIT 10
  SELECT * FROM users WHERE id = 1  -- For each post
  SELECT * FROM users WHERE id = 2
  ... (10 additional queries)

Solution - Eager Loading:
  SELECT * FROM posts LIMIT 10
  SELECT * FROM users WHERE id IN (1, 2, 3, ...)  -- Single query

Or use JOINs:
  SELECT posts.*, users.name
  FROM posts
  JOIN users ON posts.user_id = users.id
  LIMIT 10
```

**Optimize Query Patterns**
```sql
-- Bad: SELECT * when you need few columns
SELECT * FROM users WHERE id = 1;

-- Good: Select only needed columns
SELECT id, name, email FROM users WHERE id = 1;

-- Bad: LIKE with leading wildcard (can't use index)
SELECT * FROM products WHERE name LIKE '%phone%';

-- Good: Full-text search or prefix match
SELECT * FROM products WHERE name LIKE 'phone%';
-- Or use full-text search index
```

### 2. Memory Issues

**Symptoms**: Growing memory usage, OOM errors, GC pauses

**Common Issues**:
- Memory leaks
- Large object allocations
- Unbounded caches
- Retained references

**Solutions**:

**Find and Fix Memory Leaks**
```
Common leak sources:
- Event listeners not removed
- Closures holding references
- Global caches without eviction
- Timers/intervals not cleared
- Circular references

Detection approach:
1. Take heap snapshot
2. Perform operations
3. Take another snapshot
4. Compare - look for growing objects
5. Trace back to source
```

**Bounded Caches**
```
Problem: Unbounded cache grows forever
cache = {}
cache[key] = value  // Never removed

Solution: LRU cache with max size
- Set maximum entries
- Evict least recently used
- Consider TTL for time-sensitive data
```

**Reduce Object Allocations**
```
Problem: Creating objects in hot loops

Solution:
- Object pooling for frequently created objects
- Reuse buffers instead of allocating new ones
- Use primitives instead of wrapper objects
- Avoid unnecessary intermediate objects
```

### 3. Network & I/O

**Symptoms**: High latency, timeout errors, slow file operations

**Common Issues**:
- Sequential requests that could be parallel
- Large payloads
- Missing compression
- No connection pooling
- Chatty protocols

**Solutions**:

**Parallelize Independent Requests**
```
Sequential (slow):
  result1 = await fetch('/api/users')      // 100ms
  result2 = await fetch('/api/products')   // 100ms
  result3 = await fetch('/api/orders')     // 100ms
  // Total: 300ms

Parallel (fast):
  [result1, result2, result3] = await Promise.all([
    fetch('/api/users'),
    fetch('/api/products'),
    fetch('/api/orders')
  ])
  // Total: ~100ms (limited by slowest)
```

**Reduce Payload Size**
```
- Enable gzip/brotli compression
- Paginate large results
- Use field selection (GraphQL, sparse fieldsets)
- Optimize images (WebP, responsive images)
- Minify and bundle assets
```

**Connection Pooling**
```
Problem: Creating new connection per request
  for request in requests:
    conn = createConnection()  // Expensive
    conn.query(...)
    conn.close()

Solution: Reuse connections from pool
  pool = createPool(min=5, max=20)
  for request in requests:
    conn = pool.acquire()  // Fast - reuses existing
    conn.query(...)
    pool.release(conn)
```

### 4. CPU-Bound Operations

**Symptoms**: High CPU usage, slow computations

**Common Issues**:
- Inefficient algorithms
- Unnecessary computations
- Blocking operations
- Missing memoization

**Solutions**:

**Algorithm Optimization**
```
O(n²) → O(n log n) or O(n)

Example - Finding duplicates:

  O(n²) - Nested loops:
  for i in range(len(arr)):
    for j in range(i+1, len(arr)):
      if arr[i] == arr[j]:
        # duplicate found

  O(n) - Hash set:
  seen = set()
  for item in arr:
    if item in seen:
      # duplicate found
    seen.add(item)
```

**Memoization/Caching**
```
Problem: Recalculating expensive results

Solution: Cache computation results
  cache = {}

  def expensive_calculation(input):
    if input in cache:
      return cache[input]

    result = # ... expensive work
    cache[input] = result
    return result
```

**Offload Heavy Work**
```
- Move to background jobs/workers
- Use worker threads for CPU-intensive tasks
- Consider compiled languages for hot paths
- Pre-compute during off-peak times
```

## Caching Strategies

### Cache Hierarchy

```
Fastest → Slowest:
1. CPU cache (nanoseconds)
2. In-memory/application cache (microseconds)
3. Distributed cache - Redis/Memcached (milliseconds)
4. CDN cache (tens of milliseconds)
5. Database cache (tens of milliseconds)
6. Disk (milliseconds)
7. Network/API calls (hundreds of milliseconds)
```

### Caching Patterns

**Cache-Aside (Lazy Loading)**
```
1. Check cache
2. If miss, load from source
3. Store in cache
4. Return data

Pros: Only caches what's needed
Cons: Cache miss penalty, potential stale data
```

**Write-Through**
```
1. Write to cache
2. Cache writes to database
3. Return success

Pros: Cache always consistent
Cons: Write latency, cache may have unused data
```

**Write-Behind (Write-Back)**
```
1. Write to cache
2. Return immediately
3. Async write to database

Pros: Fast writes
Cons: Risk of data loss, complexity
```

### Cache Invalidation Strategies

**Time-Based (TTL)**
```
- Set expiration time
- Simple to implement
- May serve stale data until expiry
- Good for: Data that changes infrequently
```

**Event-Based**
```
- Invalidate on data change
- Always fresh data
- More complex to implement
- Good for: Data that must be current
```

**Version-Based**
```
- Include version in cache key
- New version = new cache entry
- Old versions naturally expire
- Good for: Assets, configurations
```

### What to Cache

**Good Candidates**:
- Expensive computations
- Frequent database queries
- External API responses
- Static assets
- Session data
- Aggregated/computed data

**Poor Candidates**:
- Rapidly changing data
- User-specific data (unless segmented)
- Large objects (memory pressure)
- Security-sensitive data
- Data that must be real-time

## Frontend Performance

### Critical Rendering Path

**Optimize Loading**:
```
1. Minimize critical resources
2. Minimize critical path length
3. Minimize critical bytes

Techniques:
- Inline critical CSS
- Async/defer non-critical JS
- Preload critical assets
- Lazy load below-fold content
```

### Bundle Optimization

**Code Splitting**
```
- Split by route/page
- Split by component (lazy loading)
- Separate vendor bundles
- Dynamic imports for heavy libraries
```

**Tree Shaking**
```
- Remove unused code
- Use ES modules (import/export)
- Avoid side effects in modules
- Check bundle analyzer for dead code
```

**Asset Optimization**
```
Images:
- Use modern formats (WebP, AVIF)
- Responsive images (srcset)
- Lazy loading
- Compression

JavaScript:
- Minification
- Compression (gzip/brotli)
- Remove source maps in production

CSS:
- Minification
- Remove unused styles
- Critical CSS inlining
```

### Core Web Vitals

**LCP (Largest Contentful Paint)** - < 2.5s
```
- Optimize server response time
- Preload critical resources
- Optimize images
- Remove render-blocking resources
```

**FID (First Input Delay)** - < 100ms
```
- Break up long tasks
- Use web workers for heavy computation
- Minimize JavaScript execution
- Optimize event handlers
```

**CLS (Cumulative Layout Shift)** - < 0.1
```
- Set dimensions on images/videos
- Reserve space for dynamic content
- Avoid inserting content above existing
- Use transform for animations
```

## Database Optimization

### Query Optimization

**Use EXPLAIN/ANALYZE**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders
WHERE user_id = 123 AND status = 'pending';

-- Look for:
-- - Seq Scan (full table scan) → Add index
-- - High row estimates → Statistics outdated
-- - Nested loops with large tables → Consider hash join
```

**Index Strategy**
```
When to add index:
- Columns in WHERE clauses
- Columns in JOIN conditions
- Columns in ORDER BY
- Columns in GROUP BY

When NOT to index:
- Small tables
- Columns with low cardinality
- Frequently updated columns
- Columns rarely used in queries
```

### Connection Management

**Pool Sizing**
```
connections = (core_count * 2) + effective_spindle_count

For SSD: connections ≈ core_count * 2
For cloud: Start small (10-20), increase based on metrics

Too few: Requests queue up
Too many: Context switching overhead, memory pressure
```

### Query Patterns

**Pagination**
```sql
-- Offset pagination (simple but slow for large offsets)
SELECT * FROM products ORDER BY id LIMIT 20 OFFSET 10000;

-- Keyset pagination (fast, consistent)
SELECT * FROM products
WHERE id > 10000
ORDER BY id
LIMIT 20;
```

**Batch Operations**
```sql
-- Bad: Individual inserts
INSERT INTO logs (message) VALUES ('log 1');
INSERT INTO logs (message) VALUES ('log 2');
-- ... 1000 times

-- Good: Batch insert
INSERT INTO logs (message) VALUES
  ('log 1'), ('log 2'), ... ('log 1000');
```

## Performance Testing

### Types of Tests

**Load Testing**
- Test under expected load
- Find performance baseline
- Identify bottlenecks

**Stress Testing**
- Test beyond normal capacity
- Find breaking points
- Verify recovery behavior

**Soak Testing**
- Extended duration under load
- Find memory leaks
- Identify gradual degradation

**Spike Testing**
- Sudden traffic increase
- Test auto-scaling
- Verify graceful handling

### Benchmarking Best Practices

```
1. Isolate the test environment
2. Use realistic data volumes
3. Warm up before measuring
4. Run multiple iterations
5. Measure percentiles, not just averages
6. Account for variance
7. Test one change at a time
8. Document test conditions
```

### Performance Budgets

**Set Limits**:
```
- Page load time: < 3 seconds
- Time to interactive: < 5 seconds
- Bundle size: < 200KB (gzipped)
- API response time: < 200ms (p95)
- Database query: < 50ms
```

**Enforce in CI/CD**:
- Fail build if budget exceeded
- Alert on performance regression
- Track trends over time

## Optimization Checklist

### Quick Wins
- [ ] Enable compression (gzip/brotli)
- [ ] Add database indexes for slow queries
- [ ] Implement caching for repeated operations
- [ ] Enable HTTP/2
- [ ] Use CDN for static assets
- [ ] Optimize images
- [ ] Minify and bundle assets

### Database
- [ ] Analyze slow query log
- [ ] Review and optimize indexes
- [ ] Fix N+1 queries
- [ ] Implement connection pooling
- [ ] Use appropriate data types
- [ ] Partition large tables

### Application
- [ ] Profile CPU and memory usage
- [ ] Cache expensive computations
- [ ] Parallelize independent operations
- [ ] Use efficient algorithms and data structures
- [ ] Implement pagination
- [ ] Offload heavy work to background jobs

### Frontend
- [ ] Optimize critical rendering path
- [ ] Code split and lazy load
- [ ] Minimize bundle size
- [ ] Optimize Core Web Vitals
- [ ] Use efficient event handlers
- [ ] Implement virtual scrolling for long lists

### Infrastructure
- [ ] Right-size servers
- [ ] Configure auto-scaling
- [ ] Use appropriate instance types
- [ ] Optimize network topology
- [ ] Implement load balancing
- [ ] Monitor and alert on performance

## Anti-Patterns

**Premature Optimization**
- Don't optimize without profiling
- Don't sacrifice readability for micro-optimizations
- Don't optimize rarely-executed code

**Over-Caching**
- Don't cache everything
- Don't ignore cache invalidation
- Don't cache sensitive data inappropriately

**Ignoring Fundamentals**
- Don't skip database indexes
- Don't ignore algorithm complexity
- Don't forget about network latency

**Not Measuring**
- Don't assume you know the bottleneck
- Don't skip baseline measurements
- Don't ignore production metrics

---

**Remember**: Performance optimization is iterative. Measure, identify the biggest bottleneck, fix it, then repeat. Focus on changes that provide the most impact for your users.
