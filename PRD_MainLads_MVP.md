# Product Requirements Document (PRD)
# MainLads — Platform Penghubung Olahragawan dan Komunitas Olahraga

**Versi:** 1.0 (MVP)
**Tanggal:** 18 Juli 2026
**Status:** Draft

---

## 1. Latar Belakang

Komunitas olahraga rekreasi seperti mini soccer saat ini umumnya mengorganisir event melalui grup WhatsApp atau media sosial. Cara ini punya banyak keterbatasan: informasi event tercecer, sulit melacak siapa yang sudah bayar, sulit menemukan event baru di luar grup yang sudah diikuti, dan tidak ada riwayat partisipasi yang tersimpan rapi.

MainLads hadir sebagai platform yang menjembatani **komunitas penyelenggara event olahraga** dengan **olahragawan/peserta** yang ingin ikut serta, dimulai dari fokus olahraga **mini soccer**.

## 2. Tujuan Produk

- Memudahkan komunitas membuat dan mengelola event olahraga secara terpusat.
- Memudahkan peserta menemukan dan bergabung ke event olahraga di sekitar mereka.
- Menyediakan sistem pendaftaran dan konfirmasi pembayaran yang jelas dan tertelusur (meski masih manual di tahap MVP).
- Memvalidasi model bisnis dan minat pasar sebelum berinvestasi pada fitur yang lebih kompleks (payment gateway, multi-sport, dsb).

## 3. Target Pengguna

| Peran | Deskripsi |
|---|---|
| **Penyelenggara (Komunitas)** | Individu/kelompok yang rutin mengadakan event mini soccer dan butuh alat untuk mengumpulkan peserta serta mengelola pendaftaran. |
| **Peserta (Olahragawan)** | Individu yang ingin ikut bermain mini soccer secara kasual, mencari event berdasarkan lokasi dan jadwal yang cocok. |

## 4. Ruang Lingkup MVP

### 4.1 Termasuk dalam MVP
- Registrasi & autentikasi pengguna (dua role: peserta dan penyelenggara)
- Pembuatan dan pengelolaan event oleh penyelenggara
- Pencarian dan filter event oleh peserta
- Pendaftaran ke event dengan sistem kuota
- Konfirmasi pembayaran manual (upload bukti transfer, approval oleh admin komunitas)
- Notifikasi dasar (status pendaftaran, pengingat H-1)
- Riwayat event (untuk peserta dan penyelenggara)

### 4.2 Tidak Termasuk dalam MVP (Fase Selanjutnya)
- Payment gateway otomatis (Midtrans, dsb)
- Dukungan multi-sport (badminton, basket, dll)
- Chat antar pengguna
- Rating & review
- Sistem badge/level/leaderboard
- Rekomendasi event berbasis histori
- Integrasi kalender eksternal

## 5. User Stories

### Penyelenggara
1. Sebagai penyelenggara, saya ingin membuat akun komunitas agar bisa mulai membuat event.
2. Sebagai penyelenggara, saya ingin membuat event baru dengan detail lengkap (lokasi, tanggal, kuota, biaya) agar peserta bisa menemukan dan bergabung.
3. Sebagai penyelenggara, saya ingin melihat daftar peserta yang mendaftar beserta status pembayarannya agar saya bisa mengelola kehadiran.
4. Sebagai penyelenggara, saya ingin mengonfirmasi atau menolak bukti transfer peserta agar status pendaftaran akurat.
5. Sebagai penyelenggara, saya ingin membatalkan atau mengubah event agar peserta mendapat informasi terbaru.

### Peserta
1. Sebagai peserta, saya ingin membuat akun agar bisa mendaftar ke event.
2. Sebagai peserta, saya ingin melihat daftar event mini soccer di kota saya agar bisa memilih yang sesuai jadwal.
3. Sebagai peserta, saya ingin melihat detail event (lokasi, biaya, sisa kuota) sebelum mendaftar.
4. Sebagai peserta, saya ingin mendaftar ke event dan mengunggah bukti transfer agar pendaftaran saya diproses.
5. Sebagai peserta, saya ingin menerima notifikasi saat pendaftaran saya dikonfirmasi atau saat event mendekat.
6. Sebagai peserta, saya ingin melihat riwayat event yang pernah saya ikuti.

## 6. Functional Requirements

### 6.1 Autentikasi & Profil
- FR-1.1: Sistem harus mendukung registrasi via email atau nomor HP.
- FR-1.2: Pengguna dapat memilih satu atau kedua role (peserta dan/atau penyelenggara).
- FR-1.3: Profil pengguna minimal berisi: nama, foto (opsional), kota domisili, nomor kontak.
- FR-1.4: Profil komunitas (untuk penyelenggara) berisi tambahan: nama komunitas, kota operasional, kontak admin.

