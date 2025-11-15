---
name: security-auditor
description: Performs security audits to identify vulnerabilities including OWASP Top 10, authentication issues, injection attacks, and insecure dependencies. Apply when reviewing security-critical code or conducting security assessments.
---

# Security Auditor Skill

Systematically identify and remediate security vulnerabilities in your codebase to protect against attacks and data breaches.

## When to Use

✅ **Before deploying to production**: Security audit before launch
✅ **Authentication/authorization code**: Review auth flows
✅ **API endpoints**: Check for security issues
✅ **Payment processing**: Audit financial transactions
✅ **User input handling**: Validate input sanitization
✅ **Dependency updates**: Check for vulnerabilities
✅ **After security incident**: Post-mortem analysis
✅ **Handling sensitive data**: PII, passwords, tokens
✅ **Regular security reviews**: Monthly/quarterly audits

## Security Mindset

**Think like an attacker**:
- What's the worst that could happen?
- How can I bypass this protection?
- What input would break this?
- Where's the sensitive data flowing?
- What happens if auth fails?

**Defense in depth**:
- Multiple layers of security
- Assume every layer can fail
- Validate at boundaries
- Fail securely

## OWASP Top 10 (2021)

### 1. Broken Access Control

**What**: Users can access resources they shouldn't

**Common issues**:
- Missing authorization checks
- Insecure direct object references (IDOR)
- Privilege escalation
- Exposed admin endpoints

**Example vulnerability**:
```typescript
// ❌ No authorization check
app.delete('/api/users/:id', async (req, res) => {
  await db.users.delete(req.params.id)
  res.sendStatus(204)
})
// Any authenticated user can delete ANY user!
```

**Fix**:
```typescript
// ✅ Check authorization
app.delete('/api/users/:id', requireAuth, async (req, res) => {
  const userId = req.params.id
  const currentUser = req.user

  // Check if user owns this resource or is admin
  if (currentUser.id !== userId && !currentUser.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' })
  }

  await db.users.delete(userId)
  res.sendStatus(204)
})
```

**IDOR Example**:
```typescript
// ❌ Predictable IDs, no ownership check
app.get('/api/orders/:id', async (req, res) => {
  const order = await db.orders.findById(req.params.id)
  res.json(order)
})
// User can access /api/orders/1, /api/orders/2, etc.
```

**Fix**:
```typescript
// ✅ Check ownership
app.get('/api/orders/:id', requireAuth, async (req, res) => {
  const order = await db.orders.findById(req.params.id)

  if (!order) {
    return res.status(404).json({ error: 'Not found' })
  }

  // Verify user owns this order
  if (order.userId !== req.user.id) {
    return res.status(403).json({ error: 'Forbidden' })
  }

  res.json(order)
})
```

**Checklist**:
- [ ] Every endpoint checks authentication
- [ ] Every resource access checks authorization
- [ ] No predictable sequential IDs for sensitive resources
- [ ] Admin functions require admin role
- [ ] Users can't elevate their own privileges
- [ ] Default deny (whitelist, not blacklist)

### 2. Cryptographic Failures

**What**: Weak or missing encryption/hashing

**Common issues**:
- Plaintext passwords
- Weak hashing algorithms (MD5, SHA1)
- Hardcoded secrets
- Insecure random number generation
- Missing encryption for sensitive data

**Example vulnerability**:
```typescript
// ❌ Plaintext password storage
async function createUser(email, password) {
  await db.users.create({
    email,
    password // Stored as plain text!
  })
}

// ❌ Weak hashing
const crypto = require('crypto')
const hash = crypto.createHash('md5').update(password).digest('hex')
```

**Fix**:
```typescript
// ✅ Use bcrypt for password hashing
import bcrypt from 'bcrypt'

async function createUser(email, password) {
  const saltRounds = 12
  const hashedPassword = await bcrypt.hash(password, saltRounds)

  await db.users.create({
    email,
    password: hashedPassword
  })
}

// ✅ Verify password
async function verifyPassword(email, password) {
  const user = await db.users.findByEmail(email)
  if (!user) return false

  return await bcrypt.compare(password, user.password)
}
```

