# CodeAssess

CodeAssess is a React assessment workspace backed by an Express API, PostgreSQL, Redis/BullMQ, and Judge0. Candidate accounts are self-service; administrators are provisioned from environment variables when the seed command runs. No code is executed by the API process.

## Local setup

1. Install Docker Desktop and start the local data services:

   ```powershell
   docker compose up -d
   ```

2. Copy `.env.example` to `.env`. Replace `JWT_SECRET` with a random value at least 32 characters long. Set `SEED_ADMIN_EMAIL` and `SEED_ADMIN_PASSWORD` to provision an admin account. Leave `JUDGE0_URL` blank until a Judge0 instance is available; code submissions will then report that judging is not configured.

3. Install packages and initialize the database:

   ```powershell
   pnpm install
   pnpm migrate
   pnpm seed
   ```

4. Start the API, evaluation worker, and frontend in three terminals:

   ```powershell
   pnpm api
   pnpm worker
   pnpm dev
   ```

Open the Vite URL shown in the frontend terminal. Candidate users can register from the sign-in screen. Use the configured admin email/password to access admin-only endpoints.

The Vite frontend calls `http://localhost:4000/api` by default. Set `VITE_API_URL` at build time when the API is hosted elsewhere. Set `CLIENT_ORIGIN` on the API to the browser origins that should be allowed.

## Judge0

Set `JUDGE0_URL` to a reachable Judge0 CE API and optionally set `JUDGE0_API_KEY`. The worker submits each test case to Judge0 with the question's CPU and memory limits. `JUDGE0_LANGUAGE_JAVASCRIPT`, `JUDGE0_LANGUAGE_PYTHON`, `JUDGE0_LANGUAGE_JAVA`, and `JUDGE0_LANGUAGE_CPP` override the default Judge0 language IDs. Run mode evaluates visible examples only; submit mode evaluates visible and hidden cases. Hidden inputs and expected outputs are never returned by candidate endpoints.

The demo coding questions use standard input/output formats, so solutions are evaluated in Judge0's sandbox. Do not expose Judge0 keys in the browser.

## API outline

- `POST /api/auth/register`, `/login`, `/refresh`, `/logout`; `GET /api/auth/me`
- `GET /api/candidate/dashboard`, `GET /api/assessments`, `POST /api/assessments/:id/start`
- `GET /api/attempts/:id`, `POST /api/attempts/:id/answers`, `/submit`, `GET /api/attempts/:id/result`
- `POST /api/submissions`, `GET /api/submissions/:id`
- `GET /api/practice/questions`, `POST /api/practice/questions/:id/start`
- Admin-only assessment and question CRUD, test-case management, user listing, and analytics under `/api`
- `GET /api/health` checks PostgreSQL and Redis and reports Judge0 configuration

All list endpoints return JSON; errors use `{ "success": false, "message": "...", "errors": ... }`. Access tokens are short-lived JWTs; refresh tokens are rotated and stored as hashes in PostgreSQL in an HTTP-only cookie. New registrations always receive the candidate role. Never expose hidden cases, answer keys, passwords, or refresh tokens in frontend responses.
