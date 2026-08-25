# SCAM Insurance

SCAM Insurance is a classroom security-awareness simulation. It looks like a fictional customer portal, but it only accepts a freely typed name, a selected fictional demo value, a manually selected location, and the database-generated submission time.

**This project intentionally does not collect real passwords, IP addresses, browser fingerprints, or other unnecessary identifying information.** The five displayed values are fictional training values only. There is no free-text password field, analytics, tracking pixel, Google login, OAuth flow, or service-role key in the frontend.

## Project structure

```text
SCAM-INSURANCE/
├── index.html          # Audience simulation
├── admin.html          # Authenticated presenter dashboard
├── README.md
└── supabase/
	└── schema.sql      # Table, grants, RLS, and Realtime setup
```

## Supabase setup

1. Create a project at [supabase.com](https://supabase.com/).
2. In the Supabase dashboard, open **SQL Editor**, paste the contents of `supabase/schema.sql`, and run it.
3. The SQL enables the `demo_submissions` table for Realtime. Confirm it under **Database > Publications** and ensure `demo_submissions` is included in `supabase_realtime`.
4. Open **Authentication > Users** and create the presenter account. Use a dedicated classroom account, not a personal account. Email confirmation can be disabled for a local classroom project if desired.

## Configure the pages

Open `Project Settings > API` and copy the **Project URL** and public **anon/publishable key**. In both `index.html` and `admin.html`, replace the two values near the top of the script:

```js
const SUPABASE_URL = "https://YOUR_PROJECT_REF.supabase.co";
const SUPABASE_ANON_KEY = "YOUR_PUBLIC_ANON_OR_PUBLISHABLE_KEY";
```

The anon/publishable key is intended for browser use. Never place a Supabase `service_role` or secret key in either page. The presenter signs in with Supabase Auth; no administrator password is embedded in JavaScript.

## Security model

`supabase/schema.sql` enables Row Level Security. Anonymous and authenticated visitors can insert only the three permitted fields, and the database constraint accepts only `DEMO-PASS-01` through `DEMO-PASS-05`. Anonymous users have no select policy, so they cannot read submissions. Authenticated users can read the presenter table after Supabase Auth login. This is suitable for a controlled demonstration; for a production system, limit select access to a dedicated admin role rather than every authenticated user.

The admin **Clear visible table** button only clears the current browser view. It does not delete database rows. To permanently reset the demonstration, use the Supabase SQL Editor as an administrator:

```sql
delete from public.demo_submissions;
```

Do not grant audience users delete access.

## Deploy and present

1. Deploy `index.html` and `admin.html` together to a static host. GitHub Pages can serve these pages because the browser only needs the public Supabase client key. Do not put secrets in repository files. Netlify, Cloudflare Pages, or Vercel static hosting are also suitable.
2. Use the public `index.html` URL for the audience.
3. Open `admin.html` on the presenter laptop and sign in with the Supabase presenter account. Keep that page on the projector.
4. On another device, submit a name, one demo value, and a region. The new row should appear on the presenter board without a refresh.
5. If Realtime does not update, check the browser console, the project URL/key, the table's Realtime publication membership, and that the SQL ran successfully.

Because the pages connect directly from each device to Supabase, they work across devices and networks as long as the project is reachable. GitHub Pages cannot protect private frontend secrets, but none are required here: RLS and Supabase Auth enforce the data boundary. Use a server-backed host if you later add privileged operations or any secret integration.

## Classroom reset and testing

Test the complete flow with two browser windows or devices: authenticate on `admin.html`, submit from `index.html`, and verify that the row appears newest first. Try submitting without a name, without a demo selection, and without a location to confirm validation. Verify that the submitted values are always one of the five fictional options.

At the end of the session, use the SQL delete above to remove all demonstration submissions. You can also sign out from the admin page. Keep the presenter account separate from audience accounts and rotate or delete it after the event.

The simulation explains its purpose to participants after submission. It intentionally demonstrates persuasion without handling real credentials or collecting hidden device data.