### 6.2 Manajemen Event
- FR-2.1: Penyelenggara dapat membuat event dengan field: nama event, jenis olahraga (default mini soccer), tanggal & jam, lokasi (alamat + titik peta), jumlah kuota peserta, biaya per peserta, nomor rekening/e-wallet tujuan, deskripsi tambahan.
- FR-2.2: Penyelenggara dapat mengedit event selama belum ada peserta yang confirmed (atau dengan notifikasi ke peserta jika sudah ada).
- FR-2.3: Penyelenggara dapat membatalkan event; sistem mengirim notifikasi ke seluruh peserta terdaftar.
- FR-2.4: Sistem menampilkan status kuota real-time (contoh: "7/10 slot terisi").

### 6.3 Pencarian & Discovery
- FR-3.1: Peserta dapat melihat daftar event yang tersedia, diurutkan berdasarkan tanggal terdekat.
- FR-3.2: Peserta dapat memfilter event berdasarkan kota dan rentang tanggal.
- FR-3.3: Peserta dapat membuka halaman detail event untuk melihat seluruh informasi sebelum mendaftar.

### 6.4 Pendaftaran & Pembayaran
- FR-4.1: Peserta dapat mendaftar ke event selama kuota masih tersedia.
- FR-4.2: Jika kuota penuh, peserta dapat masuk ke daftar tunggu (waitlist).
- FR-4.3: Setelah mendaftar, peserta diarahkan untuk mengunggah bukti transfer.
- FR-4.4: Status pendaftaran memiliki tiga kondisi: *Menunggu Konfirmasi*, *Terkonfirmasi*, *Ditolak*.
- FR-4.5: Penyelenggara dapat mengubah status pendaftaran berdasarkan verifikasi bukti transfer.

### 6.5 Notifikasi
- FR-5.1: Sistem mengirim notifikasi ke peserta saat status pendaftaran berubah.
- FR-5.2: Sistem mengirim pengingat H-1 sebelum event kepada peserta yang berstatus *Terkonfirmasi*.
- FR-5.3: Sistem mengirim notifikasi ke seluruh peserta terdaftar jika event dibatalkan atau diubah jadwalnya.

### 6.6 Riwayat
- FR-6.1: Peserta dapat melihat daftar event yang pernah diikuti beserta statusnya.
- FR-6.2: Penyelenggara dapat melihat daftar event yang pernah dibuat beserta ringkasan jumlah peserta.

## 7. Non-Functional Requirements

- NFR-1: Aplikasi harus dapat diakses secara responsif dari perangkat mobile (mayoritas pengguna diperkirakan mengakses lewat HP).
- NFR-2: Waktu muat halaman daftar event tidak lebih dari 3 detik pada koneksi 4G normal.
- NFR-3: Data bukti transfer disimpan dengan akses terbatas (hanya penyelenggara terkait dan admin platform yang bisa melihat).
- NFR-4: Sistem harus mencegah race condition saat dua peserta mendaftar bersamaan di slot kuota terakhir.

## 8. Alur Pengguna Utama (User Flow)

**Alur Penyelenggara:**
Daftar/Login → Lengkapi Profil Komunitas → Buat Event → Event Tayang di Listing → Terima Pendaftaran → Verifikasi Bukti Transfer → Kelola Peserta → (Opsional) Batalkan/Ubah Event

**Alur Peserta:**
Daftar/Login → Lengkapi Profil → Cari Event (filter kota/tanggal) → Lihat Detail Event → Daftar → Upload Bukti Transfer → Menunggu Konfirmasi → Terkonfirmasi → Menerima Reminder H-1 → Ikut Event → Riwayat Tersimpan

## 9. Model Data (Kasar)

```
users
- id
- nama
- email / no_hp
- role (peserta / penyelenggara / keduanya)
- kota
- foto_url (opsional)

komunitas
- id
- user_id (FK ke penyelenggara)
- nama_komunitas
- kota_operasional
- kontak_admin

events
- id
- komunitas_id (FK)
- nama_event
- jenis_olahraga (default: mini_soccer)
- tanggal_waktu
- lokasi_alamat
- lokasi_koordinat
- kuota_total
- biaya_per_orang
- rekening_tujuan
- deskripsi
- status (aktif / dibatalkan)

registrations
- id
- event_id (FK)
- user_id (FK)
- status (pending / confirmed / rejected / waitlist)
- bukti_transfer_url
- created_at
- updated_at
```

## 10. Metrik Keberhasilan (Success Metrics)

