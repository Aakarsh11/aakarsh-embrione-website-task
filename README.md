# The Embrione — Member Portal

A full-stack recruitment challenge project for The Embrione at PES University. It includes member registration with a live preview, browser-side WebP conversion, admin moderation, and an approved-only team directory.

## Features

- `/register`: two-step responsive form, real-time member-card preview, image conversion to WebP (maximum 800px dimension and quality 0.8), duplicate-SRN check, friendly success/error states.
- `/admin`: challenge credentials, signed HttpOnly admin session cookie, counts, status tabs, search, approve/reject/delete, and refresh.
- `/team`: approved-only directory, search, domain/position filters, profile links, and empty/loading/error states.
- Premium dark UI with lime accents, responsive cards, subtle motion, and clear visual hierarchy.
- Server-side Supabase access: the service-role key is never included in browser JavaScript. Admin updates are checked by the server.

## Requirements

- Node.js 20 or newer
- A free Supabase project
- A Vercel account for deployment

## Run locally

1. Install Node.js if it is not already installed: https://nodejs.org/
2. Open a terminal in this folder.
3. Run `npm install`.
4. Copy `.env.example` to `.env.local`.
5. Fill in `NEXT_PUBLIC_SUPABASE_URL` and `SUPABASE_SERVICE_ROLE_KEY` from Supabase Project Settings → API. Set `ADMIN_SESSION_SECRET` to a long random secret. Keep `.env.local` private; never commit it.
6. In Supabase, open SQL Editor → New query, paste all of `supabase/setup.sql`, and run it.
7. Run `npm run dev`.
8. Open http://localhost:3000.

If you have not configured Supabase yet, the pages can render but data submissions and directory loading will show a helpful setup error. This is expected until the database and environment variables are ready.

## Admin login

The challenge requires these reviewer credentials:

- Email: `admin@embrione.pes.edu`
- Password: `EmbrioneAdmin@2026`

The server defaults to these challenge values so reviewers can test the project. For production, override them with environment variables and use a strong `ADMIN_SESSION_SECRET`. The challenge password is known to reviewers, so do not use it for any other service. The session expires after eight hours.

## Database and storage

The SQL script creates `public.members`, makes SRNs unique, constrains semester/status values, enables RLS, creates a public `member-photos` storage bucket that accepts WebP images up to 2 MB, and inserts six fictional approved demo profiles. Demo records use `DEMO-EMB-*` SRNs and example.com email addresses. They are sample data, not real Embrione members.

The app intentionally does **not** copy the brief's broad public update/delete policies. The application uses the Supabase service-role key only on the server, so browser visitors cannot directly approve or delete records. The public directory is served through a server endpoint that returns only approved members. Public storage is required so profile pictures can display in the directory; do not put private documents in this bucket.

## Deploy to Vercel

1. Create a public GitHub repository and push this folder's code.
2. In Vercel, import the repository.
3. Add the same environment variables from `.env.local` in Vercel Project Settings → Environment Variables. Add them for Production (and Preview if needed).
4. Deploy and open the generated domain.
5. If the challenge requires a custom subdomain, set the project domain to `https://yourfirstname-embrione-task.vercel.app` (or the documented SRN-suffix fallback) in Vercel project settings.
6. Test registration, admin login, approve/reject, and the team directory on the deployed site—not only on localhost.

## Demo video checklist (about 2 minutes)

1. Open `/register`, type a name and bio, and show the card updating live.
2. Choose a PNG/JPG photo. Explain that the browser scales it to at most 800 pixels and converts it to WebP before upload. In DevTools Network, show the request carrying a `.webp` file.
3. Submit a profile and show the pending confirmation.
4. Open `/admin`, sign in with the challenge credentials, and show the pending counter and submission.
5. Approve the profile.
6. Open `/team`, search for the member, and show that the approved profile appears. Optionally reject another profile and show that it is absent.

## Git commit suggestion

Use small, truthful commits as features are completed, for example:

- `chore: scaffold Next.js app`
- `feat: create member registration form`
- `feat: add live member card preview`
- `feat: convert profile photos to WebP`
- `feat: connect Supabase registration API`
- `feat: add protected admin moderation`
- `feat: build approved team directory`
- `style: polish responsive dark UI`
- `docs: add setup and demo instructions`

Only make commits that match the work actually done; do not fake a development history.

## Troubleshooting

- `Supabase is not configured`: check `.env.local`, then restart `npm run dev` after editing environment variables.
- `Invalid API key` or Supabase errors: use the Project URL and **service_role** key from the same Supabase project. The service-role key must remain server-only.
- Images upload but do not display: confirm the `member-photos` bucket exists and is public, and that the uploaded file is WebP.
- No members show: run the sample seed section in `supabase/setup.sql`, then check that the demo records have `status = 'approved'`.
- Admin cannot sign in: check `ADMIN_EMAIL` and `ADMIN_PASSWORD` in `.env.local`; restart the server after changing them.
- Vercel fails while local works: make sure every required environment variable is configured in Vercel and redeploy.

## Important limitations

This is a recruitment-challenge implementation, not a production identity platform. The admin account is a single challenge account, not per-person accounts with MFA. Add rate limiting, audit logs, stronger operational controls, and a real identity provider before using it for sensitive real-world information. The service-role key bypasses RLS, so it must never be imported by a client component or exposed as a `NEXT_PUBLIC_` variable.
