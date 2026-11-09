# Deployment Fix Report — Vercel Root Directory

## Failure from supplied Vercel build log

The build reached `npm run build` and executed `next build`, then failed with:

`Couldn't find any pages or app directory. Please create one under the project root`

This is a project-root/layout failure. The supplied project archive stores the application under a top-level `Business-Flow/` directory. If Vercel executes the build from a repository directory that contains the package manifest but not `src/app/` (or `app/`), Next.js cannot locate the App Router.

## Fix applied to this distribution

This distribution is flattened so all Next.js root files are at the archive root:

- `package.json`
- `vercel.json`
- `next.config.ts`
- `src/app/layout.tsx`
- `src/app/page.tsx`

The cron configuration remains daily (`0 20 * * *`) for Hobby compatibility.

## Required GitHub / Vercel alignment

1. Extract the contents of this ZIP directly into the root of the GitHub repository (not into an extra `Business-Flow/` folder).
2. Commit and push the resulting root-level project files.
3. In Vercel → Project → Settings → Build and Deployment, set **Root Directory** to `.` (repository root).
4. Use the Next.js framework preset, Node.js 22.x, default Next.js output directory, and `npm run build`.
5. Redeploy the same branch/commit after confirming the files appear at the GitHub repository root.

Alternative only when keeping the nested layout: set Vercel Root Directory to `Business-Flow`. Do not combine this alternative with the flattened layout.

## Validation status

- Verified project structure contains root `package.json`, `vercel.json`, `next.config.ts`, and `src/app/layout.tsx`.
- Verified `package.json` and `vercel.json` parse as JSON.
- Verified shell scripts pass `bash -n` and JavaScript `.mjs` files pass `node --check`.
- Verified ZIP archive integrity after packaging.
- A local `npm install` attempt timed out in this environment; therefore a full Next.js build, typecheck, lint, database connection, and Vercel deployment have not been verified here.

## Scope

This fix addresses the exact error shown in the supplied build log. It does not prove that later compilation, environment variables, database connectivity, SMTP delivery, or runtime requests are error-free.
