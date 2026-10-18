# High Res

An Astro + EmDash theme for the minimal `tehtarik` photo journal. It preserves the original single-column feed, five-at-a-time loading, captions, dates, and source image order while switching the public typography to Inter.

## What is in the migration

- The 15 Jekyll photo records are represented as the `photos` EmDash collection in `seed/seed.json`. The public feed uses the same records as a read-only fallback until initial CMS setup is complete, so no original content disappears during setup.
- The original local images are retained under `public/img/` as a safe first-deploy fallback. The remote Teh Tarik image is retained as its original URL.
- New photography belongs in the EmDash Media Library: assign it to **High-resolution image** in a Photo entry. On Cloudflare, the `Image` component uses the private R2 binding and Cloudflare image transformations for responsive variants.
- `source_url` is deliberately kept only to render the migrated legacy records before those originals are uploaded to the Media Library. It can be cleared per record after import.

## Local development

```sh
npm install
npm run dev
```

Open `http://localhost:4321/_emdash/admin`. The first visit starts the EmDash setup wizard. For local development, use `wrangler dev` if you need the actual D1/R2 bindings.

## Cloudflare deployment

1. In `wrangler.jsonc`, keep the Worker, D1, and R2 names unique to this site. The supplied bindings are `DB` and `MEDIA`; they match `astro.config.mjs` and must not be renamed independently.
2. Authenticate Wrangler, then run `npm install` and `npm run deploy`. The initial deploy provisions the named D1 database and R2 bucket when they do not already exist.
3. Open the Worker URL once to apply EmDash's initial schema/seed, then visit `/_emdash/admin` on the final `tehtarik.net` domain and complete the setup wizard. Do this on the final domain because passkeys are domain-bound.
4. Upload every original high-resolution image in Media Library, choose it in the corresponding Photo entry, publish, and only then clear that record's **Legacy image URL**. Keep R2 private unless a deliberately restricted media-only public origin is needed.
5. Add the custom domain in Cloudflare after the Worker URL responds. Set `EMDASH_SITE_URL` as a Worker variable before setup if the final domain is already known.

`src/worker.ts` and the one-minute Cron Trigger are included for EmDash maintenance and scheduled publishing. After deployment, verify a public page, admin sign-in, one media upload/retrieval, and the scheduled handler with `npx wrangler tail`.

## GitHub integration

Connect `opensos/high_res` to Cloudflare Workers Builds (or deploy from a GitHub Action with Cloudflare credentials). Use `npm ci && npm run build` as the build command and `npx wrangler deploy` as deployment command. Do not point branch previews at production D1/R2: EmDash's documentation requires separate preview bindings.

The repository intentionally contains no Cloudflare account IDs, API tokens, or secrets.