**Secrets management**:
```typescript
// ❌ Hardcoded secret
const API_KEY = 'sk_live_abc123def456'

// ❌ Committed .env file
// .env file committed to git

// ✅ Environment variables
const API_KEY = process.env.API_KEY
if (!API_KEY) {
  throw new Error('API_KEY environment variable is required')
}

// ✅ Secrets manager (production)
import { SecretsManager } from 'aws-sdk'
const secret = await secretsManager.getSecretValue({ SecretId: 'api-key' })
```

**Random number generation**:
```typescript
// ❌ Insecure randomness
const token = Math.random().toString(36).substring(7)

// ✅ Cryptographically secure
import crypto from 'crypto'
const token = crypto.randomBytes(32).toString('hex')
```

**Checklist**:
- [ ] Passwords hashed with bcrypt/argon2 (not MD5/SHA1)
- [ ] Secrets in environment variables, not code
- [ ] Sensitive data encrypted at rest
- [ ] HTTPS/TLS for data in transit
- [ ] Crypto.randomBytes for tokens/IDs
- [ ] No secrets in git history
- [ ] Strong encryption algorithms (AES-256)

### 3. Injection

**What**: Untrusted data sent to interpreter as command/query

**Types**: SQL, NoSQL, OS command, LDAP, XPath

**SQL Injection**:
```typescript
// ❌ SQL injection vulnerability
app.get('/users', async (req, res) => {
  const name = req.query.name
  const sql = `SELECT * FROM users WHERE name = '${name}'`
  const users = await db.query(sql)
  res.json(users)
})
// Attack: /users?name=admin' OR '1'='1
// Query: SELECT * FROM users WHERE name = 'admin' OR '1'='1'
```

**Fix**:
```typescript
// ✅ Parameterized query
app.get('/users', async (req, res) => {
  const name = req.query.name
  const sql = 'SELECT * FROM users WHERE name = ?'
  const users = await db.query(sql, [name])
  res.json(users)
})

// ✅ Or use ORM
const users = await db.users.findAll({
  where: { name: req.query.name }
})
```

**NoSQL Injection**:
```typescript
// ❌ NoSQL injection
app.post('/login', async (req, res) => {
  const user = await db.users.findOne({
    email: req.body.email,
    password: req.body.password
  })
})
// Attack: { "email": "admin@example.com", "password": { "$ne": null } }
```

**Fix**:
```typescript
// ✅ Validate input types
app.post('/login', async (req, res) => {
  const { email, password } = req.body

  // Ensure strings (not objects)
  if (typeof email !== 'string' || typeof password !== 'string') {
    return res.status(400).json({ error: 'Invalid input' })
  }

  // Hash password for comparison
  const user = await db.users.findOne({ email })
  if (!user) return res.status(401).json({ error: 'Invalid credentials' })

  const valid = await bcrypt.compare(password, user.password)
  if (!valid) return res.status(401).json({ error: 'Invalid credentials' })

  res.json({ token: generateToken(user) })
})
```

**Command Injection**:
```typescript
// ❌ Command injection
app.get('/ping', (req, res) => {
  const host = req.query.host
  exec(`ping -c 4 ${host}`, (err, stdout) => {
    res.send(stdout)
  })
})
// Attack: /ping?host=google.com;rm -rf /
```

**Fix**:
```typescript
// ✅ Validate and sanitize input
app.get('/ping', (req, res) => {
  const host = req.query.host

  // Validate hostname format
  if (!/^[a-z0-9.-]+$/i.test(host)) {
    return res.status(400).json({ error: 'Invalid hostname' })
  }

  // Use array form (no shell interpretation)
  execFile('ping', ['-c', '4', host], (err, stdout) => {
    if (err) return res.status(500).json({ error: 'Ping failed' })
    res.send(stdout)
  })
})

// ✅ Better: Don't execute commands based on user input
```

**Checklist**:
- [ ] Parameterized queries (never string concatenation)
- [ ] ORM/query builder instead of raw SQL
- [ ] Input validation (type, format, whitelist)
- [ ] Avoid exec/eval with user input
- [ ] Sanitize special characters
- [ ] Use prepared statements

