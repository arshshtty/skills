---
name: security-audit
description: Perform security audits using OWASP Top 10 principles. Check for SQL injection, XSS, CSRF, authentication issues, and other vulnerabilities. Apply when reviewing code or building security-sensitive features.
---

# Security Audit Skill

Systematically identify and fix security vulnerabilities in applications using industry-standard frameworks and best practices.

## Core Principles

### 1. **Defense in Depth**
- Multiple layers of security
- No single point of failure
- Assume every layer can be breached

### 2. **Least Privilege**
- Grant minimum necessary permissions
- Restrict access by default
- Escalate privileges only when needed

### 3. **Fail Securely**
- Secure defaults
- Graceful degradation
- Don't leak information in errors

### 4. **Never Trust User Input**
- Validate everything
- Sanitize all inputs
- Use parameterized queries
- Encode outputs

### 5. **Security by Design**
- Build security in from the start
- Not an afterthought
- Make secure path the easy path

## OWASP Top 10 Vulnerabilities

### 1. Broken Access Control

**What it is**: Users can access resources they shouldn't

**Common Issues**:
- Missing authorization checks
- Insecure direct object references (IDOR)
- Elevation of privilege
- Forced browsing to unauthorized pages

**How to Check**:
```
- Can users access other users' data by changing IDs in URLs?
- Are authorization checks performed on every request?
- Can regular users access admin endpoints?
- Are there horizontal privilege violations?
- Are file permissions correctly set?
```

**Example Vulnerability**:
```javascript
// ❌ VULNERABLE: No authorization check
app.get('/api/orders/:id', async (req, res) => {
  const order = await db.orders.findById(req.params.id)
  return res.json(order)
})

// ✅ SECURE: Verify user owns the order
app.get('/api/orders/:id', async (req, res) => {
  const order = await db.orders.findById(req.params.id)

  if (!order) {
    return res.status(404).json({ error: 'Order not found' })
  }

  if (order.userId !== req.user.id && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Access denied' })
  }

  return res.json(order)
})
```

**Fixes**:
- Implement authorization checks on every protected endpoint
- Use role-based access control (RBAC)
- Deny by default, allow explicitly
- Log access violations
- Never expose internal object IDs directly (use UUIDs or hash IDs)

### 2. Cryptographic Failures

**What it is**: Sensitive data exposed due to weak or missing encryption

**Common Issues**:
- Passwords stored in plaintext
- Weak hashing algorithms (MD5, SHA1)
- Hardcoded secrets in code
- Sensitive data sent over HTTP
- Weak encryption keys

**How to Check**:
```
- Are passwords hashed with strong algorithms (bcrypt, Argon2)?
- Is sensitive data encrypted at rest?
- Is HTTPS enforced everywhere?
- Are secrets in environment variables, not code?
- Are API keys and tokens stored securely?
- Is TLS version up to date (1.2+)?
```

**Example Vulnerability**:
```javascript
// ❌ VULNERABLE: Plaintext password storage
async function createUser(username, password) {
  await db.users.create({ username, password })
}

// ❌ VULNERABLE: Weak hashing
const hash = crypto.createHash('md5').update(password).digest('hex')

// ✅ SECURE: Strong password hashing
const bcrypt = require('bcrypt')
const saltRounds = 12

async function createUser(username, password) {
  const hashedPassword = await bcrypt.hash(password, saltRounds)
  await db.users.create({ username, password: hashedPassword })
}

// ✅ SECURE: Verify password
async function verifyPassword(username, password) {
  const user = await db.users.findOne({ username })
  const isValid = await bcrypt.compare(password, user.password)
  return isValid
}
```

**Fixes**:
- Use bcrypt, Argon2, or scrypt for password hashing
- Enforce HTTPS everywhere
- Store secrets in environment variables or secret managers
- Encrypt sensitive data at rest
- Use strong, random encryption keys
- Implement key rotation policies

### 3. Injection

**What it is**: Untrusted data sent to an interpreter as part of a command/query

**Types**:
- SQL Injection
- NoSQL Injection
- Command Injection
- LDAP Injection
- Template Injection

**How to Check**:
```
- Are all database queries parameterized?
- Is user input validated before use?
- Are ORM/query builders used instead of raw SQL?
- Are shell commands avoided or properly escaped?
- Is user input in templates escaped?
```

