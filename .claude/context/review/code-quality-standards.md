---
title: Enterprise Code Quality Standards
description: Concrete examples of good vs bad code patterns for enterprise-grade software
tags: [code-quality, best-practices, enterprise, security, testing]
---

# Enterprise Code Quality Standards

Technical reference for `.claude/agents/code-quality-reviewer.md` with concrete examples of enterprise-grade patterns. Validated with `/validate-code-quality-review`.

## Category 1: Code Quality & Standards

### Type Safety

#### ❌ Bad: Using 'any' types
```typescript
// TypeScript
function processData(data: any): any {
    return data.map((item: any) => item.value);
}
```

#### ✅ Good: Proper type definitions
```typescript
// TypeScript
interface DataItem {
    id: string;
    value: number;
}

function processData(data: DataItem[]): number[] {
    return data.map(item => item.value);
}
```

#### ❌ Bad: No type hints
```python
# Python
def calculate_total(items):
    return sum(item['price'] * item['quantity'] for item in items)
```

#### ✅ Good: Explicit type hints
```python
# Python
from typing import TypedDict

class OrderItem(TypedDict):
    price: float
    quantity: int

def calculate_total(items: list[OrderItem]) -> float:
    return sum(item['price'] * item['quantity'] for item in items)
```

### DRY Principle

#### ❌ Bad: Code duplication
```python
# Python
def get_active_users():
    users = db.query("SELECT * FROM users WHERE active = 1")
    return [{"id": u.id, "name": u.name, "email": u.email} for u in users]

def get_premium_users():
    users = db.query("SELECT * FROM users WHERE premium = 1")
    return [{"id": u.id, "name": u.name, "email": u.email} for u in users]
```

#### ✅ Good: Extracted common logic
```python
# Python
def _format_user(user):
    return {"id": user.id, "name": user.name, "email": user.email}

def get_users_by_condition(condition: str):
    users = db.query(f"SELECT * FROM users WHERE {condition}")
    return [_format_user(u) for u in users]

def get_active_users():
    return get_users_by_condition("active = 1")

def get_premium_users():
    return get_users_by_condition("premium = 1")
```

### Single Responsibility

#### ❌ Bad: Function doing too much
```typescript
// TypeScript
function handleUserRegistration(email: string, password: string) {
    // Validate email
    if (!email.includes('@')) throw new Error('Invalid email');

    // Hash password
    const hashedPassword = bcrypt.hashSync(password, 10);

    // Save to database
    db.users.insert({ email, password: hashedPassword });

    // Send welcome email
    sendEmail(email, 'Welcome!', 'Thanks for signing up');

    // Log analytics
    analytics.track('user_registered', { email });
}
```

#### ✅ Good: Separated concerns
```typescript
// TypeScript
function validateEmail(email: string): void {
    if (!email.includes('@')) {
        throw new ValidationError('Invalid email format');
    }
}

function hashPassword(password: string): string {
    return bcrypt.hashSync(password, 10);
}

function createUser(email: string, hashedPassword: string): User {
    return db.users.insert({ email, password: hashedPassword });
}

function handleUserRegistration(email: string, password: string): User {
    validateEmail(email);
    const hashedPassword = hashPassword(password);
    const user = createUser(email, hashedPassword);

    // Side effects handled separately
    sendWelcomeEmail(user.email);
    trackUserRegistration(user.email);

    return user;
}
```

## Category 2: Security

### No Hardcoded Secrets

#### ❌ Bad: Hardcoded credentials
```python
# Python
API_KEY = "sk-1234567890abcdef"
DATABASE_URL = "postgresql://admin:password123@localhost/db"

def fetch_data():
    headers = {"Authorization": f"Bearer {API_KEY}"}
    return requests.get("https://api.example.com/data", headers=headers)
```

#### ✅ Good: Environment variables
```python
# Python
import os
from dotenv import load_dotenv

load_dotenv()

API_KEY = os.getenv("API_KEY")
DATABASE_URL = os.getenv("DATABASE_URL")

if not API_KEY or not DATABASE_URL:
    raise ValueError("Missing required environment variables")

def fetch_data():
    headers = {"Authorization": f"Bearer {API_KEY}"}
    return requests.get("https://api.example.com/data", headers=headers)
```

### Input Validation

#### ❌ Bad: SQL injection vulnerability
```python
# Python
def get_user_by_email(email: str):
    query = f"SELECT * FROM users WHERE email = '{email}'"
    return db.execute(query).fetchone()
```

#### ✅ Good: Parameterized queries
```python
# Python
def get_user_by_email(email: str):
    query = "SELECT * FROM users WHERE email = ?"
    return db.execute(query, (email,)).fetchone()
```

