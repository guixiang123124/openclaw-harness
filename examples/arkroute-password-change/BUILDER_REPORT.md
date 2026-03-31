# Builder Report: Password Change Endpoint

## Changes Made

### db.py
Added `update_user_password(user_id, new_password)` function (lines 115-120) that:
- Hashes the new password with bcrypt (truncated to 72 bytes per existing pattern)
- Updates `password_hash` in the users table
- Follows the existing `get_conn() / commit / close` pattern

### main.py
Added `POST /api/change-password` endpoint (lines 278-290) that:
- Authenticates via `auth_module.get_current_user(authorization)` (same pattern as `/api/me`, `/api/keys`)
- Accepts `{"current_password", "new_password"}` JSON body
- Returns 400 if `new_password` is fewer than 8 characters
- Verifies `current_password` against stored hash using `db.bcrypt.verify`; returns 401 if wrong
- Calls `db.update_user_password` on success
- Returns 200 `{"message": "Password updated"}`

## Success Criteria Checklist

- [x] POST `/api/change-password` accepts `{current_password, new_password}` with Bearer auth
- [x] Validates current password before allowing change
- [x] Returns 401 if current password is wrong
- [x] Returns 400 if new_password is <8 characters
- [x] Returns 200 `{"message": "Password updated"}` on success
- [x] Updates password_hash in DB using bcrypt
- [x] py_compile passes on all changed files
- [x] No TODO / FIXME / console.log in code

## Files Modified
- `main.py` (+13 lines)
- `db.py` (+7 lines)

## Files NOT Touched
- `auth.py`, `config.py`, `models.py`, `providers/` - unchanged
- Database schema - no ALTER TABLE
