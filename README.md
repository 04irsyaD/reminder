# MSF - Manage Schedule Flow

MSF adalah aplikasi reminder berbasis web untuk mahasiswa dan karyawan. Aplikasi ini dirancang sebagai sistem reminder berbasis approval: reminder dikirim ke Telegram, lalu akan terus diulang sampai user menekan tombol selesai atau menunda reminder.

Project ini masih berada pada fase dokumentasi requirement. Belum ada implementasi aplikasi, setup framework, database migration, atau integrasi layanan eksternal.

## Status Project

Status saat ini:

- Requirement utama sudah didokumentasikan di `REQUIREMENTS.md`.
- Implementasi kode aplikasi belum dibuat.
- Setup Next.js belum dilakukan.
- Migration database belum dibuat.

## Planned Tech Stack

Stack yang direncanakan:

| Area | Teknologi |
| --- | --- |
| Frontend | Next.js App Router |
| Backend API | Next.js Route Handler |
| Bahasa | TypeScript |
| Hosting | Vercel |
| Database | Supabase PostgreSQL |
| Authentication | Supabase Auth |
| Scheduler | Upstash QStash |
| Notifikasi | Telegram Bot API |
| Validasi Input | Zod |
| Styling | Tailwind CSS |
| Timezone Default | Asia/Jakarta |

## MVP Features

Fitur MVP yang direncanakan:

- Login dan register dengan Supabase Auth.
- Dashboard daftar reminder.
- Form tambah reminder.
- Edit dan cancel reminder.
- Connect Telegram melalui bot.
- Webhook Telegram untuk command dan inline button.
- Scheduler untuk mengecek reminder pending.
- Pengiriman reminder ke Telegram.
- Tombol selesai dan tunda 15 menit.
- Log pengiriman reminder.
- Basic settings untuk timezone dan default repeat interval.

## Documentation

Dokumen utama requirement:

- [REQUIREMENTS.md](./REQUIREMENTS.md)

Gunakan dokumen tersebut sebagai acuan sebelum masuk ke fase setup project, database, API, frontend, dan integrasi Telegram.

## Development Roadmap

Tahapan development yang direncanakan:

1. Documentation
2. Project setup
3. Database schema dan RLS
4. Auth dan profile
5. Reminder CRUD
6. Telegram integration
7. Scheduler integration
8. Testing dan deployment

Detail lengkap tersedia di `REQUIREMENTS.md`.

## Security Notes

Saat implementasi dimulai:

- Jangan commit file `.env.local`.
- Jangan expose `SUPABASE_SERVICE_ROLE_KEY` ke frontend.
- Jangan expose `TELEGRAM_BOT_TOKEN` ke frontend.
- Simpan secret di `.env.local` untuk local development.
- Simpan secret production di Vercel Environment Variables.
- Aktifkan Row Level Security di Supabase.

## Current Definition of Done

Untuk fase dokumentasi awal, project dianggap siap direview jika:

- `REQUIREMENTS.md` tersedia dan lengkap.
- `README.md` tersedia sebagai ringkasan project.
- `.gitignore` tersedia untuk melindungi file lokal, build output, dependency, dan secret.
- Belum ada kode aplikasi yang dibuat sebelum requirement disetujui.