#### ❌ Bad: No input validation
```typescript
// TypeScript
app.post('/api/users', (req, res) => {
    const user = req.body;
    db.users.insert(user);
    res.json({ success: true });
});
```

#### ✅ Good: Input validation with schema
```typescript
// TypeScript
import { z } from 'zod';

const UserSchema = z.object({
    email: z.string().email(),
    age: z.number().min(13).max(120),
    name: z.string().min(1).max(100)
});

app.post('/api/users', (req, res) => {
    try {
        const validatedUser = UserSchema.parse(req.body);
        db.users.insert(validatedUser);
        res.json({ success: true });
    } catch (error) {
        res.status(400).json({ error: 'Invalid input' });
    }
});
```

### Authentication & Authorization

#### ❌ Bad: No authorization check
```python
# Python
@app.route('/api/admin/users', methods=['DELETE'])
def delete_user(user_id: str):
    db.users.delete(user_id)
    return {"success": True}
```

#### ✅ Good: Proper authorization
```python
# Python
from functools import wraps

def require_admin(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if not current_user.is_authenticated:
            return {"error": "Authentication required"}, 401
        if not current_user.is_admin:
            return {"error": "Admin access required"}, 403
        return f(*args, **kwargs)
    return decorated_function

@app.route('/api/admin/users', methods=['DELETE'])
@require_admin
def delete_user(user_id: str):
    db.users.delete(user_id)
    return {"success": True}
```

## Category 3: Error Handling & Reliability

### Proper Exception Handling

#### ❌ Bad: Bare except
```python
# Python
def fetch_user_data(user_id: str):
    try:
        return api.get_user(user_id)
    except:  # Catches everything including KeyboardInterrupt!
        return None
```

#### ✅ Good: Specific exception handling
```python
# Python
from requests.exceptions import RequestException, Timeout

def fetch_user_data(user_id: str) -> dict | None:
    try:
        return api.get_user(user_id)
    except Timeout:
        logger.warning(f"Timeout fetching user {user_id}")
        return None
    except RequestException as e:
        logger.error(f"Error fetching user {user_id}: {e}")
        raise
```

### Meaningful Error Messages

#### ❌ Bad: Vague error
```typescript
// TypeScript
function processPayment(amount: number, cardToken: string) {
    if (amount <= 0) {
        throw new Error('Invalid amount');
    }
    // ...
}
```

#### ✅ Good: Actionable error
```typescript
// TypeScript
class PaymentError extends Error {
    constructor(message: string, public code: string) {
        super(message);
        this.name = 'PaymentError';
    }
}

function processPayment(amount: number, cardToken: string) {
    if (amount <= 0) {
        throw new PaymentError(
            `Payment amount must be positive. Received: ${amount}`,
            'INVALID_AMOUNT'
        );
    }
    if (!cardToken) {
        throw new PaymentError(
            'Card token is required for payment processing',
            'MISSING_CARD_TOKEN'
        );
    }
    // ...
}
```

### Resource Cleanup

#### ❌ Bad: No cleanup
```python
# Python
def process_file(filepath: str):
    file = open(filepath, 'r')
    data = file.read()
    return process_data(data)
    # File never closed if process_data raises exception
```

#### ✅ Good: Context manager
```python
# Python
def process_file(filepath: str):
    with open(filepath, 'r') as file:
        data = file.read()
        return process_data(data)
    # File automatically closed even if exception occurs
```

## Category 4: Testing

### Meaningful Tests

#### ❌ Bad: Testing implementation details
```typescript
// TypeScript
test('user service has correct method names', () => {
    const service = new UserService();
    expect(service.findUser).toBeDefined();
    expect(service.createUser).toBeDefined();
});
```

#### ✅ Good: Testing behavior
```typescript
// TypeScript
test('creates user with valid email and returns user object', async () => {
    const service = new UserService();
    const user = await service.createUser({
        email: 'test@example.com',
        name: 'Test User'
    });

    expect(user).toMatchObject({
        email: 'test@example.com',
        name: 'Test User'
    });
    expect(user.id).toBeDefined();
});

test('throws ValidationError for invalid email', async () => {
    const service = new UserService();

    await expect(
        service.createUser({ email: 'invalid-email', name: 'Test' })
    ).rejects.toThrow(ValidationError);
});
```

### Descriptive Test Names

#### ❌ Bad: Vague test names
```python
# Python
def test_user():
    assert True

def test_user_2():
    user = create_user("test@example.com")
    assert user is not None
```

