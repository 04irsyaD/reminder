# MSF — Manage Schedule Flow Requirements

## 1. Overview

**MSF — Manage Schedule Flow** adalah aplikasi reminder berbasis web untuk mahasiswa dan karyawan. Aplikasi ini bukan hanya pengingat biasa, tetapi sistem reminder berbasis approval.

User membuat reminder dari dashboard web. Data reminder disimpan di database. Pada waktu yang sudah ditentukan, sistem mengirim pesan ke Telegram. Jika user belum menekan tombol **Selesai**, sistem akan mengirim ulang reminder sesuai interval pengulangan, default setiap 15 menit, sampai user menyelesaikan atau membatalkan reminder tersebut.

Timezone default aplikasi adalah `Asia/Jakarta`.

## 2. Goals

Tujuan utama aplikasi:

1. User bisa register, login, dan logout.
2. User bisa menghubungkan akun Telegram melalui bot.
3. User bisa membuat reminder dari dashboard web.
4. Reminder menyimpan judul, pesan, waktu kirim, kategori, prioritas, interval pengulangan, dan status.
5. Sistem mengirim reminder ke Telegram sesuai waktu.
6. Pesan Telegram memiliki tombol:
   - ✅ Selesai
   - ⏰ Tunda 15 menit
7. Jika user menekan **Selesai**, reminder berubah menjadi `done` dan tidak dikirim ulang.
8. Jika user belum approve, reminder dikirim ulang sesuai `repeat_minutes`, default 15 menit.
9. Jika user menekan **Tunda 15 menit**, reminder digeser 15 menit dari waktu sekarang.
10. Semua riwayat pengiriman reminder dicatat ke log.

## 3. Tech Stack

Stack yang direncanakan:

| Area | Teknologi |
| --- | --- |
| Frontend | Next.js App Router |
| Backend API | Next.js Route Handler |
| Bahasa | TypeScript |
| Hosting | Vercel |
| Database | Supabase PostgreSQL |
| Authentication | Supabase Auth |
| Scheduler / pemicu serverless | Upstash QStash |
| Notifikasi | Telegram Bot API |
| Validasi input | Zod |
| Styling | Tailwind CSS |
| Timezone default | Asia/Jakarta |

## 4. MVP Scope

Fitur yang masuk MVP:

1. Login/register dengan Supabase Auth.
2. Dashboard daftar reminder.
3. Form tambah reminder.
4. Edit reminder.
5. Cancel reminder.
6. Connect Telegram via command `/start`.
7. API webhook Telegram.
8. API check reminder pending.
9. Integrasi QStash untuk memanggil endpoint pengecekan reminder.
10. Integrasi Telegram Bot untuk mengirim reminder dengan tombol inline.
11. Tabel log pengiriman reminder.
12. Basic settings untuk timezone dan default repeat interval.

## 5. Out of Scope

Fitur yang tidak masuk MVP:

- WhatsApp API
- Google Calendar
- AI scheduler
- Mobile app
- Multi-user team
- Statistik kompleks
- Payment/subscription
- Push notification browser

## 6. User Role

Untuk MVP hanya ada satu role.

### User

User dapat:

- Register
- Login
- Logout
- Connect Telegram
- Membuat reminder
- Melihat reminder
- Mengedit reminder
- Membatalkan reminder
- Menekan tombol selesai dari Telegram
- Menunda reminder dari Telegram
- Melihat log pengiriman reminder

## 7. User Flow

Flow utama aplikasi:

1. User register/login.
2. User masuk ke dashboard.
3. User membuka settings.
4. User menghubungkan Telegram dengan mengetik `/start` di bot.
5. Sistem menyimpan `telegram_chat_id`.
6. User membuat reminder baru.
7. Reminder disimpan dengan status `pending`.
8. QStash memanggil endpoint `check-reminders` secara berkala.
9. Sistem mencari reminder yang sudah waktunya dikirim.
10. Sistem mengirim pesan ke Telegram.
11. User memilih salah satu aksi:
    - Klik ✅ Selesai
    - Klik ⏰ Tunda 15 menit
    - Tidak melakukan apa-apa
12. Jika selesai, status reminder menjadi `done`.
13. Jika tunda, `send_at` digeser 15 menit dari waktu sekarang.
14. Jika tidak ada aksi, reminder dikirim ulang sesuai `repeat_minutes`.

