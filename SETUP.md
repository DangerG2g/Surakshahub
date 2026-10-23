# SurakshaHub — Vercel + managed Postgres

This version is designed for **Vercel hosting and Vercel Functions**, with a managed PostgreSQL database connected from the Vercel Marketplace. It does not use Supabase and does not require `VITE_SUPABASE_URL` or `VITE_SUPABASE_ANON_KEY`. You do not need a separate always-on Node.js server.

## Features
- Username/password signup and login without email addresses.
- Password hashes stored using bcrypt; no plaintext passwords.
- HttpOnly session cookies signed with a server-only JWT secret.
- Public signup always creates a standard User account. Choosing Staff/Admin on the form never grants elevated permissions.
- Admin can assign trusted roles, assign maintenance requests to registered staff, and verify completed jobs.
- Staff can update work assigned to them and submit it for admin verification.
- Postgres stores accounts, roles, maintenance requests, alerts, assets and visitors.

## 1. Connect a database in Vercel
1. Push this repository to GitHub and import it into Vercel (Framework preset Vite, build command `npm run build`, output directory `dist`).
2. In Vercel, open **Storage / Marketplace → Browse Storage → Database → Neon Postgres** (the exact menu labels may vary).
3. Create/connect a Neon Postgres database to this Vercel project. Vercel's integration adds database connection variables such as `DATABASE_URL` or `POSTGRES_URL`.
4. In Vercel → Project → Settings → Environment Variables, add:
   - `JWT_SECRET`: a long random secret (at least 32 characters; use a cryptographically random value).
   - `BOOTSTRAP_ADMIN_USERNAME`: a username for your first administrator, 3–32 letters/numbers/dots/underscores/hyphens.
   - `BOOTSTRAP_ADMIN_PASSWORD`: a unique password at least 12 characters long.
5. Apply variables to Production, then redeploy.

**No Supabase environment variables are needed.** Do not put `JWT_SECRET`, database connection strings, or bootstrap admin passwords in `VITE_` variables or frontend code.

## 2. First run
The API creates its database tables automatically when it is first called and adds sample asset rows. If no administrator exists, it creates the bootstrap administrator from the two `BOOTSTRAP_ADMIN_*` variables.

After you successfully sign in to the admin account:
1. Remove `BOOTSTRAP_ADMIN_PASSWORD` from Vercel Environment Variables and redeploy. Do not remove `JWT_SECRET` or the database connection variable.
2. Use **People & roles** to promote verified accounts to Maintenance Staff or Security Staff.
3. Keep administrator role assignment limited to trusted people.

The bootstrap administrator is created only if no admin exists and the chosen username is not already registered. If you deploy without the required variables, the API will explain which server configuration is missing.

## 3. App behavior
- Username format: 3–32 characters, letters/numbers/dots/underscores/hyphens.
- Password length: 8–128 characters for public signup; use a unique password.
- Public signups receive User permissions regardless of selected portal. An admin must assign staff/admin roles.
- Staff can update jobs assigned to them and move them to `Awaiting verification`.
- Only an admin can mark a task `Completed`, and only after it is awaiting verification.
- Email-based password recovery is not included. Before production, add an administrator-assisted password reset process with identity checks.

## 4. Local development
The frontend Vite server proxies `/api` requests to `http://localhost:3001`, but this package uses Vercel Functions for the backend. For the most faithful test, deploy a Preview build to Vercel with a connected Postgres database and the environment variables above. Local API execution can be added using `vercel dev` and the Vercel CLI if desired.

## 5. Security and reliability
- Use HTTPS in production; the session cookie is `HttpOnly`, `Secure`, and `SameSite=Lax`.
- Keep `JWT_SECRET` stable and private; rotating it signs out all active sessions.
- Keep database credentials server-side only.
- The database is a managed service attached to Vercel; Vercel Functions are serverless, not a persistent server.
- This is a project prototype. Review backups, privacy, access control, password recovery, logging and institutional requirements before storing sensitive visitor or building data.