#### ✅ Good: Descriptive test names
```python
# Python
def test_create_user_with_valid_email_returns_user_object():
    user = create_user("test@example.com")
    assert user.email == "test@example.com"
    assert user.id is not None

def test_create_user_with_duplicate_email_raises_duplicate_error():
    create_user("test@example.com")
    with pytest.raises(DuplicateEmailError):
        create_user("test@example.com")

def test_create_user_with_invalid_email_raises_validation_error():
    with pytest.raises(ValidationError):
        create_user("not-an-email")
```

### Test Coverage

#### ❌ Bad: Only happy path
```python
# Python
def test_calculate_discount():
    assert calculate_discount(100, "SAVE10") == 90
```

#### ✅ Good: Happy path + edge cases
```python
# Python
def test_calculate_discount_with_valid_code():
    assert calculate_discount(100, "SAVE10") == 90

def test_calculate_discount_with_invalid_code():
    assert calculate_discount(100, "INVALID") == 100

def test_calculate_discount_with_expired_code():
    with pytest.raises(ExpiredCouponError):
        calculate_discount(100, "EXPIRED")

def test_calculate_discount_with_zero_amount():
    assert calculate_discount(0, "SAVE10") == 0

def test_calculate_discount_with_negative_amount():
    with pytest.raises(ValueError):
        calculate_discount(-10, "SAVE10")
```

## Category 5: Maintainability

### Magic Numbers

#### ❌ Bad: Magic numbers
```typescript
// TypeScript
function isPasswordValid(password: string): boolean {
    return password.length >= 8 && password.length <= 128;
}

function calculateSessionExpiry(): Date {
    return new Date(Date.now() + 3600000);
}
```

#### ✅ Good: Named constants
```typescript
// TypeScript
const PASSWORD_MIN_LENGTH = 8;
const PASSWORD_MAX_LENGTH = 128;
const SESSION_DURATION_MS = 60 * 60 * 1000; // 1 hour

function isPasswordValid(password: string): boolean {
    return password.length >= PASSWORD_MIN_LENGTH
        && password.length <= PASSWORD_MAX_LENGTH;
}

function calculateSessionExpiry(): Date {
    return new Date(Date.now() + SESSION_DURATION_MS);
}
```

### Code Documentation

#### ❌ Bad: No documentation for complex logic
```python
# Python
def calculate_shipping(weight, zone, express):
    base = 5.99
    if weight > 10:
        base += (weight - 10) * 0.5
    if zone == 2:
        base *= 1.5
    elif zone == 3:
        base *= 2
    if express:
        base *= 1.5
    return round(base, 2)
```

#### ✅ Good: Documented complex logic
```python
# Python
def calculate_shipping(weight_lbs: float, zone: int, express: bool) -> float:
    """
    Calculate shipping cost based on weight, zone, and service level.

    Base rate is $5.99 for packages up to 10 lbs.
    Additional weight charged at $0.50/lb.

    Zone multipliers:
    - Zone 1 (local): 1.0x
    - Zone 2 (regional): 1.5x
    - Zone 3 (national): 2.0x

    Express service adds 50% surcharge.

    Args:
        weight_lbs: Package weight in pounds
        zone: Shipping zone (1-3)
        express: Whether express shipping is selected

    Returns:
        Shipping cost in dollars, rounded to 2 decimal places
    """
    BASE_RATE = 5.99
    WEIGHT_THRESHOLD_LBS = 10
    ADDITIONAL_WEIGHT_RATE = 0.50
    ZONE_MULTIPLIERS = {1: 1.0, 2: 1.5, 3: 2.0}
    EXPRESS_MULTIPLIER = 1.5

    cost = BASE_RATE

    if weight_lbs > WEIGHT_THRESHOLD_LBS:
        additional_weight = weight_lbs - WEIGHT_THRESHOLD_LBS
        cost += additional_weight * ADDITIONAL_WEIGHT_RATE

    cost *= ZONE_MULTIPLIERS.get(zone, 1.0)

    if express:
        cost *= EXPRESS_MULTIPLIER

    return round(cost, 2)
```

### Dead Code Removal

#### ❌ Bad: Commented-out code
```typescript
// TypeScript
function processOrder(order: Order) {
    // const oldPrice = calculateOldPrice(order);
    // console.log('Old pricing:', oldPrice);

    const price = calculatePrice(order);

    // applyLegacyDiscount(order);
    // validateOldRules(order);

    return {
        orderId: order.id,
        total: price
    };
}
```

#### ✅ Good: Clean code
```typescript
// TypeScript
function processOrder(order: Order) {
    const price = calculatePrice(order);

    return {
        orderId: order.id,
        total: price
    };
}
```

## Category 6: Performance & Efficiency

### Database Optimization