### 4. Insecure Design

**What**: Missing or ineffective security controls by design

**Examples**:
- No rate limiting → brute force attacks
- No account lockout → credential stuffing
- Unlimited file uploads → DoS
- No CSRF protection → cross-site request forgery

**Rate limiting**:
```typescript
// ❌ No rate limiting
app.post('/login', loginHandler)

// ✅ Rate limit login attempts
import rateLimit from 'express-rate-limit'

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts per window
  message: 'Too many login attempts, please try again later'
})

app.post('/login', loginLimiter, loginHandler)
```

**CSRF protection**:
```typescript
// ❌ No CSRF protection
app.post('/transfer', requireAuth, transferMoney)

// ✅ CSRF token
import csrf from 'csurf'

const csrfProtection = csrf({ cookie: true })

app.get('/transfer-form', csrfProtection, (req, res) => {
  res.render('transfer', { csrfToken: req.csrfToken() })
})

app.post('/transfer', csrfProtection, requireAuth, transferMoney)
```

**File upload limits**:
```typescript
// ❌ Unlimited uploads
app.post('/upload', upload.single('file'), uploadHandler)

// ✅ Size and type limits
import multer from 'multer'

const upload = multer({
  limits: {
    fileSize: 5 * 1024 * 1024 // 5MB max
  },
  fileFilter: (req, file, cb) => {
    const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf']
    if (!allowedTypes.includes(file.mimetype)) {
      return cb(new Error('Invalid file type'))
    }
    cb(null, true)
  }
})

app.post('/upload', upload.single('file'), uploadHandler)
```

**Checklist**:
- [ ] Rate limiting on sensitive endpoints
- [ ] Account lockout after failed attempts
- [ ] CSRF protection for state-changing operations
- [ ] File upload restrictions (size, type)
- [ ] Session timeout
- [ ] Audit logging for sensitive operations

### 5. Security Misconfiguration

**What**: Insecure default configs, missing patches, exposed error details

**Examples**:

**Verbose error messages**:
```typescript
// ❌ Exposes stack traces to users
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.message,
    stack: err.stack // Leaks internal details!
  })
})

// ✅ Generic error message
app.use((err, req, res, next) => {
  console.error(err) // Log internally

  res.status(500).json({
    error: 'Internal server error'
  })
})
```

**Security headers**:
```typescript
// ❌ Missing security headers
app.use(express.json())

// ✅ Add security headers
import helmet from 'helmet'

app.use(helmet()) // Adds multiple security headers

// Manually:
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff')
  res.setHeader('X-Frame-Options', 'DENY')
  res.setHeader('X-XSS-Protection', '1; mode=block')
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains')
  next()
})
```

**CORS configuration**:
```typescript
// ❌ Allow all origins
app.use(cors({ origin: '*' }))

// ✅ Whitelist specific origins
app.use(cors({
  origin: ['https://yourdomain.com', 'https://app.yourdomain.com'],
  credentials: true
}))
```

**Checklist**:
- [ ] Security headers enabled (helmet.js)
- [ ] CORS properly configured
- [ ] Error messages don't leak details
- [ ] Dependencies updated regularly
- [ ] Default passwords changed
- [ ] Unnecessary services disabled
- [ ] Debug mode off in production

### 6. Vulnerable and Outdated Components

**What**: Using libraries with known vulnerabilities

**Detection**:
```bash
# Check for vulnerabilities
npm audit

# Fix automatically (if possible)
npm audit fix

# Check specific package
npm outdated
```

**Example**:
```bash
# Output shows:
lodash  4.17.15  →  4.17.21 (security fix)
axios   0.19.0   →  1.6.0   (security fix)
```

**Prevention**:
```bash
# Use Snyk
npm install -g snyk
snyk test

# Use Dependabot (GitHub)
# Automatically creates PRs for updates

# Lock file
npm ci # Use exact versions from package-lock.json
```

**Checklist**:
- [ ] Run npm audit regularly
- [ ] Update dependencies monthly
- [ ] Review security advisories
- [ ] Use dependabot or similar
- [ ] Remove unused dependencies
- [ ] Pin versions in package.json
- [ ] Test updates before deploying