**SQL Injection Example**:
```javascript
// ❌ VULNERABLE: SQL injection
app.get('/users/:id', async (req, res) => {
  const query = `SELECT * FROM users WHERE id = ${req.params.id}`
  const user = await db.query(query)
  // Attack: /users/1 OR 1=1 -- dumps all users
})

// ✅ SECURE: Parameterized query
app.get('/users/:id', async (req, res) => {
  const query = 'SELECT * FROM users WHERE id = ?'
  const user = await db.query(query, [req.params.id])
})

// ✅ SECURE: ORM/Query Builder
app.get('/users/:id', async (req, res) => {
  const user = await db.users.findById(req.params.id)
})
```

**Command Injection Example**:
```javascript
// ❌ VULNERABLE: Command injection
const { exec } = require('child_process')
app.get('/convert', (req, res) => {
  exec(`convert ${req.query.file} output.pdf`, (err, stdout) => {
    // Attack: ?file=; rm -rf /
  })
})

// ✅ SECURE: Use libraries instead of shell commands
const sharp = require('sharp')
app.get('/convert', async (req, res) => {
  // Validate input
  const allowedFile = /^[a-zA-Z0-9_-]+\.(jpg|png)$/.test(req.query.file)
  if (!allowedFile) {
    return res.status(400).json({ error: 'Invalid file' })
  }

  // Use library, not shell
  await sharp(req.query.file).toFile('output.pdf')
})
```

**Fixes**:
- Always use parameterized queries/prepared statements
- Use ORMs and query builders
- Validate and sanitize all inputs
- Avoid shell commands (use libraries instead)
- Use whitelists for allowed values
- Escape special characters if concatenation unavoidable

### 4. Insecure Design

**What it is**: Fundamental flaws in architecture and design

**Common Issues**:
- Missing security requirements
- No threat modeling
- Insecure design patterns
- Lack of security controls in design phase

**How to Check**:
```
- Was threat modeling performed?
- Are security requirements documented?
- Is there a secure SDLC process?
- Are security patterns used (e.g., secure session management)?
- Is input validation consistent across the app?
```

**Example**:
```javascript
// ❌ INSECURE DESIGN: Unlimited password reset attempts
app.post('/forgot-password', async (req, res) => {
  const user = await db.users.findOne({ email: req.body.email })
  if (user) {
    sendResetEmail(user)
  }
  res.json({ message: 'If email exists, reset link sent' })
})

// ✅ SECURE DESIGN: Rate limiting + token expiration
const rateLimit = require('express-rate-limit')

const resetLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 3, // 3 attempts per 15 minutes
  message: 'Too many reset attempts'
})

app.post('/forgot-password', resetLimiter, async (req, res) => {
  const user = await db.users.findOne({ email: req.body.email })
  if (user) {
    const token = generateSecureToken()
    const expiresAt = Date.now() + 1000 * 60 * 60 // 1 hour

    await db.resetTokens.create({
      userId: user.id,
      token: hashToken(token),
      expiresAt
    })

    sendResetEmail(user, token)
  }
  // Same response whether user exists or not (prevents enumeration)
  res.json({ message: 'If email exists, reset link sent' })
})
```

**Fixes**:
- Perform threat modeling
- Document security requirements
- Use established security patterns
- Implement rate limiting
- Design for security from the start

### 5. Security Misconfiguration

**What it is**: Insecure default configurations, incomplete setup, open cloud storage

**Common Issues**:
- Default credentials still in use
- Unnecessary features enabled
- Detailed error messages exposing internals
- Missing security headers
- Outdated software
- Open cloud buckets

**How to Check**:
```
- Are default passwords changed?
- Are unnecessary services disabled?
- Are error messages generic in production?
- Are security headers configured (CSP, HSTS, X-Frame-Options)?
- Are all dependencies up to date?
- Is directory listing disabled?
- Are admin interfaces restricted?
```

**Example**:
```javascript
// ❌ VULNERABLE: Detailed error in production
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.message,
    stack: err.stack,
    query: req.query
  })
})

// ✅ SECURE: Generic error, log details
app.use((err, req, res, next) => {
  // Log full details server-side
  logger.error({
    message: err.message,
    stack: err.stack,
    url: req.url,
    user: req.user?.id
  })

  // Send generic error to client
  res.status(500).json({
    error: 'Internal server error',
    requestId: req.id
  })
})

// ✅ SECURE: Security headers
const helmet = require('helmet')
app.use(helmet())
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    scriptSrc: ["'self'"],
    imgSrc: ["'self'", 'data:', 'https:']
  }
}))
```

**Fixes**:
- Use security-hardened configurations
- Disable unnecessary features
- Keep software updated
- Configure security headers
- Generic errors in production, detailed logs server-side
- Regular security audits of configuration