### Telegram Linking Flow

Untuk mencegah akun Telegram salah terhubung, proses linking sebaiknya memakai token unik.

1. User membuka halaman `/settings`.
2. Sistem membuat token linking sementara untuk user yang sedang login.
3. Sistem menampilkan instruksi untuk membuka bot Telegram dengan format `/start <linking_token>`.
4. Bot menerima command `/start`.
5. Sistem memvalidasi token linking.
6. Jika valid, sistem menyimpan `telegram_chat_id` dan `telegram_username` ke `profiles`.
7. Token linking dianggap selesai dan tidak boleh digunakan ulang.

## 8. Data Model

Bagian ini hanya dokumentasi rancangan tabel. Jangan membuat migration SQL pada tahap ini.

### Tabel `profiles`

| Field | Type | Keterangan |
| --- | --- | --- |
| `id` | uuid | Primary key, references `auth.users` |
| `email` | text | Email user |
| `name` | text nullable | Nama user |
| `telegram_chat_id` | text nullable | Chat ID Telegram |
| `telegram_username` | text nullable | Username Telegram |
| `timezone` | text | Default `Asia/Jakarta` |
| `default_repeat_minutes` | integer | Default `15` |
| `created_at` | timestamptz | Waktu dibuat |
| `updated_at` | timestamptz | Waktu diperbarui |

### Tabel `reminders`

| Field | Type | Keterangan |
| --- | --- | --- |
| `id` | uuid | Primary key |
| `user_id` | uuid | References `profiles.id` |
| `title` | text | Judul reminder |
| `message` | text | Isi pesan reminder |
| `category` | text | Kategori reminder |
| `priority` | text | Prioritas reminder |
| `send_at` | timestamptz | Waktu pengiriman |
| `status` | text | Status reminder |
| `repeat_minutes` | integer | Interval pengulangan dalam menit |
| `last_sent_at` | timestamptz nullable | Waktu terakhir dikirim |
| `approved_at` | timestamptz nullable | Waktu user menekan selesai |
| `cancelled_at` | timestamptz nullable | Waktu reminder dibatalkan |
| `created_at` | timestamptz | Waktu dibuat |
| `updated_at` | timestamptz | Waktu diperbarui |

Allowed status:

- `pending`
- `done`
- `cancelled`

Allowed priority:

- `low`
- `normal`
- `high`
- `urgent`

Default categories:

- `kuliah`
- `kerja`
- `pribadi`
- `lainnya`

### Tabel `reminder_logs`

| Field | Type | Keterangan |
| --- | --- | --- |
| `id` | uuid | Primary key |
| `reminder_id` | uuid | References `reminders.id` |
| `user_id` | uuid | References `profiles.id` |
| `channel` | text | Default `telegram` |
| `sent_at` | timestamptz | Waktu percobaan pengiriman |
| `status` | text | Status pengiriman |
| `error_message` | text nullable | Pesan error jika gagal |
| `telegram_message_id` | text nullable | ID pesan dari Telegram |
| `response_payload` | jsonb nullable | Response dari Telegram atau metadata lain |

Recommended indexes untuk tahap implementasi nanti:

- `profiles(id)`
- `profiles(telegram_chat_id)`
- `reminders(user_id)`
- `reminders(status)`
- `reminders(send_at)`
- `reminders(last_sent_at)`
- `reminders(user_id, status, send_at)`
- `reminder_logs(reminder_id)`
- `reminder_logs(user_id)`

## 9. API Requirements

Semua API harus menggunakan TypeScript, mengembalikan JSON konsisten, dan memvalidasi input menggunakan Zod jika menerima request body atau query parameter.

### Profile

#### `GET /api/profile`

Fungsi:

- Mengambil profil user login.
- Menampilkan status Telegram connected atau belum.

#### `PATCH /api/profile`

Fungsi:

- Update nama.
- Update timezone.
- Update default repeat interval.

### Reminder

#### `GET /api/reminders`

Fungsi:

- Mengambil daftar reminder user login.
- Mendukung filter status, kategori, dan tanggal.
- Tidak boleh mengembalikan reminder milik user lain.

Query parameter yang direkomendasikan:

