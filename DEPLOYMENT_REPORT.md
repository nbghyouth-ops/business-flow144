# DEPLOYMENT_REPORT — konfigurasi Vercel Hobby (10 November 2026)

- `vercel.json` kini menjadwalkan `/api/cron` dengan `0 20 * * *` (sekali per hari, UTC), sehingga tidak lagi meminta cron setiap lima menit pada akun Hobby.
- Waktu Hobby memiliki presisi per jam; jadwal 20:00 UTC dapat dieksekusi antara 20:00–20:59 UTC (sekitar 03:00–03:59 WIB hari berikutnya).
- `/api/cron` dan `/api/v1/worker/tick` memerlukan `Authorization: Bearer <CRON_SECRET>`; tanpa secret valid request ditolak.
- `npm run vercel:cron-secret` disediakan untuk membuat secret acak dan mengirimkannya via stdin ke Environment Variables Production sebagai sensitive value. Jalankan setelah `vercel link` dan login pemilik akun, kemudian redeploy.
- Pada Pro/Enterprise, ubah jadwal menjadi `*/5 * * * *` bila worker perlu polling tiap lima menit. Jadwal harian Hobby tidak setara dengan polling lima menit.
- Queue impor CSV tetap memicu `runJobs` setelah persetujuan impor; cron harian terutama untuk housekeeping dan drain job terbatas.
- Tidak ada akun Vercel yang dihubungkan, environment variable milik user yang ditambahkan, atau deployment yang dilakukan dari sesi ini.

**Status:** konfigurasi source untuk cron Hobby telah diperbaiki dan diverifikasi secara statis; deployment dan cron nyata belum diverifikasi. Vercel Hobby hanya sesuai penggunaan personal/non-komersial; gunakan paket hosting komersial yang sesuai untuk Business Flow sebagai layanan bisnis.