### 7. Identification and Authentication Failures

**What**: Weak authentication allowing account compromise

**Common issues**:

**Weak passwords**:
```typescript
// ❌ No password requirements
function validatePassword(password) {
  return password.length > 0
}

// ✅ Enforce strong passwords
function validatePassword(password) {
  if (password.length < 12) {
    return { valid: false, error: 'Password must be at least 12 characters' }
  }

  const hasUppercase = /[A-Z]/.test(password)
  const hasLowercase = /[a-z]/.test(password)
  const hasNumber = /[0-9]/.test(password)
  const hasSpecial = /[!@#$%^&*]/.test(password)

  if (!(hasUppercase && hasLowercase && hasNumber && hasSpecial)) {
    return {
      valid: false,
      error: 'Password must contain uppercase, lowercase, number, and special character'
    }
  }

  // Check against common passwords
  const commonPasswords = ['Password123!', '12345678', 'qwerty']
  if (commonPasswords.includes(password)) {
    return { valid: false, error: 'Password too common' }
  }

  return { valid: true }
}
```

**Session management**:
```typescript
// ❌ Session never expires
app.use(session({
  secret: 'keyboard cat',
  resave: false,
  saveUninitialized: false
}))

// ✅ Session timeout
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    maxAge: 30 * 60 * 1000, // 30 minutes
    httpOnly: true, // Prevent XSS access
    secure: true, // HTTPS only
    sameSite: 'strict' // CSRF protection
  }
}))
```

**JWT best practices**:
```typescript
// ❌ Insecure JWT
const token = jwt.sign({ userId: user.id }, 'weak-secret')

// ✅ Secure JWT
const token = jwt.sign(
  { userId: user.id },
  process.env.JWT_SECRET, // Strong secret
  {
    expiresIn: '15m', // Short-lived
    issuer: 'your-app',
    audience: 'your-app'
  }
)

// Use refresh tokens for long-lived sessions
const refreshToken = crypto.randomBytes(64).toString('hex')
await db.refreshTokens.create({
  userId: user.id,
  token: refreshToken,
  expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 days
})
```

**Multi-factor authentication**:
```typescript
// ✅ Add 2FA
import speakeasy from 'speakeasy'

// Generate secret
const secret = speakeasy.generateSecret({ name: 'YourApp' })

// Verify TOTP code
const verified = speakeasy.totp.verify({
  secret: user.twoFactorSecret,
  encoding: 'base32',
  token: req.body.code,
  window: 1 // Allow 1 step before/after
})
```

**Checklist**:
- [ ] Strong password requirements
- [ ] Password hashing (bcrypt/argon2)
- [ ] Account lockout after failed attempts
- [ ] Session timeout
- [ ] Secure session cookies (httpOnly, secure, sameSite)
- [ ] JWT with expiration
- [ ] MFA for sensitive accounts
- [ ] Password reset with time-limited tokens

### 8. Software and Data Integrity Failures

**What**: Code/infrastructure without integrity verification

**Examples**:

**Insecure deserialization**:
```typescript
// ❌ Unsafe deserialization
app.post('/api/data', (req, res) => {
  const data = eval(req.body.payload) // Never do this!
})

// ✅ Use JSON
app.post('/api/data', (req, res) => {
  const data = JSON.parse(req.body.payload)
  // Validate data structure
})
```

**Unverified CDN resources**:
```html
<!-- ❌ No integrity check -->
<script src="https://cdn.example.com/library.js"></script>

<!-- ✅ Subresource integrity -->
<script
  src="https://cdn.example.com/library.js"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/ux..."
  crossorigin="anonymous"
></script>
```

**Checklist**:
- [ ] Verify package signatures
- [ ] Use SRI for CDN resources
- [ ] Code signing for releases
- [ ] Verify integrity of downloads
- [ ] No eval() or unsafe deserialization

### 9. Security Logging and Monitoring Failures

**What**: Insufficient logging prevents detection of breaches