### 6. Vulnerable and Outdated Components

**What it is**: Using libraries with known vulnerabilities

**Common Issues**:
- Outdated dependencies
- Unused dependencies
- Dependencies with known CVEs
- No dependency update process

**How to Check**:
```bash
# Check for vulnerabilities (language-specific examples)
npm audit
pip check
bundle audit
composer audit

# Check for outdated packages
npm outdated
pip list --outdated
```

**Automated Checking**:
```json
// package.json - Add audit to CI/CD
{
  "scripts": {
    "audit": "npm audit --audit-level=moderate",
    "audit:fix": "npm audit fix"
  }
}
```

**Fixes**:
- Regularly update dependencies
- Use automated dependency checking (Dependabot, Snyk)
- Remove unused dependencies
- Pin versions to prevent unexpected updates
- Monitor security advisories
- Have update policy (critical patches within 48 hours)

### 7. Identification and Authentication Failures

**What it is**: Weak authentication, session management issues

**Common Issues**:
- Weak password requirements
- Missing brute force protection
- Session fixation
- Exposed session IDs
- Insecure credential storage
- Missing multi-factor authentication

**How to Check**:
```
- Are passwords required to be strong?
- Is there brute force protection (rate limiting, account lockout)?
- Are sessions invalidated on logout?
- Are session IDs regenerated after login?
- Is MFA available for sensitive accounts?
- Are sessions timed out after inactivity?
```

**Example**:
```javascript
// ❌ VULNERABLE: No password requirements, no rate limiting
app.post('/login', async (req, res) => {
  const { username, password } = req.body
  const user = await db.users.findOne({ username })

  if (user && user.password === password) {
    req.session.userId = user.id
    return res.json({ success: true })
  }

  return res.status(401).json({ error: 'Invalid credentials' })
})

// ✅ SECURE: Strong passwords, rate limiting, hashing
const rateLimit = require('express-rate-limit')
const bcrypt = require('bcrypt')

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: 'Too many login attempts'
})

app.post('/login', loginLimiter, async (req, res) => {
  const { username, password } = req.body

  // Constant-time lookup to prevent timing attacks
  const user = await db.users.findOne({ username })

  if (!user) {
    // Fake hash to prevent timing attacks
    await bcrypt.compare(password, '$2b$12$fake.hash.value')
    return res.status(401).json({ error: 'Invalid credentials' })
  }

  const isValid = await bcrypt.compare(password, user.password)

  if (!isValid) {
    // Log failed attempt
    await db.loginAttempts.create({ userId: user.id, success: false })
    return res.status(401).json({ error: 'Invalid credentials' })
  }

  // Regenerate session ID to prevent session fixation
  req.session.regenerate((err) => {
    if (err) return next(err)

    req.session.userId = user.id
    req.session.loginAt = Date.now()

    // Log successful login
    db.loginAttempts.create({ userId: user.id, success: true })

    res.json({ success: true })
  })
})

// Session timeout
app.use((req, res, next) => {
  if (req.session.userId) {
    const maxAge = 30 * 60 * 1000 // 30 minutes
    if (Date.now() - req.session.loginAt > maxAge) {
      req.session.destroy()
      return res.status(401).json({ error: 'Session expired' })
    }
  }
  next()
})
```

**Password Requirements**:
```javascript
function validatePassword(password) {
  const minLength = 12
  const hasUppercase = /[A-Z]/.test(password)
  const hasLowercase = /[a-z]/.test(password)
  const hasNumber = /[0-9]/.test(password)
  const hasSpecial = /[!@#$%^&*]/.test(password)

  if (password.length < minLength) {
    return { valid: false, error: 'Password must be at least 12 characters' }
  }

  if (!hasUppercase || !hasLowercase || !hasNumber || !hasSpecial) {
    return {
      valid: false,
      error: 'Password must contain uppercase, lowercase, number, and special character'
    }
  }

  return { valid: true }
}
```

**Fixes**:
- Enforce strong password policies
- Implement rate limiting on login
- Use bcrypt/Argon2 for password hashing
- Regenerate session IDs after login
- Implement session timeouts
- Offer multi-factor authentication
- Log authentication events

### 8. Software and Data Integrity Failures

**What it is**: Code/infrastructure without integrity verification

**Common Issues**:
- Insecure CI/CD pipelines
- Auto-updates without verification
- Unsigned code
- Insecure deserialization
- No supply chain security

**How to Check**:
```
- Are dependencies verified with checksums/signatures?
- Is code review required before deployment?
- Are CI/CD pipelines secured?
- Is deserialization of untrusted data avoided?
- Are digital signatures used for releases?
```

