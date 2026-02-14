# Steps Taken to Solve the Assignment

This document outlines the systematic steps taken to debug and fix the authentication flow in the "Silent Server" assignment.

## 1. Initial Analysis and Setup
- **Environment Review**: Read the `README.md` to understand the 4 required tasks and the expected outcome (obtaining a `success_flag`).
- **Codebase Exploration**: Examined `server.js`, `middleware/`, and `utils/` to map out the authentication logic.
- **Dependency Installation**: Ran `npm install` to set up the required packages (`express`, `cookie-parser`, `jsonwebtoken`).

## 2. Identifying and Fixing Bugs

### Step 2.1: Fixing the Logger Middleware
- **Issue**: All incoming requests were hanging indefinitely.
- **Root Cause**: The `middleware/logger.js` was missing a `next()` call, preventing the request from reaching the route handlers.
- **Fix**: Added `next()` at the end of the `requestLogger` function.

### Step 2.2: Fixing the Login Endpoint (Task 1)
- **Issue**: The OTP was generated but not visible to the user.
- **Fix**: Added `console.log` in `server.js` to output the session ID and the 6-digit OTP.

### Step 2.3: Fixing OTP Verification (Task 2)
- **Issue**: Cookies were not being parsed, so the session could not be tracked efficiently in subsequent steps.
- **Fix**: Registered the `cookie-parser` middleware in `server.js` using `app.use(cookieParser())`.

### Step 2.4: Fixing Token Generation (Task 3)
- **Issue**: The `/auth/token` endpoint failed because it tried to read the session from the `Authorization` header instead of the `session_token` cookie. It also contained a `ReferenceError` where it used an undefined variable `token`.
- **Fix**: 
    - Updated logic to read the session ID from `req.cookies.session_token`.
    - Corrected the variable name to `sessionToken` in the JWT signing payload.

### Step 2.5: Fixing Protected Route Access (Task 4)
- **Issue**: Validating the token was working, but the request never reached the final route handler.
- **Root Cause**: The `authMiddleware` in `middleware/auth.js` was missing a `next()` call after successful verification.
- **Fix**: Added `next()` after `req.user = decoded`.

## 3. Verification and Testing
- **Server Execution**: Started the server using `npm run dev` (utilizing node's `--watch` mode).
- **Manual Testing**: Performed a full execution of the authentication flow using PowerShell's `Invoke-RestMethod` and `curl.exe`:
    1. `POST /auth/login` -> Received `loginSessionId`.
    2. `POST /auth/verify-otp` -> Verified OTP and captured `session_token` cookie.
    3. `POST /auth/token` -> Exchanged cookie for JWT `access_token`.
    4. `GET /protected` -> Accessed the route with the Bearer token and received the `success_flag`.

## 4. Finalizing Submission
- **Output Generation**: Created `output.txt` containing the terminal output for all 4 test commands as requested.
- **Documentation**: Verified the `success_flag` matches the expected format based on the email used.
