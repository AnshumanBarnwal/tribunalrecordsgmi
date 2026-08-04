# G.M.I. Tribunal Archive

A staff-only case-file and audit-log archive for tracking Discord server moderation actions, backed by Supabase (Postgres + Auth) with Row Level Security.

## Structure

```
.
├── index.html                    # The app — single static page, deployable as-is
├── supabase/
│   ├── schema.sql                 # Tables + Row Level Security policies
│   └── seed_staff.sql             # Links Supabase Auth users to staff roles
├── scripts/
│   ├── migrate-data.js            # One-time import of legacy JSON records
│   └── package.json
└── .github/workflows/deploy-pages.yml   # Auto-deploys index.html to GitHub Pages on push to main
```

## One-time setup

1. **Create a Supabase project** at [supabase.com](https://supabase.com) (free tier is enough for this scale).
2. **Run `supabase/schema.sql`** in the Supabase SQL Editor — creates `staff_profiles`, `case_files`, `offenses`, `audit_log`, and their RLS policies.
3. **Create staff accounts** under Authentication → Users, using the synthetic emails `seed_staff.sql` expects (e.g. `proctor@staff.gmi.internal`, `wicked@staff.gmi.internal`), each with a real, private password.
4. **Run `supabase/seed_staff.sql`** in the SQL Editor to link each account to its role and permissions.
5. **(Optional) migrate old data:**
   ```bash
   cd scripts
   npm install
   SUPABASE_URL=https://xxxx.supabase.co SUPABASE_SERVICE_ROLE_KEY=your-service-role-key npm run migrate
   ```
   Run this locally only — never put the service_role key in `index.html` or commit it anywhere.

`index.html` is already wired to this project's URL and public (anon/publishable) key. That key is safe to have in client-side code and in the public repo — Row Level Security in `schema.sql` is what actually controls who can read or write which rows, not secrecy of that key.

## Deploying to GitHub Pages

1. Push this repo to GitHub.
2. In the repo, go to **Settings → Pages → Build and deployment → Source: GitHub Actions**.
3. Push to `main` (or run the workflow manually) — `.github/workflows/deploy-pages.yml` publishes the site automatically.
4. Your archive will be live at `https://<username>.github.io/<repo>/`.

## Security notes

- The original prototype exposed every staff password in plaintext directly on the login screen (quick-fill buttons + a "Staff Passkey Directory" modal, visible with no login required). That has been removed. Authentication now goes through Supabase Auth exclusively — passwords live only in Supabase's Auth system, never in this repo or the deployed page.
- Manage staff passwords going forward from the Supabase dashboard (Authentication → Users), not in code.
- `case_files`, `offenses`, and `audit_log` access is enforced server-side via the RLS policies in `schema.sql` — a signed-in user only sees what their `staff_profiles` row permits, regardless of what the UI shows.
- Never commit the `service_role` key (from Settings → API) anywhere in this repo — it bypasses RLS entirely and is only for the local one-time migration script.