- Jumlah komunitas aktif yang membuat minimal 1 event per bulan.
- Jumlah event yang berhasil terisi penuh kuotanya.
- Tingkat konversi dari "lihat detail event" ke "mendaftar".
- Rata-rata waktu konfirmasi pembayaran oleh penyelenggara.
- Jumlah peserta yang kembali mendaftar ke event lain (retensi).

## 11. Model Monetisasi

**Keputusan untuk MVP: MainLads versi awal 100% gratis.**
Tidak ada komisi transaksi, listing premium berbayar, maupun iklan yang diimplementasikan di MVP. Tujuannya adalah mempercepat adopsi komunitas dan peserta tanpa gesekan biaya, sekaligus menyederhanakan scope teknis di tahap paling awal. Seluruh mekanisme monetisasi di bawah ini disiapkan sebagai **rencana Fase 2+**, bukan bagian dari MVP.

**Rencana Fase 2+ — Komisi Transaksi**
- Setelah payment gateway terintegrasi (bukan manual transfer lagi), platform akan mengambil komisi dari setiap pendaftaran yang berhasil.
- Rekomendasi persentase awal: **3-5%**. Angka ini sengaja dijaga kecil di awal supaya tidak membebani komunitas yang baru terbiasa pakai platform; bisa disesuaikan naik setelah ada data retensi dan willingness-to-pay dari fase gratis.
- Perhitungan dan pemotongan komisi baru masuk akal dilakukan otomatis setelah ada payment gateway; selama pembayaran manual, potongan komisi tidak diimplementasikan sama sekali.

**Rencana Fase 2+ — Listing Premium & Iklan**
- **Listing premium**: format sederhana berupa badge "Unggulan" yang menaikkan posisi event di listing, dengan slot terbatas per minggu/kota (bukan sistem bidding otomatis di awal).
- **Iklan**: banner statis di halaman listing/detail event, dipesan manual ke admin platform (bukan sistem ad-serving otomatis), ditargetkan ke brand olahraga atau penyedia lapangan.
- Kedua fitur ini sengaja dibuat low-tech di awal implementasinya nanti, supaya tidak perlu sistem billing kompleks sebelum ada cukup traffic untuk membenarkan investasinya.

## 12. Verifikasi Komunitas

Untuk mencegah event palsu atau penyelenggara yang tidak kredibel, MVP akan menambahkan proses verifikasi komunitas sebelum sebuah komunitas dapat mempublikasikan event.

**Requirement tambahan:**
- FR-1.5: Komunitas baru yang mendaftar berstatus *Belum Terverifikasi* dan tidak dapat mempublikasikan event hingga disetujui admin platform.
- FR-1.6: Admin platform memiliki panel sederhana untuk meninjau dan menyetujui/menolak pendaftaran komunitas baru (bisa berupa proses manual di awal, misal review data komunitas + kontak yang diberikan).
- FR-1.7: Komunitas yang belum diverifikasi dapat menyiapkan draf event, namun event tersebut tidak akan muncul di listing publik sampai komunitas disetujui.

**Kriteria verifikasi (usulan awal, perlu difinalisasi):**
- Kontak admin komunitas valid dan dapat dihubungi.
- Bukti aktivitas komunitas (misal tautan grup/media sosial, riwayat event sebelumnya jika ada).

**Proses verifikasi:** dilakukan oleh **admin platform internal** (bukan pihak ketiga atau otomatis berbasis dokumen). Di tahap MVP, review dilakukan manual oleh tim MainLads sendiri terhadap setiap pendaftaran komunitas baru, mengingat volume pendaftaran awal masih kecil dan terkonsentrasi di satu kota.

## 13. Fokus Peluncuran: Kota Bandung

Peluncuran awal MainLads difokuskan di **Kota Bandung** untuk memudahkan validasi produk pada satu pasar sebelum ekspansi.

**Implikasi:**
- Filter kota pada MVP tetap dibuat generik (bukan di-hardcode ke Bandung saja) agar mudah diperluas ke kota lain di fase berikutnya, namun strategi akuisisi komunitas dan promosi awal difokuskan ke komunitas mini soccer di Bandung.
- Data lokasi event sebaiknya tetap menyertakan titik koordinat (bukan hanya nama kota) untuk mendukung fitur pencarian berbasis jarak di masa depan.
- **Target akuisisi awal**: sekitar **5-10 komunitas** mini soccer aktif di Bandung sebelum peluncuran publik. Jumlah ini dipilih agar cukup untuk memberi variasi event ke peserta pertama, namun tetap realistis untuk direkrut dan diverifikasi secara manual oleh admin platform.

## 14. Asumsi & Batasan

