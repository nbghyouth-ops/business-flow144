# Vercel root-directory fix

This archive is deliberately flattened: `package.json`, `vercel.json`, `next.config.ts`, and `src/app/` are directly in the project root.

## GitHub layout required

The repository root must look like this:

```
package.json
vercel.json
next.config.ts
src/app/layout.tsx
src/app/page.tsx
```

Do not commit this project one level deeper under `Business-Flow/` unless you also set Vercel's **Root Directory** to `Business-Flow`.

## Vercel project settings

- Root Directory: `.` (repository root)
- Framework Preset: Next.js
- Build Command: `npm run build` (or leave the default)
- Install Command: `npm install` (or leave the default)
- Output Directory: leave blank / default for Next.js
- Node.js Version: 22.x

After changing the repository layout or Root Directory, redeploy from the intended branch.

## Important

A successful dependency installation does not prove the app compiled. Confirm the Vercel build reaches the Next.js compilation and route-generation stages. Database, SMTP, and production environment values must also be configured in Vercel before runtime features are tested.
