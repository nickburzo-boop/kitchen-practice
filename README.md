# Kitchen Practice

Guided home-cooking sessions for everyone, with custom recipes, six built-in practice paths, timers, temperature and texture observations, and private account notebooks.

Live app: https://kitchen-practice.nickburzo.workers.dev/

## Architecture

- Cloudflare Worker serves the UI and same-origin JSON API.
- Cloudflare D1 stores accounts, hashed sessions and user-scoped notebooks.
- Username/password login with PBKDF2-SHA256 password hashes, per-user random salts, secure HttpOnly cookies and recovery codes.
- Login rate limiting, origin checks, parameterized SQL and revision checks on notebook writes.
- Guest mode works on the current device. Signed-in users sync recipes, history and current session, with local pending changes retained through reloads.
- Recipe instructions are supplied by the user; the app formats them into guided steps and detects minute-based timer suggestions. It does not generate recipes with AI.

## Deploy

Install Node.js and Wrangler, then authenticate to your Cloudflare account:

```sh
npm install
npx wrangler login
npx wrangler d1 execute kitchen-practice --remote --file schema.sql
npm run deploy
```

The checked-in Wrangler configuration targets the existing `kitchen-practice` D1 database and Worker. For another account, create a new D1 database and replace its ID in the config. Database IDs are resource identifiers, not credentials. Do not commit API tokens, passwords, cookies or recovery codes.

`worker.mjs` is a self-contained deployment with the UI and API. `index.html` is an editable copy of the UI; update the embedded `page` string in `worker.mjs` when changing it. The Worker hosts the application; GitHub Pages cannot run the D1 API.

## Accounts and data

Passwords require 12–128 characters. Users must keep their recovery code; no email service is configured. Recovery rotates the code and revokes existing sessions. Sessions expire after seven days. Each account is limited to 300 recipes, 1,000 sessions and a 500 KB notebook. Notebook conflicts stop syncing until the user exports local work or loads the cloud version.

Keep the page open for timer alerts. Recipe edits record the setup but do not automatically calculate new cooking temperatures or ingredient ratios.