**Example**:
```javascript
// ❌ VULNERABLE: Deserializing untrusted data
app.post('/api/data', (req, res) => {
  const data = JSON.parse(req.body.data)
  // If using eval, pickle, or other unsafe deserialization
  eval(data.code) // Extremely dangerous!
})

// ✅ SECURE: Validate structure before processing
const Joi = require('joi')

const schema = Joi.object({
  name: Joi.string().max(100).required(),
  value: Joi.number().min(0).max(1000).required()
})

app.post('/api/data', (req, res) => {
  const { error, value } = schema.validate(req.body)

  if (error) {
    return res.status(400).json({ error: error.details[0].message })
  }

  // Process validated data
  processData(value)
})
```

**Fixes**:
- Verify package checksums/signatures
- Use lock files (package-lock.json, Pipfile.lock)
- Secure CI/CD pipelines
- Code review all changes
- Avoid deserializing untrusted data
- Sign releases

### 9. Security Logging and Monitoring Failures

**What it is**: Insufficient logging, no monitoring, delayed detection

**Common Issues**:
- No logging of security events
- Logs lack context
- No alerting on suspicious activity
- Logs not reviewed
- Insufficient log retention

**What to Log**:
```
✅ Authentication (login, logout, failures)
✅ Authorization failures
✅ Input validation failures
✅ Application errors
✅ Security-relevant config changes
✅ Access to sensitive data
✅ Administrative actions

❌ Passwords or secrets
❌ Session tokens
❌ Credit card numbers
❌ Excessive PII
```

**Example**:
```javascript
const winston = require('winston')

const logger = winston.createLogger({
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'security.log' })
  ]
})

// Log authentication events
app.post('/login', async (req, res) => {
  const { username, password } = req.body
  const user = await db.users.findOne({ username })

  if (user && await verifyPassword(password, user.password)) {
    logger.info('Login successful', {
      userId: user.id,
      username: user.username,
      ip: req.ip,
      userAgent: req.get('user-agent'),
      timestamp: new Date().toISOString()
    })

    // Login logic...
  } else {
    logger.warn('Login failed', {
      username: username, // OK to log username
      ip: req.ip,
      reason: 'Invalid credentials',
      timestamp: new Date().toISOString()
    })

    // Rate limiting check
    const attempts = await getFailedAttempts(req.ip)
    if (attempts > 5) {
      logger.error('Possible brute force attack', {
        ip: req.ip,
        attempts: attempts
      })
    }
  }
})

// Log authorization failures
function requireAdmin(req, res, next) {
  if (!req.user.isAdmin) {
    logger.warn('Authorization failure', {
      userId: req.user.id,
      resource: req.path,
      ip: req.ip
    })
    return res.status(403).json({ error: 'Access denied' })
  }
  next()
}
```

**Fixes**:
- Log all security-relevant events
- Include context (user, IP, timestamp, action)
- Implement real-time alerting
- Regular log review
- Centralized logging
- Retain logs appropriately (30-90 days minimum)

### 10. Server-Side Request Forgery (SSRF)

**What it is**: Application fetches remote resource without validating URL

**Common Issues**:
- User-controlled URLs
- No URL whitelist
- Access to internal resources
- Cloud metadata access

**How to Check**:
```
- Are user-provided URLs validated?
- Is there a whitelist of allowed domains?
- Are internal IPs blocked?
- Is access to cloud metadata prevented?
```

**Example**:
```javascript
// ❌ VULNERABLE: SSRF attack
app.get('/fetch', async (req, res) => {
  const response = await fetch(req.query.url)
  const data = await response.text()
  res.send(data)
  // Attack: ?url=http://169.254.169.254/latest/meta-data/
  // Exposes cloud instance metadata
})

// ✅ SECURE: Validate and whitelist URLs
const validDomains = ['api.example.com', 'cdn.example.com']

app.get('/fetch', async (req, res) => {
  const url = new URL(req.query.url)

  // Check against whitelist
  if (!validDomains.includes(url.hostname)) {
    return res.status(400).json({ error: 'Invalid domain' })
  }

  // Block internal IPs
  const ip = await dns.lookup(url.hostname)
  if (isPrivateIP(ip)) {
    return res.status(400).json({ error: 'Access to internal resources denied' })
  }

  // Only allow HTTPS
  if (url.protocol !== 'https:') {
    return res.status(400).json({ error: 'Only HTTPS allowed' })
  }

  const response = await fetch(url.toString())
  const data = await response.text()
  res.send(data)
})

function isPrivateIP(ip) {
  const parts = ip.split('.').map(Number)
  return (
    parts[0] === 10 ||
    parts[0] === 127 ||
    (parts[0] === 172 && parts[1] >= 16 && parts[1] <= 31) ||
    (parts[0] === 192 && parts[1] === 168) ||
    ip === '169.254.169.254' // AWS metadata
  )
}
```

