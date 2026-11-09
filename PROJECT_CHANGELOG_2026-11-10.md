# Business Flow — combined patch 2026-11-10

## Referensi dan strategi penggabungan

- `Business-Flow-Vercel-Ready.zip` menjadi basis karena membawa hardening password/session, streaming request-size limits, protected worker scheduler, dan dokumentasi Vercel yang lebih baru.
- `business-flow-platform-patched(1).zip` menjadi referensi untuk SMTP/Nodemailer, OTP dev header, invitation email, richer health check, setup scripts, CORS allowlist, dan rate limit.
- Fitur dari basis yang lebih baru dipertahankan; patch lama tidak menimpa manajemen sesi, body-stream guard, guard `CRON_SECRET`, dan hardening auth.

## Perubahan

1. **Vercel Hobby:** `vercel.json` diubah dari `*/5 * * * *` menjadi `0 20 * * *` (20:00 UTC; sekitar 03:00–03:59 WIB) supaya jadwal paling banyak sekali sehari. Pro/Enterprise dapat mengembalikan `*/5 * * * *` jika dibutuhkan.
2. **SMTP:** Nodemailer helper terhubung ke register, resend/forgot, serta invitation. Production fail-closed bila SMTP tidak dikonfigurasi; kegagalan delivery dikembalikan sebagai HTTP 503 tanpa membocorkan detail provider.
3. **OTP:** Kode tidak dikirim di JSON. Header `X-Dev-OTP` hanya ada pada local development non-Vercel. API client local membaca header itu untuk menampilkan OTP di UI dev; pada production/Vercel kode tidak diekspos.
4. **CORS:** allowlist origin berdasarkan host aplikasi dan `NEXT_PUBLIC_APP_URL`, preflight `OPTIONS`, CORS headers pada success/error/idempotent replay.
5. **Rate limit dan request guard:** auth 10 request/15 menit per IP; workspace API 100/menit pada production; digest untuk key sensitif dan batas body streaming.
6. **Health/logging:** health endpoint menyajikan status, DB connection/latency, storage mode, auth/email, versi, uptime; error/request logging berupa JSON terstruktur.
7. **Database safety:** `scripts/setup.sh` tidak menjalankan schema push otomatis. `ALLOW_SCHEMA_PUSH=yes APPLY_SCHEMA=1` diperlukan dan didokumentasikan hanya untuk database development kosong.
8. **Deployment docs:** update `vercel.json`, `.env.example`, setup scripts, README, background jobs, deployment guide, blockers, readiness report, build/deploy report.
9. **Testing support:** E2E membaca header OTP dan menambahkan check untuk CORS preflight serta memastikan OTP tidak berada pada response JSON.

## Validasi

LULUS: JSON parse untuk empat konfigurasi; shell syntax dua setup scripts; JavaScript syntax pada secret script dan E2E runner; transpile syntax 58 file TypeScript/TSX tanpa diagnostic; cron config satu job harian di `/api/cron`; arsip akhir diuji dengan `unzip -t` sebelum diserahkan.

BELUM DILAKUKAN: npm dependency install/lockfile, typecheck penuh, lint, Next.js build, E2E runtime, DB migration/health HTTP test, SMTP delivery, Vercel link/deploy, cron runtime. Dependency registry tidak dapat dijangkau dari sandbox dan tidak ada database, SMTP, atau akses akun Vercel milik user.

**Status: source merge & cron compatibility patch complete; runtime/build verification incomplete; NOT READY FOR PRODUCTION.**
