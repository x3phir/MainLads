# Technical Requirements Document (TRD)
# MainLads — MVP

**Versi:** 1.0
**Tanggal:** 18 Juli 2026
**Status:** Draft
**Referensi:** PRD_MainLads_MVP.md v1.0

---

## 1. Ringkasan Teknis

Dokumen ini menerjemahkan requirement di PRD menjadi keputusan teknis: stack, arsitektur, skema database, desain API, dan infrastruktur untuk MVP MainLads. Prinsip utama yang dipegang: **sederhana, murah untuk dijalankan, dan cepat untuk divalidasi** — mengingat MVP ini gratis (tidak ada revenue di awal) dan skala awal kecil (target 5-10 komunitas di Bandung).

## 2. Pilihan Tech Stack
-
| Layer | Pilihan | Alasan |
|---|---|---|
| Frontend Web | **Vue 3 (Composition API) + Vite** | Ringan, cepat dikembangkan, cocok untuk tim kecil |
| Backend | **Fastify (Node.js) + TypeScript** | Performa baik, overhead rendah, mudah scaling horizontal nanti |
| Database | **PostgreSQL** | Relasional cocok untuk data event/registrasi yang punya banyak relasi & butuh transaksi konsisten |
| ORM | **Prisma** | Migrasi schema rapi, type-safe query, mempercepat development |
| Autentikasi | **JWT (access + refresh token)** | Stateless, mudah diimplementasikan di Fastify, cukup untuk skala MVP |
| Penyimpanan File (bukti transfer) | **Object storage (S3-compatible, misal Cloudflare R2 atau Supabase Storage)** | Murah, tidak membebani server aplikasi, mudah scaling |
| Notifikasi | **Email (SMTP, misal Resend/Postmark) + WhatsApp API (opsional Fase 1.5)** | Email cukup untuk MVP; WhatsApp API bisa jadi upgrade cepat karena target pengguna Indonesia dominan pakai WhatsApp |
| Peta/Lokasi | **OpenStreetMap + Leaflet.js (gratis)** atau Google Maps API (berbayar setelah kuota gratis habis) | OSM cukup untuk MVP tanpa biaya tambahan; bisa upgrade ke Google Maps kalau butuh akurasi pencarian lebih baik |
| Hosting Backend | **Railway / Render (PaaS)** | Setup cepat, harga terjangkau untuk traffic kecil, tidak perlu maintain server manual di awal |
| Hosting Frontend | **Vercel / Netlify** | Deploy otomatis dari Git, gratis untuk traffic MVP |
| Monitoring dasar | **Sentry (error tracking) + hosting platform logs** | Cukup untuk MVP tanpa perlu observability stack kompleks |

**Catatan konsistensi tim:** stack backend (Fastify + PostgreSQL + Prisma) dan frontend (Vue 3) sengaja disamakan dengan stack yang sudah pernah dipakai di project RaporChain sebelumnya, supaya development bisa lebih cepat karena sudah ada familiarity.

## 3. Arsitektur Sistem

Untuk MVP, direkomendasikan **arsitektur monolith modular** — satu backend service yang dipecah rapi per domain (auth, event, registration, community, notification), bukan microservices. Alasan: kompleksitas microservices tidak sebanding dengan skala pengguna di tahap awal (5-10 komunitas, kemungkinan ratusan peserta), dan monolith jauh lebih cepat untuk dikembangkan dan di-debug oleh tim kecil.

```
┌─────────────────┐         ┌──────────────────────┐
│   Vue 3 SPA      │ ──────▶ │   Fastify API (REST)  │
│  (Vercel/Netlify)│ ◀────── │   (Railway/Render)     │
└─────────────────┘         └──────────┬────────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    ▼                   ▼                   ▼
            ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
            │  PostgreSQL   │   │ Object Storage │   │  Email/WA API │
            │  (Prisma ORM) │   │ (bukti transfer)│  │  (notifikasi) │
            └───────────────┘   └───────────────┘   └───────────────┘
```

Modul backend disarankan dipecah sebagai berikut (folder-level, bukan service terpisah):
- `modules/auth` — registrasi, login, refresh token
- `modules/users` — profil pengguna
- `modules/communities` — profil & verifikasi komunitas
- `modules/events` — CRUD event, kuota
- `modules/registrations` — pendaftaran, upload bukti transfer, perubahan status
- `modules/notifications` — pengiriman email/reminder
- `modules/admin` — panel verifikasi komunitas oleh admin platform

## 4. Skema Database (Detail)

Skema di bawah ini memperjelas model data yang sudah digambarkan kasar di PRD, lengkap dengan tipe data dan constraint.