**Good security logging**:
```typescript
// Log security events
function logSecurityEvent(event, details) {
  logger.warn('SECURITY_EVENT', {
    event,
    ...details,
    timestamp: new Date(),
    ip: details.ip,
    userId: details.userId
  })
}

// Login attempts
app.post('/login', async (req, res) => {
  const { email, password } = req.body
  const user = await db.users.findByEmail(email)

  if (!user || !(await bcrypt.compare(password, user.password))) {
    logSecurityEvent('LOGIN_FAILED', {
      email,
      ip: req.ip,
      userAgent: req.headers['user-agent']
    })
    return res.status(401).json({ error: 'Invalid credentials' })
  }

  logSecurityEvent('LOGIN_SUCCESS', {
    userId: user.id,
    ip: req.ip
  })

  res.json({ token: generateToken(user) })
})

// Access denied
if (order.userId !== req.user.id) {
  logSecurityEvent('ACCESS_DENIED', {
    userId: req.user.id,
    resource: 'order',
    resourceId: req.params.id,
    ip: req.ip
  })
  return res.status(403).json({ error: 'Forbidden' })
}
```

**What to log**:
- Authentication events (success/failure)
- Authorization failures
- Input validation failures
- Admin actions
- Privilege escalations
- Account changes (email, password)
- Payment transactions
- Data exports
- System errors

**What NOT to log**:
- Passwords (even hashed)
- Session tokens
- API keys
- Credit card numbers
- Personal data (GDPR/privacy)

**Checklist**:
- [ ] Log all authentication attempts
- [ ] Log authorization failures
- [ ] Log admin actions
- [ ] Centralized logging
- [ ] Alerts for suspicious activity
- [ ] Log retention policy
- [ ] Don't log sensitive data

### 10. Server-Side Request Forgery (SSRF)

**What**: Attacker tricks server into making requests

**Example vulnerability**:
```typescript
// ❌ SSRF vulnerability
app.get('/fetch', async (req, res) => {
  const url = req.query.url
  const response = await fetch(url)
  res.send(await response.text())
})
// Attack: /fetch?url=http://localhost:6379/
// Can access internal services!
```

**Fix**:
```typescript
// ✅ Whitelist allowed domains
app.get('/fetch', async (req, res) => {
  const url = req.query.url

  // Parse URL
  let parsedUrl
  try {
    parsedUrl = new URL(url)
  } catch (e) {
    return res.status(400).json({ error: 'Invalid URL' })
  }

  // Whitelist allowed hosts
  const allowedHosts = ['api.example.com', 'cdn.example.com']
  if (!allowedHosts.includes(parsedUrl.hostname)) {
    return res.status(403).json({ error: 'Host not allowed' })
  }

  // Block private IPs
  const ip = await dns.promises.resolve4(parsedUrl.hostname)
  if (isPrivateIP(ip[0])) {
    return res.status(403).json({ error: 'Private IP not allowed' })
  }

  const response = await fetch(url)
  res.send(await response.text())
})

function isPrivateIP(ip) {
  return (
    ip.startsWith('10.') ||
    ip.startsWith('192.168.') ||
    ip.startsWith('172.16.') ||
    ip.startsWith('127.')
  )
}
```

**Checklist**:
- [ ] Whitelist allowed domains
- [ ] Block private IP ranges
- [ ] Validate URL format
- [ ] Don't allow redirects
- [ ] Use separate network for external requests

## Additional Security Concerns

### Cross-Site Scripting (XSS)

**Stored XSS**:
```typescript
// ❌ Renders user input as HTML
app.get('/profile/:id', async (req, res) => {
  const user = await db.users.findById(req.params.id)
  res.send(`<h1>${user.bio}</h1>`) // bio contains: <script>steal()</script>
})
```

**Fix**:
```typescript
// ✅ Escape HTML
import escape from 'escape-html'

app.get('/profile/:id', async (req, res) => {
  const user = await db.users.findById(req.params.id)
  res.send(`<h1>${escape(user.bio)}</h1>`)
})

// ✅ Or use templating engine that auto-escapes
res.render('profile', { bio: user.bio }) // EJS/Handlebars escape by default
```

