# Sprint: Password Reset Endpoint

## Goal
Add a password change endpoint for authenticated users, so users logged into the dashboard can update their password.

## Success Criteria
- [ ] POST `/api/change-password` accepts `{current_password, new_password}` with Bearer auth
- [ ] Validates current password before allowing change
- [ ] Returns 401 if current password is wrong
- [ ] Returns 400 if new_password is <8 characters
- [ ] Returns 200 `{"message": "Password updated"}` on success
- [ ] Updates password_hash in DB using bcrypt
- [ ] py_compile passes on all changed files
- [ ] No TODO / FIXME / console.log in code

## File Scope
**Can modify:**
- `main.py` (add the endpoint, ~15 lines)
- `db.py` (add `update_user_password` function, ~10 lines)

**Must NOT touch:**
- `auth.py` (JWT logic)
- `config.py`
- `providers/` (any provider file)
- `models.py` (no new Pydantic models needed — use inline)
- Database schema (no ALTER TABLE)

## Context
ArkRoute is a FastAPI visual AI routing API. Auth uses JWT Bearer tokens.

### Existing Auth Pattern
```python
# How auth works in this project (from main.py):
@app.get("/api/me")
async def get_me(authorization: str = Header(None)):
    user = auth_module.get_current_user(authorization)  # returns user dict or raises 401
    # ... use user["id"], user["email"], user["credits"]
```

### Existing Error Pattern
```python
# Errors use HTTPException:
raise HTTPException(status_code=401, detail="Invalid email or password")
raise HTTPException(status_code=402, detail="Insufficient credits.")
```

### DB Pattern
```python
# db.py functions follow this pattern:
def some_function(param):
    conn = get_conn()
    # ... do stuff
    conn.commit()  # if writing
    conn.close()
    return result
```

### bcrypt Pattern
```python
# Password hashing (from db.py):
pw_hash = bcrypt.hash(password[:72])  # truncate at 72 bytes (bcrypt limit)
# Verification:
bcrypt.verify(password[:72], row["password_hash"])  # returns bool
```

## DO NOT BREAK
- [ ] POST /auth/register still works
- [ ] POST /auth/login still works  
- [ ] GET /api/me still works
- [ ] All existing endpoints unchanged
- [ ] No circular imports introduced
