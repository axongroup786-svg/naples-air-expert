# Naples Air Experts — Deploy Guide (Netlify & Cloudflare)

Ek hi codebase dono platforms par chalta hai. Neeche step-by-step tareeqa hai.

---

## 0) ZIP se shuru karein

ZIP mein yeh sab shamil hai:

```
/ (project root)
├── netlify.toml            Netlify build + headers + redirects
├── wrangler.toml           Cloudflare Pages (OpenNext) config
├── .env.example            environment variables ka template
├── next.config.ts          (trailingSlash = true — clean URLs)
├── package.json
├── src/assets/images/*.jpg  optimized site imagery (bundled as modules)
└── src/
    ├── app/                18 pages + sitemap.ts + robots.ts + API routes
    ├── components/         header, footer, forms, schema, page template
    ├── lib/                site data + 12 service page contents + assets
    └── db/                 Drizzle ORM schema + Postgres client
```

ZIP extract karein, phir:

```bash
npm install
cp .env.example .env       # aur DATABASE_URL set karein
npm run build              # local verify
```

---

## 1) NETLIFY (aasan tareeqa — recommended)

### Option A: GitHub se (auto deploy)
1. Code ko GitHub par push karein.
2. [app.netlify.com](https://app.netlify.com) → **Add new site → Import an existing project** → GitHub repo chunein.
3. Netlify khud `netlify.toml` detect kare ga:
   - Build command: `npm run build`
   - Publish directory: `.next`
4. **Site settings → Environment variables**:
   - `DATABASE_URL` = aap ka Postgres connection string (Neon/Supabase)
   - `LEAD_WEBHOOK_URL` = (optional) form fallback
   - `NEXT_PUBLIC_SITE_URL` = `https://aapka-domain.netlify.app`
5. **Deploy site**.

### Option B: Manual drag & drop
1. `npm run build` locally chalayein.
2. Netlify dashboard → **Add new site → Deploy manually** → poora project folder drag karein (node_modules/ aur .next/ ko chhod dein, ZIP already exclude kar chuka hai).
3. Environment variables phir se set kar lein.

### Netlify ke baad
- **Domain settings → Domain management** mein custom domain add karein aur DNS point karein.
- Redirects (`/ac-repair` → `/ac-repair-naples-fl/`) `netlify.toml` mein pehle se hain.
- Form leads `POST /api/lead/` par aati hain aur Postgres `leads` table mein save hoti hain.

---

## 2) CLOUDFLARE PAGES

Cloudflare Workers runtime hai, is liye Next.js App Router ko **OpenNext adapter** se chalana parta hai.

### Option A: Git integration (recommended)
1. Repo ko GitHub par push karein.
2. Cloudflare dashboard → **Workers & Pages → Create → Pages → Connect to Git**.
3. Build settings (wrangler.toml auto-read hoga):
   - **Framework preset:** Next.js
   - **Build command:** `npx @opennextjs/cloudflare build && opennextjs-cloudflare build`
   - **Build output directory:** `.open-next/`
   - **Node version:** 22
4. **Settings → Variables and Secrets**:
   - `DATABASE_URL` (encrypt karein)
   - `LEAD_WEBHOOK_URL` (optional)
   - `NEXT_PUBLIC_SITE_URL`
5. **Wrangler > Compatibility flags** mein `nodejs_compat` enable karein (`wrangler.toml` mein already set hai).
6. Deploy.

### Option B: CLI se
```bash
npm install -D @opennextjs/cloudflare wrangler
npx @opennextjs/cloudflare build
npx opennextjs-cloudflare deploy
```

### Cloudflare + Postgres ka dhyan rakhein
Workers mein direct TCP Postgres connection limited hota hai. Do reliable options:

1. **`LEAD_WEBHOOK_URL` use karein (sabse aasaan)** —
   Formspree / Make.com / Zapier / Google Apps Script webhook lagayen.
   Agar DB reachable nahi, form khud webhook par lead bhej deta hai,
   aur user ko phir bhi success + phone number dikhata hai.
2. **Cloudflare Hyperdrive** — Postgres (Neon/Supabase) ke liye connection
   pooling proxy. `DATABASE_URL` Hyperdrive ke bound endpoint se replace karein.

> Note: Netlify par yeh masla nahi — wahan Node runtime hai aur `DATABASE_URL` seedha chalta hai.

---

## 3) Deploy ke baad checklist

- [ ] `/` par site khuli, hero image load ho rahi
- [ ] `tel:+12397654321` buttons kaam kar rahe hain (mobile par click-to-call)
- [ ] Estimate form submit ho raha hai aur success message aa raha hai
- [ ] Lead `leads` table mein insert ho rahi hai (ya webhook par)
- [ ] `/sitemap.xml` aur `/robots.txt` 200 de rahe hain
- [ ] `NEXT_PUBLIC_SITE_URL` real domain par set hai (canonical + schema ke liye)
- [ ] Google Maps embed 3899 Mannix Dr dikha raha hai
- [ ] Custom domain + SSL enable

---

## 4) SEO / schema already built-in

| Item | Location |
|---|---|
| HVACBusiness / LocalBusiness JSON-LD | `src/app/layout.tsx` → `businessSchema()` |
| FAQPage JSON-LD | har page par `faqSchema()` |
| Service + BreadcrumbList JSON-LD | `src/components/service-page.tsx` |
| Canonical tags | har page ka `metadata.alternates.canonical` |
| XML sitemap | `src/app/sitemap.ts` → `/sitemap.xml` |
| robots.txt (+ AI crawlers) | `src/app/robots.ts` |
| Favicon | `src/app/icon.svg` |

Domain badalne par sirf `src/lib/site.ts` mein `BUSINESS.url` update karein aur
`.env` / platform variables mein `NEXT_PUBLIC_SITE_URL` set karein.

---

## 5) Contact

**Naples Air Experts** · 3899 Mannix Dr, Naples, FL 34114 · (239) 765-4321
