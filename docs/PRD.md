# PRD — Skopa v2 (Amisbudi Edition)

**Product:** Skopa — Marketplace freelance untuk mahasiswa
**Versi:** 2.0 (Laravel API + React + PostgreSQL + Docker)
**Author:** Dicky Permana
**Status:** Draft
**Tujuan proyek:** Rebuild Skopa dengan stack PT Amisbudi Digital Salawasna, sekaligus sarana belajar API-first development, containerization, dan Git workflow tim.

---

## 1. Latar Belakang

Skopa adalah platform yang mempertemukan mahasiswa penyedia jasa (worker) dengan mahasiswa/klien yang butuh jasa (client). Versi 1 dibangun dengan Next.js + Prisma. Versi 2 dibangun ulang dengan arsitektur terpisah backend-frontend agar sesuai stack produksi Amisbudi.

Perbedaan kunci v1 -> v2: monolith Next.js menjadi **Laravel REST API** (backend murni, hanya JSON) + **React SPA** (frontend murni, konsumsi API), berjalan di atas **Docker**, dengan database **PostgreSQL**.

---

## 2. Tujuan & Non-Tujuan

### Tujuan (In-Scope untuk v2 MVP)
- Autentikasi multi-role (client & worker) berbasis token (Sanctum).
- Worker dapat membuat gig (jasa) dengan paket bertingkat.
- Client dapat browse dan melihat detail gig.
- Client dapat memesan gig (order).
- Wallet sederhana + integrasi pembayaran Midtrans.
- Upload gambar gig via Cloudinary.

### Non-Tujuan (Ditunda, JANGAN dibangun di MVP)
- Chat realtime antar user.
- Sistem rating & review.
- Notifikasi email/WhatsApp.
- Dashboard analitik.
- Multi-bahasa.
- Fitur animasi kompleks (aurora/glassmorphism v1). Fokus fungsional dulu.

> Prinsip: MVP dulu jalan end-to-end, baru poles. Scope creep adalah musuh proyek solo.

---

## 3. User Roles

| Role | Deskripsi | Kemampuan utama |
|---|---|---|
| **Guest** | Belum login | Browse gig, lihat detail, register |
| **Client** | User yang butuh jasa | Order gig, isi wallet, bayar |
| **Worker** | User penyedia jasa | Buat/kelola gig, terima order |
| **Admin** (opsional fase akhir) | Pengelola | Moderasi gig & user |

> Catatan: satu akun bisa jadi client sekaligus worker (mengikuti desain v1: auto-wallet + aktivasi worker). Untuk MVP, mulai dari client dulu, worker diaktivasi belakangan.

---

## 4. Fitur & User Story

### 4.1 Autentikasi
- Sebagai guest, saya bisa register dengan email & password agar punya akun.
- Sebagai user, saya bisa login dan menerima token untuk akses API.
- Sebagai user, saya bisa logout (revoke token).
- Sebagai client, saya bisa mengaktifkan mode worker agar bisa jual jasa.

### 4.2 Gig (Jasa)
- Sebagai worker, saya bisa membuat gig (judul, deskripsi, kategori, gambar).
- Sebagai worker, saya bisa menetapkan 3 tier paket (Basic / Standard / Premium) dengan harga & benefit berbeda.
- Sebagai worker, saya bisa edit dan hapus gig milik saya.
- Sebagai guest/client, saya bisa melihat daftar gig dan detailnya.

### 4.3 Order
- Sebagai client, saya bisa memesan satu paket dari sebuah gig.
- Sebagai client, saya bisa melihat daftar order saya beserta statusnya.
- Sebagai worker, saya bisa melihat order yang masuk ke gig saya.
- Status order: `pending` -> `paid` -> `in_progress` -> `completed` / `cancelled`.

### 4.4 Wallet & Pembayaran
- Sebagai client, saya punya wallet dengan saldo.
- Sebagai client, saya bisa top-up saldo via Midtrans.
- Sebagai client, saat order, saldo terpotong sesuai harga paket.

### 4.5 Upload
- Sebagai worker, saya bisa upload gambar gig yang disimpan di Cloudinary.

---

## 5. Arsitektur Teknis

```
Browser (React SPA / Vite)
        |  HTTP + JSON, Bearer token
        v
Laravel REST API (Sanctum)
        |  Eloquent
        v
PostgreSQL
```

Semua komponen berjalan dalam container Docker:
- `backend`  : PHP-FPM + Laravel
- `web`      : Nginx (serve API) — atau php artisan serve untuk dev
- `db`       : PostgreSQL 16
- `frontend` : Node + Vite dev server

### Kontrak API (contoh)
Frontend & backend sepakat format response sejak awal:
```json
{
  "data": { },
  "message": "success"
}
```
Endpoint utama (draft):
- `POST /api/register`
- `POST /api/login`
- `POST /api/logout`
- `GET  /api/gigs`
- `GET  /api/gigs/{id}`
- `POST /api/gigs`            (worker)
- `PUT  /api/gigs/{id}`       (worker, owner only)
- `DELETE /api/gigs/{id}`     (worker, owner only)
- `POST /api/orders`          (client)
- `GET  /api/orders`          (list milik user)
- `POST /api/wallet/topup`    (Midtrans)

---

## 6. Model Data (Draft ERD)

- **users**: id, name, email, password, is_worker, created_at
- **wallets**: id, user_id (FK), balance
- **gigs**: id, user_id (FK worker), title, description, category, image_url, status
- **gig_packages**: id, gig_id (FK), tier (basic/standard/premium), price, description, delivery_days
- **orders**: id, gig_package_id (FK), client_id (FK users), status, total_price
- **transactions**: id, wallet_id (FK), type (topup/payment), amount, reference

> ERD detail dibuat di Tahap 2 (Database Design).

---

## 7. Milestone (Tahap Pengerjaan)

Setiap tahap = 1 branch + 1 Pull Request di GitHub.

| Tahap | Deliverable | Branch |
|---|---|---|
| 0 | Repo + PRD ini masuk repo | `docs/prd` |
| 1 | Docker: Laravel + Postgres + React jalan | `chore/docker-setup` |
| 2 | Migration + model + ERD | `feat/database` |
| 3 | Auth API (register/login/logout + Sanctum) | `feat/auth-api` |
| 4 | Gig CRUD API + Cloudinary | `feat/gig-api` |
| 5 | Frontend React: auth + list gig | `feat/frontend-auth-gigs` |
| 6 | Order + Wallet + Midtrans | `feat/order-payment` |
| 7 | Pest test + GitHub Actions CI | `chore/testing-ci` |

---

## 8. Definition of Done (MVP)

Proyek dianggap "jadi" jika:
- User bisa register, login, logout lewat React.
- Worker bisa bikin gig dengan gambar & 3 paket.
- Client bisa lihat gig, order, dan bayar via wallet.
- Semua jalan lewat `docker compose up`.
- Ada minimal test untuk auth & gig.
- CI hijau di setiap PR.

---

## 9. Risiko & Mitigasi

| Risiko | Mitigasi |
|---|---|
| RAM 8GB berat untuk full Docker | Batasi WSL2 ke 3GB via `.wslconfig`; jika tetap berat, Postgres saja di Docker |
| Scope creep (nambah fitur terus) | Patuhi Non-Tujuan di bagian 2 |
| Midtrans butuh akun & rumit | Kerjakan paling akhir (Tahap 6); wallet manual dulu |
| Bingung stack baru | Kerjakan bertahap per PR, jangan lompat |
