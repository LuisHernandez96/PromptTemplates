# Coding Best Practices

## Core Principles

- **Minimal implementation** - Write just enough code to pass the tests, nothing more
- **YAGNI** - You Aren't Gonna Need It; no speculative features or "just in case" code
- **Single responsibility** - Each function/class does one thing well
- **Prefer clarity over cleverness** - Readable code beats clever code

## Anti-Patterns to Avoid

### 1. Over-Engineering

```python
# BAD - Premature abstraction for a single use case
class AbstractDataProcessor:
    def process(self, data): raise NotImplementedError

class JSONProcessor(AbstractDataProcessor):
    def process(self, data): return json.loads(data)

processor_factory = ProcessorFactory()
processor_factory.register("json", JSONProcessor)
result = processor_factory.create("json").process(data)

# GOOD - Direct implementation until abstraction is actually needed
result = json.loads(data)
```

```python
# BAD - Unnecessary configurability
def fetch_user(id, cache=None, timeout=30, retry_count=3,
               backoff_factor=2, validate=True, transform=None):
    ...

# GOOD - Sensible defaults, extend only when needed
def fetch_user(id):
    ...
```

### 2. Ignoring Test Boundaries

```python
# BAD - Test requires "add user", implementation does more
def add_user(name):
    user = User(name=name)
    db.save(user)
    send_welcome_email(user)      # Not required by test!
    log_analytics("user_created") # Not required by test!
    return user

# GOOD - Only what the test specifies
def add_user(name):
    user = User(name=name)
    db.save(user)
    return user
```

### 3. Security Vulnerabilities

```python
# BAD - SQL injection
query = f"SELECT * FROM users WHERE id = {user_id}"
cursor.execute(query)

# GOOD - Parameterized query
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
```

```python
# BAD - XSS vulnerability
return f"<div>Welcome, {username}</div>"

# GOOD - Proper escaping
from html import escape
return f"<div>Welcome, {escape(username)}</div>"
```

```python
# BAD - Hardcoded secrets
API_KEY = "sk-1234567890abcdef"
db_password = "admin123"

# GOOD - Environment variables or secret management
API_KEY = os.environ["API_KEY"]
db_password = get_secret("db_password")
```

### 4. Code Duplication vs Premature DRY

```python
# BAD - Premature abstraction for 2 similar cases
def process_entity(entity_type, data):
    if entity_type == "user":
        validate_user(data)
        save_user(data)
    elif entity_type == "product":
        validate_product(data)
        save_product(data)

# GOOD - Keep separate until pattern is clear (Rule of Three)
def process_user(data):
    validate_user(data)
    save_user(data)

def process_product(data):
    validate_product(data)
    save_product(data)
```

```python
# BAD - Obvious duplication that should be consolidated
def get_user_email(user_id):
    conn = connect_to_db()
    result = conn.execute("SELECT email FROM users WHERE id = ?", (user_id,))
    conn.close()
    return result

def get_user_name(user_id):
    conn = connect_to_db()
    result = conn.execute("SELECT name FROM users WHERE id = ?", (user_id,))
    conn.close()
    return result

# GOOD - Extract when duplication is clear
def get_user_field(user_id, field):
    conn = connect_to_db()
    result = conn.execute(f"SELECT {field} FROM users WHERE id = ?", (user_id,))
    conn.close()
    return result
```

## Code Quality Guidelines

### Naming Conventions

```python
# BAD - Cryptic names
def proc(d, f):
    return [x for x in d if f(x)]

# GOOD - Descriptive names
def filter_items(items, predicate):
    return [item for item in items if predicate(item)]
```

```python
# BAD - Misleading names
def get_users():  # Actually fetches and deletes expired users
    users = db.fetch_all()
    db.delete_expired()
    return users

# GOOD - Name matches behavior
def fetch_users_and_cleanup_expired():
    users = db.fetch_all()
    db.delete_expired()
    return users
```

### Function Size & Complexity

```python
# BAD - Function doing too much
def process_order(order):
    # Validate (20 lines)
    # Calculate totals (30 lines)
    # Apply discounts (25 lines)
    # Update inventory (20 lines)
    # Send notifications (15 lines)
    ...

# GOOD - Single responsibility per function
def process_order(order):
    validate_order(order)
    totals = calculate_totals(order)
    final_price = apply_discounts(totals, order.customer)
    update_inventory(order.items)
    send_order_confirmation(order, final_price)
```

### Error Handling

```python
# BAD - Swallowing exceptions
try:
    result = risky_operation()
except Exception:
    pass  # Silent failure

# GOOD - Handle specifically or propagate
try:
    result = risky_operation()
except ConnectionError:
    logger.warning("Connection failed, using cached data")
    result = get_cached_data()
except ValidationError as e:
    raise InvalidInputError(f"Invalid input: {e}") from e
```

```python
# BAD - Over-catching
try:
    user = get_user(id)
    send_email(user.email)
except Exception as e:
    return "Something went wrong"

# GOOD - Catch what you can handle
try:
    user = get_user(id)
except UserNotFoundError:
    return "User not found"

try:
    send_email(user.email)
except SMTPError:
    logger.error("Email failed, queuing for retry")
    queue_email_retry(user.email)
```

### Comments & Self-Documenting Code

```python
# BAD - Comment explains what (code should be clear)
# Loop through users and check if active
for u in users:
    if u.a:  # a is active flag
        process(u)

# GOOD - Code is self-documenting
for user in users:
    if user.is_active:
        process(user)
```

```python
# GOOD - Comment explains why (non-obvious reasoning)
# Using 30-second timeout because external API is slow under load
# See incident report INC-1234
timeout = 30
```

## Security Checklist (OWASP Top 10)

- [ ] **Input validation** at system boundaries (user input, external APIs)
- [ ] **Parameterized queries** for all database operations (no string concatenation)
- [ ] **Output encoding** appropriate to context (HTML, URL, JavaScript, SQL)
- [ ] **No hardcoded secrets** in source code (use environment variables or secret managers)
- [ ] **Principle of least privilege** for file access, database permissions, API scopes
- [ ] **Secure defaults** - deny by default, require explicit opt-in for risky operations
- [ ] **Authentication checks** before sensitive operations
- [ ] **Rate limiting** on public endpoints
- [ ] **Logging** of security-relevant events (without sensitive data)
- [ ] **Dependency security** - no known vulnerable packages

## Checklist Before Completion

- [ ] Implementation passes all tests (no more, no less)
- [ ] No unused code or dead branches
- [ ] No TODO comments for current task scope
- [ ] Security checklist verified
- [ ] Code is readable without comments explaining "what"
- [ ] Error handling is appropriate (not swallowing, not over-catching)
- [ ] Names are descriptive and accurate
- [ ] Functions have single responsibility