```sql
-- users
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE,
  phone VARCHAR(20) UNIQUE,
  password_hash TEXT NOT NULL,
  city VARCHAR(100),
  photo_url TEXT,
  role VARCHAR(20) NOT NULL DEFAULT 'participant', -- 'participant' | 'organizer' | 'both'
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- communities
CREATE TABLE communities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  city VARCHAR(100) NOT NULL,
  admin_contact VARCHAR(50) NOT NULL,
  social_proof_url TEXT, -- link grup WA/IG sebagai bukti aktivitas
  verification_status VARCHAR(20) NOT NULL DEFAULT 'pending', -- 'pending' | 'verified' | 'rejected'
  verified_at TIMESTAMPTZ,
  verified_by UUID REFERENCES users(id), -- admin platform yang approve
  created_at TIMESTAMPTZ DEFAULT now()
);

-- events
CREATE TABLE events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  community_id UUID NOT NULL REFERENCES communities(id),
  name VARCHAR(255) NOT NULL,
  sport_type VARCHAR(50) NOT NULL DEFAULT 'mini_soccer',
  scheduled_at TIMESTAMPTZ NOT NULL,
  location_address TEXT NOT NULL,
  location_lat DECIMAL(9,6),
  location_lng DECIMAL(9,6),
  quota_total INT NOT NULL CHECK (quota_total > 0),
  price_per_person DECIMAL(10,2) NOT NULL DEFAULT 0,
  payment_account_info TEXT, -- no rekening/e-wallet tujuan
  description TEXT,
  status VARCHAR(20) NOT NULL DEFAULT 'active', -- 'active' | 'cancelled'
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- registrations
CREATE TABLE registrations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_id UUID NOT NULL REFERENCES events(id),
  user_id UUID NOT NULL REFERENCES users(id),
  status VARCHAR(20) NOT NULL DEFAULT 'pending', -- 'pending' | 'confirmed' | 'rejected' | 'waitlist'
  payment_proof_url TEXT,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(event_id, user_id) -- cegah dobel-daftar ke event yang sama
);

-- Index penting
CREATE INDEX idx_events_city_scheduled ON events(scheduled_at) WHERE status = 'active';
CREATE INDEX idx_registrations_event ON registrations(event_id);
CREATE INDEX idx_communities_status ON communities(verification_status);
```

**Catatan implementasi kuota (menjawab NFR-4 di PRD — race condition):**
Saat peserta mendaftar, gunakan transaksi database dengan row-level lock (`SELECT ... FOR UPDATE`) pada baris event, atau manfaatkan constraint di level aplikasi dengan menghitung ulang jumlah `registrations` berstatus `confirmed`/`pending` di dalam transaksi yang sama sebelum insert baru. Ini mencegah dua peserta lolos mendaftar bersamaan di slot kuota terakhir.

## 5. Desain API (REST, Ringkasan Endpoint)

```
Auth
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/refresh
POST   /api/auth/logout

Users
GET    /api/users/me
PATCH  /api/users/me

Communities
POST   /api/communities              (buat profil komunitas, status default 'pending')
GET    /api/communities/:id
PATCH  /api/communities/:id

Events
POST   /api/events                   (hanya komunitas verified)
GET    /api/events                   (list + filter ?city=&date_from=&date_to=)
GET    /api/events/:id
PATCH  /api/events/:id
POST   /api/events/:id/cancel

Registrations
POST   /api/events/:id/register
POST   /api/registrations/:id/payment-proof   (upload bukti transfer)
PATCH  /api/registrations/:id/status          (approve/reject, hanya organizer terkait)
GET    /api/users/me/registrations            (riwayat peserta)
GET    /api/events/:id/registrations          (daftar peserta, hanya organizer terkait)

Admin (internal)
GET    /api/admin/communities?status=pending
PATCH  /api/admin/communities/:id/verify
PATCH  /api/admin/communities/:id/reject
```

Semua endpoint selain register/login/list-events publik memerlukan JWT di header `Authorization: Bearer <token>`. Endpoint `admin/*` memerlukan role tambahan `platform_admin` yang disimpan terpisah dari role user biasa (bukan role yang bisa dipilih user saat registrasi).

## 6. Autentikasi & Otorisasi

- **Skema token**: access token (umur pendek, ~15 menit) + refresh token (umur lebih panjang, ~7 hari), disimpan di HttpOnly cookie untuk mencegah XSS mengakses token langsung.
- **Role-based access control** sederhana di level middleware Fastify: cek role user (`participant`/`organizer`/`both`) dan kepemilikan resource (misal organizer hanya bisa edit event miliknya sendiri, dicek lewat `community_id` yang terhubung ke `user_id`).
- **Admin platform internal** (untuk verifikasi komunitas) dibuat sebagai role terpisah yang di-seed manual ke database, bukan lewat flow registrasi publik — mengingat jumlah admin masih sangat kecil di tahap ini.