**React XSS**:
```jsx
// ❌ Dangerous
function Profile({ user }) {
  return <div dangerouslySetInnerHTML={{ __html: user.bio }} />
}

// ✅ Safe (React escapes by default)
function Profile({ user }) {
  return <div>{user.bio}</div>
}

// ✅ If you must render HTML, sanitize first
import DOMPurify from 'dompurify'

function Profile({ user }) {
  const sanitized = DOMPurify.sanitize(user.bio)
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />
}
```

### Sensitive Data Exposure

**Don't expose sensitive data**:
```typescript
// ❌ Leaking sensitive fields
app.get('/api/user', requireAuth, async (req, res) => {
  const user = await db.users.findById(req.user.id)
  res.json(user) // Includes password hash, internal IDs, etc.
})

// ✅ Only return necessary fields
app.get('/api/user', requireAuth, async (req, res) => {
  const user = await db.users.findById(req.user.id)
  res.json({
    id: user.id,
    email: user.email,
    name: user.name
    // password, internalId, etc. NOT included
  })
})
```

**Redact logs**:
```typescript
// ❌ Logs sensitive data
logger.info('Request body:', req.body) // Includes password!

// ✅ Redact sensitive fields
function redact(obj) {
  const sensitive = ['password', 'token', 'apiKey', 'ssn', 'creditCard']
  const redacted = { ...obj }

  for (const key of sensitive) {
    if (redacted[key]) {
      redacted[key] = '[REDACTED]'
    }
  }

  return redacted
}

logger.info('Request body:', redact(req.body))
```

## Security Testing

### Automated Scanning

**Dependency scanning**:
```bash
npm audit
snyk test
```

**SAST (Static Analysis)**:
```bash
# ESLint security plugin
npm install --save-dev eslint-plugin-security
```

**DAST (Dynamic Analysis)**:
- OWASP ZAP
- Burp Suite
- Nikto

### Manual Testing

**Authentication bypass**:
- Try accessing resources without token
- Try expired/invalid tokens
- Try other users' tokens
- Try tampering with JWT payload

**Authorization bypass**:
- Access other users' resources (IDOR)
- Try admin endpoints as regular user
- Modify IDs in requests

**Input validation**:
- SQL injection payloads
- XSS payloads
- Command injection
- Path traversal (../../etc/passwd)

**Rate limiting**:
- Rapid-fire requests
- Check if limits enforced

## Security Checklist

### Authentication & Authorization
- [ ] Passwords hashed with bcrypt/argon2
- [ ] Strong password requirements enforced
- [ ] Rate limiting on login endpoint
- [ ] Account lockout after failed attempts
- [ ] Session timeout implemented
- [ ] Secure session cookies (httpOnly, secure, sameSite)
- [ ] Authorization checks on every endpoint
- [ ] Users can't access other users' data (IDOR)
- [ ] Admin functions require admin role

### Input Validation
- [ ] All input validated (type, format, range)
- [ ] Parameterized queries (no SQL injection)
- [ ] No command execution with user input
- [ ] XSS protection (escape output)
- [ ] File upload restrictions (size, type)
- [ ] CSRF protection for state-changing operations

### Data Protection
- [ ] Sensitive data encrypted at rest
- [ ] HTTPS enforced (no HTTP)
- [ ] Security headers set (helmet.js)
- [ ] Secrets in environment variables, not code
- [ ] No sensitive data in logs
- [ ] PII handled according to regulations

### Dependencies & Configuration
- [ ] Dependencies up to date (npm audit)
- [ ] No vulnerabilities in dependencies
- [ ] CORS properly configured
- [ ] Error messages don't leak details
- [ ] Debug mode off in production
- [ ] Unnecessary endpoints/services disabled

### Monitoring & Logging
- [ ] Security events logged
- [ ] Alerts for suspicious activity
- [ ] Centralized logging
- [ ] Log retention policy
- [ ] Sensitive data not logged

---

**Remember**: Security is not a checklist – it's a mindset. Stay paranoid, think like an attacker, and never trust user input. Defense in depth: multiple layers of security so if one fails, others still protect you.