- Proses verifikasi pembayaran masih manual dan bergantung pada kejujuran serta kecepatan respons penyelenggara.
- Belum ada mekanisme refund otomatis; kebijakan pembatalan/refund perlu ditentukan sebagai kebijakan platform, bukan fitur teknis di MVP ini.
- MVP tidak memiliki mekanisme monetisasi apa pun (gratis penuh); tidak ada logika pemotongan komisi, listing premium, atau iklan yang perlu dibangun di tahap ini.
- Verifikasi komunitas dilakukan manual oleh admin platform internal, sehingga kecepatan onboarding komunitas baru bergantung pada kapasitas tim internal untuk meninjau pendaftaran.

## 15. Roadmap Setelah MVP

| Fase | Fitur |
|---|---|
| Fase 2 | Payment gateway otomatis (termasuk perhitungan komisi otomatis), rating & review |
| Fase 3 | Dukungan multi-sport, chat antar pengguna, ekspansi ke kota lain |
| Fase 4 | Leaderboard, badge, rekomendasi event personalisasi |

## 16. Kriteria Verifikasi Komunitas (Detail)

Untuk menjaga proses pendaftaran tetap ringan namun cukup untuk mencegah komunitas palsu/spam, syarat verifikasi yang perlu dilampirkan saat pendaftaran komunitas baru:

- Nama komunitas dan deskripsi singkat aktivitasnya.
- Nama dan nomor kontak admin komunitas yang aktif (disarankan WhatsApp, karena jadi kanal komunikasi utama di konteks ini).
- Tautan bukti aktivitas komunitas (grup WhatsApp, akun Instagram, atau media sosial lain yang menunjukkan komunitas benar-benar aktif).
- Kota operasional komunitas.
- Persetujuan terhadap pedoman komunitas platform (semacam kesepakatan ringan, bukan kontrak formal, yang mencakup poin di bagian 17 di bawah).

Review dilakukan manual oleh admin platform internal (lihat bagian 12). Karena skala awal masih kecil (target 5-10 komunitas), proses ini realistis dilakukan satu per satu tanpa perlu sistem approval otomatis.

## 17. Kebijakan Pembatalan & Refund

Karena aliran dana terjadi langsung antara peserta dan komunitas (transfer manual, bukan lewat platform), MainLads **tidak menangani dana secara teknis** dan karenanya tidak bisa menjamin refund secara langsung. Kebijakan yang diterapkan:

- **Tanggung jawab refund ada di komunitas**, bukan di platform. Ini perlu dinyatakan secara eksplisit dalam pedoman komunitas yang disetujui saat verifikasi (lihat bagian 16).
- Sebagai syarat menjaga status terverifikasi, komunitas wajib mengembalikan dana ke peserta yang berstatus *Terkonfirmasi* dalam waktu maksimal **3x24 jam** apabila event dibatalkan oleh komunitas.
- Sistem MainLads menangani sisi teknis pembatalan: mengubah status event menjadi *Dibatalkan* dan mengirim notifikasi otomatis ke seluruh peserta terdaftar (FR-2.3, FR-5.3), namun tidak memproses transaksi refund itu sendiri.
- Pelanggaran berulang terhadap kebijakan ini (misal komunitas tidak melakukan refund atau sering membatalkan event) menjadi dasar bagi admin platform untuk meninjau ulang atau mencabut status verifikasi komunitas tersebut.
- Kebijakan ini perlu dicantumkan secara jelas di halaman detail event dan saat proses pendaftaran peserta, supaya ekspektasi jelas sejak awal.

## 18. Pemicu Waktu Mulai Monetisasi (Fase 2)

Alih-alih menetapkan tanggal tetap, MainLads akan mulai mempertimbangkan monetisasi (komisi, listing premium, iklan — lihat bagian 11) berdasarkan pencapaian metrik berikut:

- **Jumlah komunitas aktif reguler** di Bandung mencapai sekitar **15-20 komunitas** (naik dari target awal 5-10 komunitas untuk peluncuran).
- **Retensi komunitas sehat**: lebih dari 50% komunitas yang pernah membuat event, membuat event keduanya dalam rentang 2 bulan.
- **Volume event stabil**: platform secara konsisten memiliki event baru setiap minggu tanpa perlu effort akuisisi manual yang intensif dari tim.
- **Feedback peserta positif**: tingkat konversi dari "lihat detail event" ke "mendaftar" berada di angka sehat (perlu baseline data dari MVP dulu untuk menentukan angka pastinya).

Estimasi kasar: pencapaian ini realistis terjadi **3-6 bulan setelah peluncuran publik**, namun keputusan final tetap berbasis pencapaian metrik di atas, bukan jadwal kalender semata.
