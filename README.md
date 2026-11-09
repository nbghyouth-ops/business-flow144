# Business Flow

Business Flow adalah aplikasi web full-stack untuk pengelolaan usaha: Next.js 16 App Router, React 19, TypeScript, PostgreSQL, dan Drizzle ORM. Source mencakup API multi-tenant, tabel dengan Row-Level Security (RLS), modul penjualan/POS, persediaan, pembelian, keuangan, CRM, HR, operasional, marketing, proyek, aset, booking, serta 16 template usaha.

> **Status rilis:** paket ini belum boleh dipakai untuk transaksi/data pelanggan produksi. Lihat [`BLOCKERS.md`](./BLOCKERS.md) dan [`docs/PRODUCTION_READINESS.md`](./docs/PRODUCTION_READINESS.md). Email nyata, Google OAuth, Neon provisioning/Auth, migrasi SQL berversi, pemisahan credential database, backup/restore drill, dan security/performance audit eksternal belum lengkap.

## Persyaratan

- Node.js 22.x (direkomendasikan) dan npm 10+.
- PostgreSQL 15+ yang dapat dijangkau dari mesin aplikasi. Untuk Vercel, jangan gunakan host `localhost`; gunakan database remote dengan TLS/SSL dan pool/URL yang direkomendasikan penyedia database.
- `DATABASE_URL` (server-only) untuk database PostgreSQL; jangan gunakan prefix `NEXT_PUBLIC_`.
- SMTP (`SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`) wajib di production untuk verifikasi email, reset password, dan undangan anggota. Tanpa SMTP, OTP hanya tersedia pada local development melalui header dev dan ditampilkan UI dengan label DEV MODE; di Vercel/production kode tidak diekspos.
- Untuk scheduler Vercel: `CRON_SECRET` disimpan sebagai sensitive environment variable oleh script resmi di bawah; nilainya tidak ditulis ke file atau log.

## Instalasi lokal

Arsip ini tidak menyertakan `package-lock.json`. Oleh sebab itu jalankan `npm install` untuk bootstrap pertama, tinjau hasilnya, lalu commit `package-lock.json` sebelum CI/deployment reproducible. Versi mayor/minor dependency langsung di `package.json` dipatok.

```bash
npm install
cp .env.example .env.local
# Isi DATABASE_URL + SMTP (atau biarkan SMTP kosong hanya untuk local development).
# Untuk database development BARU/KOSONG saja, jalankan push secara eksplisit:
ALLOW_SCHEMA_PUSH=yes APPLY_SCHEMA=1 bash scripts/setup.sh
npm run dev
```

Untuk production build lokal setelah konfigurasi database tersedia:

```bash
npm run build
npm start
```

`db:push` adalah bootstrap/dev convenience dan **bukan** pengganti migration SQL yang ditinjau/berversi untuk production. Jangan jalankan terhadap database produksi sebelum migration workflow dan backup/restore disetujui.

## Deployment Vercel

1. Import repository ke Vercel dan pilih Next.js preset. Gunakan Node.js 22.x.
2. Tambahkan `DATABASE_URL` sebagai Environment Variable **server-only** untuk Preview dan Production, masing-masing diarahkan ke database terpisah.
3. Isi Environment Variables Vercel: `DATABASE_URL`, `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM`, dan `NEXT_PUBLIC_APP_URL`. Preview dan Production harus memakai database terpisah.
4. Jalankan `vercel link` dari direktori proyek (login melalui browser/CLI).
5. Jalankan `npm run vercel:cron-secret` untuk membuat `CRON_SECRET` acak dan memasukkannya langsung ke Environment Variables Production sebagai secret.
6. Deploy Preview, periksa health check dan jalankan manual testing dengan data uji. Jangan arahkan Preview ke database produksi.
7. Konfigurasi saat ini memakai jadwal harian `0 20 * * *` (UTC; sekitar 03:00–03:59 WIB) agar lolos batas Vercel Hobby. Worker endpoint tetap Bearer-protected. Jika memakai Vercel Pro/Enterprise dan membutuhkan polling 5 menit, ubah ke `*/5 * * * *` sebelum deploy.

Untuk aplikasi komersial, jangan gunakan Vercel Hobby; plan Hobby dibatasi untuk penggunaan personal/non-komersial. Lihat aturan paket dan cron terbaru pada dokumentasi Vercel.

## Tes

Tes integrasi mengharuskan aplikasi berjalan dalam **local development (`npm run dev`)** dan PostgreSQL khusus pengujian sudah berjalan, karena hanya local development yang boleh mengembalikan OTP development ke test runner:

```bash
BASE_URL=http://localhost:3000 npm run test:e2e
```

Jalankan juga pemeriksaan statis:

```bash
npm run typecheck
npm run lint
npm run build
```

Hasil `107/107` pada arsip asal adalah hasil historis Claude Code tertanggal 9 November 2026, bukan hasil build/test dari revisi paket ini. Tes tambahan untuk kebijakan password dan manajemen sesi ditambahkan ke skrip E2E, tetapi belum bisa dijalankan di sandbox ini.

## Dokumentasi penting

- [`MANUAL-TESTING-GUIDE.md`](./MANUAL-TESTING-GUIDE.md) — checklist uji manual per modul dan template laporan bug.
- [`BLOCKERS.md`](./BLOCKERS.md) — daftar pekerjaan dan prasyarat yang masih tertahan.
- [`docs/DEPLOYMENT.md`](./docs/DEPLOYMENT.md) — setup deployment.
- [`docs/ENVIRONMENT.md`](./docs/ENVIRONMENT.md) — matriks environment variable yang benar-benar dibaca source.
- [`docs/AUTHENTICATION.md`](./docs/AUTHENTICATION.md) — rancangan dan status autentikasi.
- [`docs/PRODUCTION_READINESS.md`](./docs/PRODUCTION_READINESS.md) — keputusan kesiapan produksi.
- [`docs/SYSTEM_ARCHITECTURE.md`](./docs/SYSTEM_ARCHITECTURE.md) — arsitektur sistem.

## Pengamanan data

Gunakan database non-produksi selama testing. Jangan mengunggah `.env`, token, API key, session cookie, dump produksi, atau data pelanggan/karyawan ke repository/ZIP. Logout tidak menghapus data database.

## Vercel: penting untuk Root Directory

Project harus dideploy dari direktori yang langsung berisi `package.json`, `vercel.json`, `next.config.ts`, dan `src/app/`. Arsip distribusi root-flat menyediakan susunan ini. Jika project disimpan di subfolder `Business-Flow/` pada GitHub, atur **Vercel → Settings → Build and Deployment → Root Directory** menjadi `Business-Flow`, bukan root repository. Lihat `VERCEL_ROOT_FIX.md`.
