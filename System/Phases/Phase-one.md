```markdown
 ## Phase 1: Foundation (once)

- [ ] **Database tables** (MySQL, see `schema.sql`): `products`, `product_assets`, `orders`, `campaigns`, `content_posts`, `weekly_metrics`, `insights`
  - [ ] Plus supporting tables: `stripe_events` (webhook dedupe), `admin_users` (admin login)
  - [ ] Run `schema.sql` and seed one test product

- [ ] **Website** (Node/Express + MySQL), product listing page + product detail page template, fed from the DB
  - [ ] Home page: hero, product catalog, FAQ, footer, navbar, responsive (mobile-first)
  - [ ] Product listing page (active products only)
  - [ ] Product detail page template (by slug, `Product` JSON-LD, fires `view_item`)
  - [ ] Admin area, private access by password
    - [ ] Login (argon2id hash, session cookie, rate limit, CSRF)
    - [ ] Dashboard of all operations (orders, revenue, top products, weekly metrics)
    - [ ] CRUD: products and assets, campaigns, content posts, insights
  - [ ] Stripe callback API routes (`/checkout/:slug`, `/stripe/webhook`, success and cancel pages)
  - [ ] Connected to MySQL to receive new products and updates (admin CRUD writes to `products`; optional token-protected `POST /api/products`)
  - [ ] **Responsive**
    - [ ] Mobile-first CSS, tested at 360px, 768px, 1280px widths
    - [ ] `<meta name="viewport">`, fluid grid (CSS grid/flex), no horizontal scroll
    - [ ] Touch targets at least 44px, readable font sizes, hamburger navbar on mobile
    - [ ] Responsive images (`srcset`/`sizes`, WebP or AVIF)
  - [ ] **Fast** (targets: LCP under 2.5s, CLS under 0.1, INP under 200ms, Lighthouse mobile 90+)
    - [ ] Server-render pages (EJS/Nunjucks), minimal JS, no heavy frameworks on public pages
    - [ ] Serve images and assets from a CDN (CloudFront/Cloudflare) with long cache headers and hashed filenames
    - [ ] gzip/brotli compression (`compression` middleware or at the CDN)
    - [ ] Cache listing and detail pages (in-memory or Redis, short TTL, invalidate on admin save)
    - [ ] MySQL: indexes on `products.slug`/`status`, connection pool, select only needed columns
    - [ ] Lazy-load below-the-fold images, set image width/height, preload the hero image
    - [ ] Load GA with `async`, self-host or `font-display: swap` fonts
    - [ ] Check with Lighthouse/PageSpeed on mobile before launch
  - [ ] **SEO structure** (professional, server-rendered so crawlers see full HTML)
    - [ ] URL structure: clean, lowercase, hyphenated slugs (`/`, `/products`, `/products/:slug`, `/faq`, `/blog/:slug`); 301 redirect old slugs when a product slug changes; one canonical host (www or non-www, HTTPS only)
    - [ ] Per-page `<title>` (under 60 chars) and meta description (under 160), unique for every page, editable in admin (add `seo_title`, `seo_description` columns to `products`)
    - [ ] One `<h1>` per page, logical `h2`/`h3` hierarchy, descriptive image `alt` text
    - [ ] `<link rel="canonical">` on every page; `noindex` on `/admin`, checkout, success and cancel pages
    - [ ] Open Graph and Twitter Card tags (title, description, image 1200x630) so shared links look right
    - [ ] Structured data (JSON-LD): `Organization` + `WebSite` on home, `Product` + `Offer` (price, currency, availability) on product pages, `FAQPage` on FAQ, `BreadcrumbList` on inner pages
    - [ ] `sitemap.xml` generated from the DB (active products only, `lastmod` from `updated_at`), referenced in `robots.txt`
    - [ ] `robots.txt`: allow public pages, disallow `/admin`, `/checkout`, `/stripe`, `/api`
    - [ ] Internal linking: navbar, breadcrumbs, related products, footer links; no orphan pages
    - [ ] Custom 404 page (with real 404 status) and no soft 404s
    - [ ] Core Web Vitals pass (covered under Fast), mobile-friendly
    - [ ] Content plan: blog or guides targeting buyer keywords, one primary keyword per page
    - [ ] Submit sitemap in Search Console and monitor Coverage and Core Web Vitals reports

- [ ] **Stripe**: webhook endpoint (`checkout.session.completed`) that records the order and sends the buyer an expiring S3 signed download link by email
  - [ ] Verify signature using the raw request body
  - [ ] Dedupe by event id (`stripe_events`) and `stripe_session_id` on `orders`
  - [ ] Generate presigned S3 URLs (for example 24h), send email, set `download_link_sent_at`
  - [ ] "Resend link" action in admin
  - [ ] Test locally with `stripe listen`

- [ ] **Data collection and privacy** (visitors, buyers, leads, accounts; tables in `schema.sql`)
  - [ ] Buyer data: capture email, name, country, billing/tax details from the Stripe session into `customers` and `orders`
  - [ ] Visitor behavior: first-party visitor cookie, `visitor_sessions` (landing page, referrer, UTMs, device, country) and `visitor_events` (page_view, view_item, begin_checkout, purchase), alongside GA
  - [ ] Newsletter/lead signup forms (double opt-in, source tracked) feeding `newsletter_subscribers`
  - [ ] User accounts: signup/login, order history and re-download page, password reset, email verification
  - [ ] Cookie consent banner: analytics and marketing off until accepted; store choices in `consent_records`
  - [ ] Privacy policy, terms, and cookie policy pages (linked in the footer and checkout)
  - [ ] Collect only what you use; do not store raw IPs or card data (Stripe holds payments)
  - [ ] User rights: export and delete a customer's data from admin (GDPR/CCPA requests), unsubscribe link in every email
  - [ ] Dashboard views: traffic by source, conversion funnel, new vs returning, subscribers growth

- [ ] **Copywriting standard** (all site text sounds like a person wrote it; uses the humanizer skill from github.com/blader/humanizer, saved at `skills/humanizer/SKILL.md`)
  - [ ] Applies to every piece of text: home hero, product titles and descriptions, FAQ, footer, navbar labels, buttons, form labels, error and 404 pages, consent banner, privacy/terms/cookie pages, and all emails (download link, resend, password reset, newsletter)
  - [ ] Write a short voice guide first: who the reader is, how you talk to them, 3 to 5 sample sentences in your own voice, words you use and words you never use
  - [ ] Draft each text, then run the humanizer pass: mark the tells, rewrite, check, and search again for the ones that survive
  - [ ] Cut these on sight: "not just X, but Y" contrasts, one-line closers ("That's the real win"), "Let's dive in" openers, forced triads, em dashes, stacked qualifiers, inflated words (pivotal, seamless, robust, elevate, unlock, game-changing), sales filler ("cutting-edge", "world-class"), chatbot phrases ("I hope this helps"), bold on every label, emojis as decoration
  - [ ] Say specific things: what the product is, what the buyer gets, file format, size, and what happens after payment; no claims you cannot back up
  - [ ] Product descriptions: one plain sentence on what it is, who it is for, what is inside, then details; no invented reviews, numbers or testimonials
  - [ ] Keep legal text (privacy, terms) plain and accurate; humanize the wording only, never change what it commits you to, and have a lawyer review it
  - [ ] Check SEO still works after rewriting: primary keyword in title, `h1` and first paragraph, titles under 60 characters, descriptions under 160
  - [ ] Store editable copy in the DB or admin (not hardcoded in templates) so it can be revised without a deploy
  - [ ] Final read-aloud review of every page before launch; anything that sounds like a brochure gets rewritten

- [ ] **Tax decision**: confirm Stripe Tax is on, or move to a merchant of record (Lemon Squeezy / Paddle) for VAT/sales-tax handling
  - [ ] Decide before building checkout (a merchant of record replaces Stripe, not just adds to it)

- [ ] **Analytics**: GA events (`view_item`, `begin_checkout`, `purchase`) and Search Console verified
  - [ ] `purchase` deduplicated with `transaction_id` = order id
  - [ ] Sitemap submitted

- [ ] **UTM convention**: `utm_source` (youtube|instagram|tiktok|seo|email), `utm_medium`, `utm_campaign` = product slug
  - [ ] Capture UTMs in a first-party cookie on landing and copy them onto the order

- [ ] **Create** `System/DataBase/Insights.md` (weekly log mirroring the `insights` table: date, what the metrics showed, action decided)


```

