# Top Tech Development

A complete React/Vite company website with Express, Node.js, Tailwind CSS, Motion, and a MySQL inquiry backend. One root package.json, one dependency installation, one server in production. MongoDB is presented as a service capability; this website itself uses MySQL as requested.

## Quick start

Use Node.js 22 or 24.

```sh
npm install
cp .env.example .env
npm run dev
```

Open http://localhost:5173. The frontend proxies `/api` to Express on port 3000. For local development set `NODE_ENV=development` and `PUBLIC_ORIGIN=http://localhost:5173`. Fill in MySQL credentials, create the database, then run `npm run db:setup`. Without a configured database the website still renders, but the inquiry form returns a clear unavailable message instead of claiming success.

## Production

```sh
npm ci --include=dev
npm run build
npm start
```

Express serves both `dist` and `/api` from the same origin. Do not deploy only `dist` if you want inquiries to work. The bundled `dist` is included for convenience; rebuild after changing source.

## Vercel

The project deploys to Vercel as-is: Vercel builds the Vite frontend into `dist`, and the inquiry API runs as serverless functions in `api/` (`api/inquiries.js`, `api/health.js`). They share the same validation and database code as the Express server (`server/inquiry.js`), so Hostinger deployment keeps working too. `vercel.json` sets the build and applies the same security headers that Helmet applies on Hostinger.

1. Push this folder to a GitHub repository (`node_modules`, `.env` and `.vercel` are ignored).
2. In Vercel choose **Add New → Project**, import the repository. The framework is detected as Vite; leave build command `npm run build` and output `dist`.
3. Add environment variables (Project → Settings → Environment Variables): `PUBLIC_ORIGIN` (your exact production URL, e.g. `https://toptech.vercel.app` or your custom domain, no trailing slash), `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_SSL`. Do not set `PORT` or `TRUST_PROXY_HOPS` on Vercel.
4. Deploy, then open `/api/health` and submit a test inquiry.

The database must accept connections from the internet, because Vercel functions do not run on your database host and have no fixed IP. With Hostinger MySQL, enable **Remote MySQL** in hPanel for the database and allow any host (`%`), and use the remote hostname Hostinger shows there as `DB_HOST` (not `localhost`). Any hosted MySQL works the same way. Import `schema.sql` once.

Rate limiting on Vercel is per function instance and therefore best effort. For a firm limit, add a Vercel Firewall rate-limit rule for `/api/inquiries`. Deployment previews can submit to themselves; requests from other origins are rejected when `PUBLIC_ORIGIN` is set.

Alternatively, from this folder: `npx vercel` (preview) and `npx vercel --prod`.

## Hostinger, one root folder

1. Extract this ZIP. Use the folder containing `package.json` as the project root. No separate client/server installations are needed.
2. In Hostinger choose **Node.js Web App**, upload the project ZIP or connect its GitHub repository. Choose the Express/Node.js server deployment option, not a static Vite-only deployment.
3. Select Node.js 22 or 24. Install command, if shown: `npm ci --include=dev`. Build command: `npm run build`. Start command: `npm start`. Entry file, if requested: `server/index.js`. Keep the application root at the folder containing package.json. The Vite frontend output is `dist`, but the Node.js server must remain the application runtime.
4. Create a MySQL database and user in the Hostinger database panel. Open phpMyAdmin for that database and import `schema.sql`. Alternatively run `npm run db:setup` where Node commands are available.
5. Add the variables from `.env.example` to the Node.js app environment. Use your real database hostname, database name, username, and password. Set `PUBLIC_ORIGIN` to your exact HTTPS domain without a trailing slash. Never put database credentials in VITE_ variables or commit `.env`.
6. Let the hosting platform supply PORT if it does. Confirm the reverse-proxy configuration with Hostinger before setting `TRUST_PROXY_HOPS`. Do not blindly enable trust for every proxy.
7. Deploy or restart the app. Open `/api/health`, then submit a test inquiry and confirm a new row appears in the `inquiries` table in phpMyAdmin.

Hostinger interfaces may vary by plan. Deployment settings were checked against these official guides:
- https://www.hostinger.com/support/how-to-deploy-a-nodejs-website-in-hostinger/
- https://www.hostinger.com/support/connecting-a-hostinger-mysql-database-to-a-node-js-application/
- https://www.hostinger.com/support/how-to-add-environment-variables-during-node-js-application-deployment/

## Viewing inquiries

Use phpMyAdmin to browse the `inquiries` table. Locally or on a server with command access, `npm run leads` prints the latest 100 inquiries. There is no public endpoint exposing customer submissions. Email notifications and CRM delivery are not configured; submissions are stored in MySQL. Review your inbox regularly.

## Editing

- `src/data.js`: all copy — projects, services, process steps, FAQ, form options (keep form options in sync with `server/validation.js`).
- `src/components/sections/`: one file per page section (Nav, Hero, Marquee, Work, Services, Growth, Approach, Faq, Contact, Footer).
- `src/components/ui/`: reusable pieces — `StackingCards` and `Spotlight` (adapted from 21st.dev components), plus `primitives.jsx` (magnetic buttons, text reveals, scroll-lit text, headings, brand mark).
- `src/styles.css`: design tokens (colors, fonts, radius) at the top, then section styles.
- `server/inquiry.js`: inquiry handling shared by both runtimes. `server/index.js`: Express server (Hostinger). `api/`: Vercel functions. `server/validation.js`: validation rules.

Design system: dark warm-stone palette with the logo's blue → violet → ember gradient as the accent; Geist (UI), Instrument Serif italic (accents) and Geist Mono (labels), all self-hosted via Fontsource so the strict CSP still applies. Motion uses Framer Motion (`motion` package) and Lenis smooth scrolling; both respect the visitor's reduced-motion setting.

The four portfolio cards use the supplied live project URLs and homepage preview images; a branded artwork placeholder shows until each preview loads. No testimonials, performance results, or ownership endorsements have been fabricated.

No phone number, business mailbox, WhatsApp number, or social profile was invented. Add these once confirmed. The privacy copy describes the actual inquiry collection; update it when you add email providers, analytics, tracking, or a retention policy. Domain/hosting and campaign ad spend are separate from website code.

## Included protections

Parameterized database writes; server-side field validation; request body limits; rate limiting; honeypot; same-origin check when PUBLIC_ORIGIN is set; Helmet headers; no client database credentials; no public inquiry listing; accessible FAQ disclosure; reduced-motion support; keyboard-friendly mobile navigation.

## Validation

`npm test` checks inquiry validation. `npm run build` produces the production frontend. Test production database connectivity and an actual saved submission after adding your Hostinger credentials. This package has not been deployed to your Hostinger account.

## Offline preview and screenshots

Double-click `OPEN-PREVIEW.html` to browse the design without installing anything. Its animations and interactions work offline (keep the `dist` folder next to it); the inquiry form requires the running Express server and MySQL, so use `npm run dev` for that. Desktop and mobile screenshots are included in `previews/`.

`node scripts/check-browser.mjs` (requires Playwright) checks 390px, 768px, and 1366px widths: no horizontal overflow, no JavaScript errors, FAQ, privacy disclosure, mobile navigation, invalid-request rejection, and honest unavailable-database feedback. A real database insertion still needs verification on your configured database.
