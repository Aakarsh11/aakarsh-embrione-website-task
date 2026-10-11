# Explain the project like you're learning it for the first time

## The big picture

A website is what the visitor sees. A full-stack app also has server code and a database behind it.

- Frontend: the pages, forms, buttons, filters, and styles in the browser.
- Backend: the `/api/...` routes. They receive requests, check data, and decide what is allowed.
- Database: Supabase Postgres, which stores each member's details and status.
- Storage: Supabase Storage, which stores the actual photo files.
- Hosting: Vercel, which runs the deployed Next.js app.

A typical registration journey is: type details → preview updates in the browser → choose photo → browser converts photo to WebP → submit form → server validates values → server checks for duplicate SRN → server uploads image → server inserts a `pending` database row → browser shows success.

An approval journey is: admin signs in → server creates a signed HttpOnly cookie → admin page asks server for submissions → server checks cookie → admin clicks Approve → server updates `status` to `approved` → `/team` endpoint returns that member because its status is approved.

## Folder map

- `app/page.tsx`: landing page.
- `app/register/page.tsx`: registration form and live card preview.
- `app/admin/page.tsx`: admin sign-in and moderation dashboard.
- `app/team/page.tsx`: public directory, search, and filters.
- `app/globals.css`: visual design and responsive rules.
- `app/api/register/route.ts`: validates registration, uploads WebP, inserts member.
- `app/api/team/route.ts`: returns approved profiles only.
- `app/api/admin/login/route.ts`: checks challenge credentials and creates session cookie.
- `app/api/admin/logout/route.ts`: clears session cookie.
- `app/api/admin/members/route.ts`: protected list/update/delete actions.
- `lib/supabase-admin.ts`: creates server-only database client.
- `lib/admin-session.ts`: signs and verifies the admin session cookie.
- `lib/types.ts`: shared lists of valid domains, positions, branches, and member type.
- `supabase/setup.sql`: creates database and bucket, seeds sample profiles.
- `.env.example`: names of environment variables required to run the app.

## Important ideas in simple words

### A route
A route is an address. `/team` is a page route; `/api/team` is an API route. A page is designed for humans to view. An API route sends data between the browser and server.

### React state
`useState` stores a value while a page is open. For example, when the user types a name, `setForm` updates the form state. The preview reads the same state, so React redraws the relevant UI. This is why the preview changes before submission.

### `useEffect`
The team page uses `useEffect` to fetch approved members when the page first loads. It also cleans up the temporary photo preview URL when appropriate.

### FormData
`FormData` packages text fields and a photo into one HTTP request. It is useful for sending a file together with ordinary form fields.

### WebP conversion
The browser loads the chosen picture, draws it onto an HTML canvas at a maximum dimension of 800 pixels, then calls `canvas.toBlob(..., 'image/webp', 0.8)`. The result is a smaller WebP image. It is not uploaded while the user is merely previewing; upload happens after Submit.

### Database row
A row is one record in a table. In `members`, one row represents one member profile. The `status` column controls the review flow: `pending`, `approved`, or `rejected`.

### API validation
The server does not blindly trust browser form values. It checks required fields, allowed domain/position/branch, semester range, URL formats, image type/size, bio length, and duplicate SRN. Browser validation helps the user; server validation protects the data.

### Supabase service-role key
This is a powerful secret that allows the server to perform database operations. It must never be included in client-side code. The browser calls our API routes; the server calls Supabase. The SQL script therefore avoids giving anonymous browsers permission to update or delete member rows.

### Admin session cookie
After checking the challenge email/password, the server sets a signed cookie. `HttpOnly` means normal browser JavaScript cannot read it. `Secure` is enabled on HTTPS production. `SameSite=Lax` helps reduce some cross-site request risks. The cookie expires after eight hours. This is better than storing a password in browser local storage, but it is still a simple challenge login rather than a full identity provider.

### Filters
The directory receives only approved profiles from the server. Search and domain/position filters then narrow that list in the browser. Filtering does not approve or reject anything; it only changes what is visible in the current view.

### Environment variables
Environment variables are settings kept outside the source code. `.env.local` is for your computer; Vercel has a separate environment-variable settings page. The service-role key is server-only and must not start with `NEXT_PUBLIC_`.

## What makes this implementation stand out

- A coherent visual identity: dark forest-green surfaces, bright lime accents, editorial typography, and subtle dotted/orbit graphics.
- A landing page that explains the purpose instead of dropping users directly into a form.
- A two-step registration flow to make a long form feel easier.
- A live profile preview and browser-side image optimization.
- Real loading, error, success, and empty states rather than blank pages.
- Search and two independent filters in the team directory.
- Search, status tabs, counters, and responsive moderation cards in the admin dashboard.
- Duplicate SRN protection and database constraints.
- A server-checked admin session and no broad public database update/delete policies.
- Seeded fictional profiles so the team page looks alive immediately after setup.

## Before the demo

Do not claim the project is deployed or connected until you have actually configured Supabase and tested it. Run through the full flow once. Be ready to explain that the browser handles the live preview and WebP conversion, while the server validates the request, uploads the image, writes the row, and controls admin actions.