- `status`
- `category`
- `date`
- `from`
- `to`

#### `POST /api/reminders`

Fungsi:

- Membuat reminder baru.
- Validasi input menggunakan Zod.
- Status awal `pending`.
- Menggunakan `repeat_minutes` dari request body atau default dari profile.

Request body contoh:

```json
{
  "title": "Kerjakan tugas kuliah",
  "message": "Jangan lupa kerjakan tugas Sistem Informasi.",
  "category": "kuliah",
  "priority": "high",
  "send_at": "2026-06-01T21:00:00+07:00",
  "repeat_minutes": 15
}
```

#### `GET /api/reminders/:id`

Fungsi:

- Mengambil detail reminder milik user login.
- Termasuk log pengiriman.
- Tidak boleh mengembalikan detail reminder milik user lain.

#### `PATCH /api/reminders/:id`

Fungsi:

- Mengubah `title`, `message`, `category`, `priority`, `send_at`, `repeat_minutes`, atau `status`.
- Tidak boleh mengubah reminder milik user lain.
- Perubahan status hanya boleh ke status valid.
- Jika status diubah menjadi `done`, set `approved_at`.
- Jika status diubah menjadi `cancelled`, set `cancelled_at`.

#### `DELETE /api/reminders/:id`

Fungsi:

- Tidak hard delete.
- Ubah status menjadi `cancelled`.
- Set `cancelled_at`.
- Tidak boleh membatalkan reminder milik user lain.

### Scheduler

#### `POST /api/check-reminders`

Fungsi:

- Dipanggil oleh Upstash QStash.
- Validasi signature QStash atau `CRON_SECRET`.
- Ambil reminder dengan status `pending`.
- Cek `send_at <= now()`.
- Cek `last_sent_at` kosong atau sudah melewati `repeat_minutes`.
- Kirim pesan ke Telegram.
- Update `last_sent_at`.
- Simpan log ke `reminder_logs`.

Catatan idempotency:

- Endpoint harus aman jika dipanggil lebih dari sekali dalam waktu berdekatan.
- Implementasi nanti perlu menghindari pengiriman dobel untuk reminder yang sama.
- Update `last_sent_at` dan pencatatan log harus dilakukan secara konsisten.

### Telegram

#### `POST /api/telegram/webhook`

Fungsi:

- Menerima command `/start`.
- Menerima callback tombol `done:REMINDER_ID`.
- Menerima callback tombol `snooze:REMINDER_ID`.
- Validasi secret webhook Telegram.

Logic `/start`:

- Ambil `chat_id`.
- Ambil `username` jika tersedia.
- Validasi proses linking user.
- Simpan `telegram_chat_id` dan `telegram_username` ke profil user jika proses linking valid.
- Balas bahwa Telegram berhasil terhubung.

Logic `done:REMINDER_ID`:

- Pastikan reminder milik user terkait.
- Update status menjadi `done`.
- Set `approved_at = now()`.
- Balas ke Telegram bahwa reminder selesai.

Logic `snooze:REMINDER_ID`:

- Pastikan reminder milik user terkait.
- Set `send_at = now() + interval tunda`.
- Set `last_sent_at = null`.
- Status tetap `pending`.
- Balas ke Telegram bahwa reminder ditunda.

## 10. Telegram Requirements

### Format Pesan

Format pesan Telegram:

```text
🔔 Reminder MSF

Judul: Kerjakan tugas kuliah
Kategori: Kuliah
Prioritas: Tinggi
Waktu: 21:00

Pesan:
Jangan lupa kerjakan tugas Sistem Informasi.

Status: Belum selesai
```

### Inline Keyboard

Inline keyboard:

```text
[✅ Selesai] [⏰ Tunda 15 Menit]
```

Callback data:

```text
done:<reminder_id>
snooze:<reminder_id>
```

### Telegram Behavior

- Pesan dikirim hanya ke `telegram_chat_id` milik user.
- Reminder tidak boleh dikirim jika user belum menghubungkan Telegram, kecuali sistem menyediakan status warning khusus.
- Jika Telegram API gagal, error harus dicatat ke `reminder_logs`.
- Tombol inline harus tetap memvalidasi kepemilikan reminder di server.

## 11. Page Requirements

### `/login`

Fitur:

- Login
- Register
- Redirect ke dashboard jika sudah login

### `/dashboard`

Fitur:

- Menampilkan reminder hari ini
- Menampilkan reminder pending
- Menampilkan reminder selesai
- Menampilkan status Telegram connected
- Tombol tambah reminder
- Filter status/kategori

### `/reminders/new`

Fitur:

- Form tambah reminder

Field:

- `title`
- `message`
- `category`
- `priority`
- `send_at`
- `repeat_minutes`

### `/reminders/[id]`

Fitur:

- Detail reminder
- Edit reminder
- Cancel reminder
- Lihat log pengiriman

### `/settings`

Fitur:

- Status koneksi Telegram
- Panduan connect bot
- Set timezone
- Set default repeat interval

## 12. Business Rules

Aturan bisnis:

1. Reminder hanya dikirim jika status `pending`.
2. Reminder tidak dikirim jika status `done` atau `cancelled`.
3. Reminder pertama dikirim jika `send_at <= now()`.
4. Reminder ulang dikirim jika `last_sent_at` sudah melewati `repeat_minutes`.
5. Klik **Selesai** menghentikan reminder.
6. Klik **Tunda 15 menit** menggeser `send_at` 15 menit dari waktu sekarang.
7. Jika Telegram belum terhubung, reminder tidak boleh diaktifkan atau harus diberi status warning.
8. Reminder yang gagal dikirim harus dicatat ke `reminder_logs`.
9. Delete reminder adalah soft delete dengan status `cancelled`.
10. Semua waktu disimpan sebagai `timestamptz`.
11. Tampilan waktu mengikuti timezone user, default `Asia/Jakarta`.
12. `repeat_minutes` default adalah `15`.

## 13. Security Requirements

Requirement keamanan:

1. Jangan expose `SUPABASE_SERVICE_ROLE_KEY` ke frontend.
2. Jangan expose `TELEGRAM_BOT_TOKEN` ke frontend.
3. Endpoint `/api/check-reminders` harus dilindungi validasi QStash signature atau `CRON_SECRET`.
4. Endpoint Telegram webhook harus divalidasi dengan secret.
5. Aktifkan Row Level Security di Supabase.
6. User hanya boleh membaca dan mengubah data miliknya sendiri.
7. Semua input user harus divalidasi dengan Zod.
8. Semua waktu disimpan sebagai `timestamptz`.
9. Gunakan timezone default `Asia/Jakarta`.
10. Jangan menyimpan token rahasia di repository.
11. Semua environment variable sensitif harus diletakkan di `.env.local` dan Vercel Environment Variables.
12. Callback Telegram harus memvalidasi ownership reminder sebelum mengubah data.
13. Endpoint API user harus memvalidasi session Supabase.

## 14. Environment Variables

Dokumentasikan file `.env.example` pada tahap implementasi nanti dengan isi berikut:

```env
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

TELEGRAM_BOT_TOKEN=
TELEGRAM_WEBHOOK_SECRET=

QSTASH_TOKEN=
QSTASH_CURRENT_SIGNING_KEY=
QSTASH_NEXT_SIGNING_KEY=

CRON_SECRET=
APP_URL=
```

### Frontend-safe Variables

Variable yang boleh digunakan di frontend:

- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`

### Server-only Variables

Variable yang hanya boleh digunakan di server:

- `SUPABASE_SERVICE_ROLE_KEY`
- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_WEBHOOK_SECRET`
- `QSTASH_TOKEN`
- `QSTASH_CURRENT_SIGNING_KEY`
- `QSTASH_NEXT_SIGNING_KEY`
- `CRON_SECRET`

### Shared Configuration

- `APP_URL` digunakan untuk URL aplikasi, callback, webhook, dan konfigurasi QStash.
- Di local development, value dapat mengarah ke URL tunnel seperti ngrok jika webhook Telegram perlu diuji.
- Di production, value harus mengarah ke domain Vercel.

## 15. Non-Functional Requirements

Requirement non-functional:

- Aplikasi harus aman untuk deployment di Vercel.
- API harus menggunakan TypeScript.
- Validasi input harus jelas.
- Error harus dikembalikan dalam format JSON konsisten.
- UI harus mobile-friendly.
- Sistem harus cukup ringan untuk pemakaian pribadi/prototype.
- MVP harus bisa berjalan dengan free tier Vercel, Supabase, QStash, dan Telegram Bot API.
- Sistem tidak perlu real-time per detik; toleransi keterlambatan mengikuti interval scheduler.
- Dokumentasi harus cukup jelas untuk menjadi acuan development bertahap.

### Format Error JSON

Format error yang direkomendasikan:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Input tidak valid",
    "details": {}
  }
}
```

## 16. Testing Checklist

### Auth

- [ ] User bisa register.
- [ ] User bisa login.
- [ ] User bisa logout.
- [ ] User tidak bisa akses dashboard tanpa login.

### Reminder

- [ ] User bisa membuat reminder.
- [ ] User bisa edit reminder.
- [ ] User bisa cancel reminder.
- [ ] User hanya bisa melihat reminder miliknya.
- [ ] Reminder pending muncul di dashboard.
- [ ] Reminder done tidak dikirim ulang.
- [ ] Reminder cancelled tidak dikirim ulang.

### Telegram

- [ ] Bot bisa menerima `/start`.
- [ ] Chat ID tersimpan.
- [ ] Reminder terkirim ke Telegram.
- [ ] Tombol Selesai mengubah status menjadi done.
- [ ] Tombol Tunda menggeser waktu reminder.
- [ ] Callback Telegram tidak bisa mengubah reminder milik user lain.

### Scheduler

- [ ] Endpoint `check-reminders` hanya bisa dipanggil dengan secret valid.
- [ ] Reminder dikirim saat waktunya tiba.
- [ ] Reminder dikirim ulang jika belum approve.
- [ ] Reminder berhenti setelah done.
- [ ] Reminder berhenti setelah cancelled.
- [ ] Log pengiriman tersimpan.
- [ ] Pengiriman gagal tetap tercatat ke log.

## 17. Development Roadmap

### Phase 1 — Documentation

- Buat `REQUIREMENTS.md`
- Review requirement
- Finalisasi scope MVP

### Phase 2 — Project Setup

- Setup Next.js
- Setup Tailwind
- Setup Supabase client
- Setup environment variables

### Phase 3 — Database

- Buat schema SQL
- Aktifkan RLS
- Buat policy dasar

### Phase 4 — Auth & Profile

- Login/register
- Profile page
- Telegram connection status

### Phase 5 — Reminder CRUD

- Dashboard
- Create reminder
- Edit reminder
- Cancel reminder

### Phase 6 — Telegram Integration

- Buat bot
- Set webhook
- Handle `/start`
- Kirim test message
- Handle inline button

### Phase 7 — Scheduler Integration

- Setup QStash
- Endpoint `check-reminders`
- Kirim reminder otomatis
- Log pengiriman

### Phase 8 — Testing & Deployment

- End-to-end testing
- Deploy ke Vercel
- Setup environment production
- Final bug fixing

## 18. Definition of Done

### Definition of Done untuk Dokumentasi Requirement

Task dokumentasi ini dianggap selesai jika:

1. File `REQUIREMENTS.md` berhasil dibuat di root project.
2. Isi file mencakup semua bagian requirement yang diminta.
3. Tidak ada file implementasi kode yang dibuat.
4. Tidak ada setup framework yang dilakukan.
5. Tidak ada migration SQL yang dibuat.
6. Dokumen siap direview sebelum masuk ke fase development.

### Definition of Done untuk MVP Aplikasi

MVP aplikasi dianggap selesai jika:

1. User bisa register, login, logout, dan mengakses dashboard secara aman.
2. User bisa menghubungkan akun Telegram melalui bot.
3. User bisa membuat, melihat, mengedit, dan membatalkan reminder.
4. Sistem bisa mengirim reminder ke Telegram sesuai waktu.
5. Sistem bisa mengirim ulang reminder jika belum diselesaikan.
6. Tombol ✅ Selesai mengubah status reminder menjadi `done`.
7. Tombol ⏰ Tunda 15 Menit menggeser waktu reminder.
8. Semua pengiriman dan kegagalan tercatat di `reminder_logs`.
9. Endpoint scheduler dan webhook terlindungi secret/signature.
10. RLS Supabase aktif dan user hanya dapat mengakses data miliknya sendiri.