## 7. Penanganan File (Bukti Transfer)

- Upload langsung dari client ke object storage menggunakan **pre-signed URL** (client minta URL upload ke backend, backend generate pre-signed URL, client upload langsung ke storage). Ini mengurangi beban bandwidth di server backend.
- Validasi tipe file (jpg/png/pdf) dan ukuran maksimal (misal 5MB) dilakukan di sisi backend sebelum pre-signed URL diberikan.
- Akses ke file bukti transfer dibatasi: hanya organizer terkait event dan admin platform yang bisa generate signed URL untuk melihat file (sesuai NFR-3 di PRD).

## 8. Notifikasi

Untuk MVP, notifikasi cukup lewat **email** (paling murah dan mudah diimplementasikan tanpa approval API pihak ketiga yang rumit):
- Trigger: perubahan status registrasi, event dibatalkan/diubah, reminder H-1.
- Reminder H-1 dijalankan lewat **scheduled job** (cron sederhana, bisa pakai `node-cron` di dalam service yang sama untuk MVP, tanpa perlu job queue terpisah dulu).
- Rekomendasi upgrade cepat pasca-MVP: integrasi WhatsApp Business API (misal lewat provider seperti Fonnte/Qontak) karena target pengguna Indonesia jauh lebih responsif ke WhatsApp dibanding email.

## 9. Keamanan

- Password di-hash dengan **bcrypt/argon2**, tidak pernah disimpan plain text.
- Rate limiting di endpoint sensitif (login, register, upload bukti transfer) untuk mencegah brute-force/spam — bisa pakai plugin `@fastify/rate-limit`.
- Validasi input ketat di setiap endpoint (misal pakai `zod` atau JSON schema Fastify) untuk mencegah injection dan data tidak valid.
- HTTPS wajib di semua environment (otomatis tersedia lewat Railway/Render/Vercel).
- CORS dikonfigurasi hanya mengizinkan origin frontend resmi.

## 10. Infrastruktur & Deployment

| Kebutuhan | Rekomendasi MVP |
|---|---|
| Backend hosting | Railway/Render (auto-deploy dari GitHub, scaling manual dulu) |
| Frontend hosting | Vercel/Netlify (auto-deploy dari GitHub) |
| Database | Managed PostgreSQL dari provider yang sama (Railway/Render/Supabase) |
| CI/CD | GitHub Actions: jalankan test + lint sebelum merge ke `main`, auto-deploy setelah merge |
| Environment | `development`, `staging`, `production` — minimal staging + production untuk MVP |
| Backup database | Backup otomatis harian dari provider managed Postgres |

Estimasi biaya bulanan di tahap awal (skala kecil): kemungkinan besar masih masuk *free tier* atau di bawah Rp500rb/bulan gabungan (hosting + storage + email), tergantung provider yang dipilih.

## 11. Testing Strategy

- **Unit test** untuk logic penting: perhitungan kuota, validasi status registrasi, role-based access.
- **Integration test** untuk endpoint kritikal: register event, join event, approve/reject pembayaran — terutama skenario race condition kuota.
- **Manual QA** untuk flow end-to-end sebelum setiap rilis, mengingat tim kecil dan belum perlu automated E2E test penuh di tahap MVP.

## 12. Batasan Teknis & Keputusan yang Sengaja Ditunda

Selaras dengan keputusan di PRD, hal-hal berikut sengaja **tidak** dibangun di MVP:
- Payment gateway integration (biaya integrasi + kompleksitas belum sepadan di skala awal).
- Sistem komisi otomatis (baru relevan begitu payment gateway masuk).
- Native mobile app (web responsive dianggap cukup untuk validasi awal).
- Real-time chat (butuh infrastruktur websocket/tambahan yang belum perlu di MVP).
- Sistem rekomendasi/personalisasi (butuh data historis yang belum ada di awal).

## 13. Roadmap Teknis Selaras dengan Roadmap Produk

| Fase Produk (dari PRD) | Perubahan Teknis Utama |
|---|---|
| Fase 2: Payment gateway + komisi otomatis | Integrasi Midtrans/Xendit, webhook payment, kalkulasi & pencatatan komisi otomatis di database |
| Fase 3: Multi-sport, chat, ekspansi kota | Generalisasi model `events` untuk sport type lain, tambah modul chat (kemungkinan pakai layanan pihak ketiga seperti Stream/Pusher daripada bangun sendiri) |
| Fase 4: Leaderboard, badge, rekomendasi | Tambah tabel agregasi statistik pengguna, kemungkinan mulai butuh data pipeline sederhana untuk rekomendasi |