#### ❌ Bad: N+1 query problem
```python
# Python
def get_users_with_posts():
    users = User.query.all()
    result = []
    for user in users:
        posts = Post.query.filter_by(user_id=user.id).all()  # N queries
        result.append({"user": user, "posts": posts})
    return result
```

#### ✅ Good: Eager loading
```python
# Python
def get_users_with_posts():
    users = User.query.options(joinedload(User.posts)).all()
    return [{"user": user, "posts": user.posts} for user in users]
```

### Appropriate Data Structures

#### ❌ Bad: List for membership testing
```python
# Python
def filter_active_users(user_ids: list[str], all_users: list[User]) -> list[User]:
    active_ids = [u.id for u in all_users if u.active]  # List
    return [u for u in all_users if u.id in active_ids]  # O(n²)
```

#### ✅ Good: Set for membership testing
```python
# Python
def filter_active_users(user_ids: list[str], all_users: list[User]) -> list[User]:
    active_ids = {u.id for u in all_users if u.active}  # Set
    return [u for u in all_users if u.id in active_ids]  # O(n)
```

### Loop Efficiency

#### ❌ Bad: Repeated computation in loop
```typescript
// TypeScript
function processItems(items: Item[]) {
    const results = [];
    for (const item of items) {
        const config = getConfig();  // Called every iteration!
        const multiplier = config.baseValue * config.factor;
        results.push(item.value * multiplier);
    }
    return results;
}
```

#### ✅ Good: Computation outside loop
```typescript
// TypeScript
function processItems(items: Item[]) {
    const config = getConfig();  // Called once
    const multiplier = config.baseValue * config.factor;

    return items.map(item => item.value * multiplier);
}
```

## Language-Specific Patterns

### Python Best Practices

#### Use type hints everywhere
```python
from typing import Optional, Union

def get_user(user_id: str) -> Optional[User]:
    return db.users.find_one({"id": user_id})
```

#### Use context managers for resources
```python
with database.transaction():
    user = create_user(email)
    send_welcome_email(user)
```

#### Use dataclasses for data structures
```python
from dataclasses import dataclass

@dataclass
class User:
    id: str
    email: str
    created_at: datetime
```

### TypeScript Best Practices

#### Avoid non-null assertions
```typescript
// ❌ Bad
const user = users.find(u => u.id === id)!;

// ✅ Good
const user = users.find(u => u.id === id);
if (!user) {
    throw new Error(`User not found: ${id}`);
}
```

#### Use readonly for immutable data
```typescript
interface User {
    readonly id: string;
    readonly email: string;
    name: string;  // Mutable
}
```

#### Prefer unknown over any
```typescript
// ❌ Bad
function parseJson(json: string): any {
    return JSON.parse(json);
}

// ✅ Good
function parseJson(json: string): unknown {
    return JSON.parse(json);
}
```

## Security Patterns (OWASP Top 10)

### 1. Injection Prevention
- Use parameterized queries
- Validate and sanitize inputs
- Use ORM/query builders

### 2. Broken Authentication
- Use established auth libraries (Passport, Auth0)
- Implement MFA where appropriate
- Secure session management

### 3. Sensitive Data Exposure
- Encrypt sensitive data at rest
- Use HTTPS for all communications
- Don't log sensitive information

### 4. XML External Entities (XXE)
- Disable XML external entity processing
- Use JSON instead of XML when possible
- Validate and sanitize XML inputs

### 5. Broken Access Control
- Implement authorization checks on every endpoint
- Use principle of least privilege
- Validate user permissions server-side

## Integration Points

### Agent Guidance
These examples support the workflow defined in `.claude/agents/code-quality-reviewer.md`.

### Validation Command
Examples demonstrate what passes/fails validation in `/validate-code-quality-review`.

### Related Context
- Security patterns align with OWASP standards
- Testing patterns follow industry best practices
- Performance patterns based on algorithmic complexity

## Context7 Validation

These standards are validated against latest documentation:
- **Python**: Ruff, mypy, pytest best practices
- **TypeScript**: ESLint, TypeScript compiler, Jest best practices
- **Security**: OWASP Top 10 (2021)
- **Testing**: Testing best practices from pytest, Jest documentation

## Usage in Reviews

When reviewing code:
1. Compare code against these examples
2. Reference specific good/bad examples in feedback
3. Provide file:line references for issues
4. Suggest specific improvements based on patterns shown

## Continuous Improvement

These standards should evolve with:
- New security vulnerabilities (OWASP updates)
- New language features (Python 3.13+, TypeScript 5.x)
- New testing patterns
- Team-specific conventions (see project CLAUDE.md)