**Fixes**:
- Whitelist allowed domains
- Block internal IP ranges
- Validate and sanitize URLs
- Disable redirects or validate redirect targets
- Use separate network for external requests

## Additional Security Concerns

### Cross-Site Scripting (XSS)

**Types**:
- **Stored XSS**: Malicious script stored in database
- **Reflected XSS**: Script in URL reflected in response
- **DOM XSS**: Client-side script manipulation

**Example**:
```javascript
// ❌ VULNERABLE: XSS via unescaped output
app.get('/search', (req, res) => {
  res.send(`<h1>Results for: ${req.query.q}</h1>`)
  // Attack: ?q=<script>alert('XSS')</script>
})

// ✅ SECURE: Escape output
const escapeHtml = require('escape-html')

app.get('/search', (req, res) => {
  const query = escapeHtml(req.query.q)
  res.send(`<h1>Results for: ${query}</h1>`)
})

// ✅ SECURE: Use templating with auto-escaping
app.get('/search', (req, res) => {
  res.render('search', { query: req.query.q })
  // Template engine auto-escapes by default
})
```

**Fixes**:
- Escape all user input in HTML contexts
- Use Content Security Policy (CSP)
- Use templating engines with auto-escaping
- Validate input
- Use httpOnly cookies for sessions

### Cross-Site Request Forgery (CSRF)

**What it is**: Attacker tricks user's browser into making unwanted requests

**Example**:
```javascript
// ❌ VULNERABLE: No CSRF protection
app.post('/transfer', (req, res) => {
  const { to, amount } = req.body
  transfer(req.user.id, to, amount)
})
// Attack: Malicious site submits form to /transfer

// ✅ SECURE: CSRF tokens
const csrf = require('csurf')
const csrfProtection = csrf({ cookie: true })

app.get('/transfer', csrfProtection, (req, res) => {
  res.render('transfer', { csrfToken: req.csrfToken() })
})

app.post('/transfer', csrfProtection, (req, res) => {
  // Token is automatically validated
  const { to, amount } = req.body
  transfer(req.user.id, to, amount)
})
```

**Fixes**:
- Use CSRF tokens for state-changing operations
- SameSite cookie attribute
- Verify Origin/Referer headers
- Re-authenticate for sensitive actions

## Security Audit Checklist

### Authentication & Authorization
- [ ] Passwords hashed with bcrypt/Argon2
- [ ] Strong password requirements enforced
- [ ] Rate limiting on login endpoints
- [ ] Session IDs regenerated after login
- [ ] Session timeout implemented
- [ ] Authorization checked on every protected endpoint
- [ ] MFA available for admin accounts

### Input Validation
- [ ] All user input validated
- [ ] Parameterized queries used (no SQL injection)
- [ ] Input length limits enforced
- [ ] Whitelists used over blacklists
- [ ] File upload validation (type, size, content)

### Output Encoding
- [ ] HTML output escaped
- [ ] JSON responses properly encoded
- [ ] Headers sanitized

### Cryptography
- [ ] HTTPS enforced everywhere
- [ ] TLS 1.2+ used
- [ ] Secrets in environment variables
- [ ] Strong encryption algorithms used
- [ ] Random values cryptographically secure

### Configuration
- [ ] Default credentials changed
- [ ] Unnecessary features disabled
- [ ] Security headers configured (CSP, HSTS, X-Frame-Options)
- [ ] Error messages generic in production
- [ ] Dependencies up to date
- [ ] CORS configured properly

### Logging & Monitoring
- [ ] Authentication events logged
- [ ] Authorization failures logged
- [ ] Errors logged with context
- [ ] Sensitive data not logged
- [ ] Alerts configured for suspicious activity

### Application Security
- [ ] CSRF protection on state-changing operations
- [ ] XSS protection (output escaping, CSP)
- [ ] SSRF prevention (URL validation)
- [ ] File upload restrictions
- [ ] Rate limiting on expensive operations
- [ ] Audit trail for sensitive actions

---

**Remember**: Security is not a checkbox—it's an ongoing process. Regular audits, staying updated on threats, and building security into your development workflow are essential for maintaining a secure application.
