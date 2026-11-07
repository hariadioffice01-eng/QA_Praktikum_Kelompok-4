# 03 — Requirement Review dan Risk Analysis (P03)

Praktikum Quality Assurance · Aplikasi: **Lernova Web v0.1.0**
Kelompok: `Kelompok 4`
Anggota:
1. Muhammad Adzkia Madani / 23076012
2. Alim Kurniadi / 23076003
3. Muhammad Hariadi / 23076043
4. Zekia Rahmaddiano / 23076056

> Sumber analisis: README.md proyek, kode sumber (`app/`, `components/`, `hooks/`, `lib/`), rancangan security rules di `lib/firestore-schema.md`, serta potensi masalah PM-01 sampai PM-06 dari P01. Temuan dari kode ditandai dengan lokasi berkas. Lernova Web adalah aplikasi milik pihak lain yang diuji atas izin pemiliknya. Pengujian memakai project Firebase uji milik kelompok dengan rules rancangan dari `lib/firestore-schema.md`, sehingga risiko yang bergantung pada rules (R-01, R-06, R-07) dinilai terhadap rules tersebut, bukan terhadap rules produksi milik pemilik aplikasi. Penilaian akan diverifikasi saat eksekusi (P05–P06).

---

## 1. Dokumentasi Requirement dan Perbaikan Bagian Ambigu

### 1.1 Pernyataan awal yang ambigu (dari README)

| No | Pernyataan awal | Masalah | Dirujuk oleh |
|---|---|---|---|
| A1 | "User authentication (Email/Password via Firebase)" | Tidak ada aturan password, format email, maupun perilaku saat gagal. | REQ-01, REQ-02 |
| A2 | "Role-based access (Admin, Guru, Siswa)" | Tidak jelas halaman mana yang dibatasi dan apa yang terjadi saat akses ditolak. | REQ-03, REQ-07 |
| A3 | "Interactive assessment (10 questions)" | Tidak jelas apakah semua soal wajib dijawab, dan apakah urutan opsi tetap atau acak. Jumlah soal juga tidak konsisten: README menyebut 10, sedangkan data awal di `lib/mockData.ts` memuat **8** soal. Kelompok memutuskan menguji dengan **8 soal** sesuai data aplikasi, dan selisih dengan README dicatat sebagai temuan dokumentasi. | REQ-04 |
| A4 | "Results with visualization" | Tidak ada rumus persentase, aturan gaya dominan, dan aturan jika nilai seri. | REQ-05, REQ-06 |

### 1.2 Requirement setelah diperbaiki (testable)

| ID | Requirement | Perbaikan dari bagian ambigu | Bukti di kode |
|---|---|---|---|
| **REQ-01** | Pengguna dapat mendaftar dengan nama lengkap, email, password, konfirmasi password, dan role (`siswa` atau `guru`). Pendaftaran **ditolak** jika ada kolom kosong, password dan konfirmasi berbeda, atau password kurang dari 6 karakter. Pendaftaran berhasil membuat akun Firebase Auth dan dokumen `users/{uid}`. | A1: batas password dan kondisi penolakan dibuat eksplisit. | `components/auth/RegisterForm.tsx`, `hooks/useAuth.ts` |
| **REQ-02** | Pengguna terdaftar dapat login dengan email dan password yang benar, lalu diarahkan ke `/dashboard/{role}`. Kredensial salah menampilkan pesan error dan pengguna tetap di halaman login. | A1: hasil sukses dan gagal dibuat terukur. | `components/auth/LoginForm.tsx`, `app/dashboard/page.tsx` |
| **REQ-03** | Halaman tes (`/tes`) hanya dapat digunakan pengguna ber-role `siswa`. Role lain atau pengguna yang belum login melihat pesan "Akses Dikunci" dan tombol kembali ke dashboard, tanpa soal dimuat. | A2: halaman yang dibatasi dan respons penolakan didefinisikan. | `app/tes/page.tsx` |
| **REQ-04** | Tes menampilkan seluruh soal dari koleksi `pertanyaan` (**8 soal** sesuai data awal `lib/mockData.ts`; README menyebut 10 sehingga selisihnya dicatat sebagai temuan dokumentasi), masing-masing 3 opsi (A/B/C) dengan urutan opsi diacak setiap soal dimuat. Pengiriman jawaban **hanya diizinkan jika semua soal sudah dijawab**. | A3: kewajiban menjawab dan pengacakan opsi dinyatakan eksplisit. | `app/tes/page.tsx`, `app/api/seed/route.ts` |
| **REQ-05** | Setiap jawaban dipetakan ke satu kategori (visual/auditori/kinestetik). Persentase tiap gaya = jumlah jawaban kategori ÷ total jawaban × 100. Gaya dominan adalah gaya dengan persentase tertinggi. **Jika dua gaya atau lebih seri tertinggi, sistem harus menampilkan semuanya sebagai gaya dominan** (aturan seri diusulkan; perlu konfirmasi pemilik produk). | A4: rumus dan aturan seri ditambahkan karena sebelumnya tidak terdefinisi. | `app/tes/page.tsx` (fungsi `handleSubmit`) |
| **REQ-06** | Hasil tes tersimpan di koleksi `hasilTes` dengan `userId`, tanggal, gaya utama, dan persentase tiap gaya. Halaman `/hasil` menampilkan persentase, gaya utama, dan rekomendasi belajar sesuai gaya. Riwayat diurutkan dari yang terbaru. | A4: data yang disimpan dan ditampilkan dirinci. | `app/api/test-results/route.ts`, `app/hasil/page.tsx`, `lib/learningRecommendations.ts` |
| **REQ-07** | Seorang siswa hanya dapat membaca dan menulis hasil tes **miliknya sendiri**. Endpoint penyimpanan dan pembacaan hasil harus memverifikasi identitas pemanggil, tidak hanya menerima `userId` dari klien. | A2: batas hak akses data dibuat eksplisit. | `app/api/test-results/route.ts`, `lib/firestore-schema.md` |
| **REQ-08** | Endpoint diagnostik (`/api/debug/*`) dan penyiapan akun demo tidak boleh dapat diakses publik pada lingkungan produksi. | A2: pemisahan fungsi pengembangan dan produksi dinyatakan. | `app/api/debug/*`, `app/quick-setup/page.tsx` |
| **REQ-09** | Server memvalidasi data hasil tes sebelum disimpan: setiap `counts` bernilai bilangan bulat ≥ 0, setiap persentase berada pada rentang 0–100, jumlah persentase = 100 (toleransi pembulatan), dan `dominantStyle` konsisten dengan nilai tertinggi. Data yang tidak memenuhi ditolak dengan status 400. | A4 dan SL-1/SL-2 pada P02: rentang 0–100 dan validasi sisi server dijadikan requirement. | `app/api/test-results/route.ts` (saat ini hanya memeriksa kehadiran field) |

---

## 2. Acceptance Criteria (Given–When–Then)

| ID | Requirement | Kriteria |
|---|---|---|
| **AC-01** | REQ-01 | **Given** halaman `/register` terbuka dan email belum terdaftar, **When** pengguna mengisi semua kolom valid dengan password 6 karakter dan konfirmasi sama lalu menekan daftar, **Then** akun dibuat, dokumen `users/{uid}` berisi role yang dipilih, dan pengguna mendapat tanda berhasil. |
| **AC-02** | REQ-02 | **Given** akun `siswa@lernova.com` ada, **When** pengguna login dengan password yang salah, **Then** pesan error tampil dan URL tetap `/login`. |
| **AC-03** | REQ-03 | **Given** pengguna login sebagai `guru`, **When** membuka `/tes`, **Then** halaman "Akses Dikunci" tampil dan permintaan soal ke `/api/seed` tidak dikirim. |
| **AC-04** | REQ-04 | **Given** siswa berada pada soal terakhir dengan 7 dari 8 soal terjawab, **When** menekan kirim, **Then** muncul peringatan soal belum lengkap dan tidak ada data tersimpan ke `hasilTes`. |
| **AC-05** | REQ-05 | **Given** siswa menjawab sehingga terkumpul 4 visual, 2 auditori, 2 kinestetik, **When** jawaban dikirim, **Then** persentase tampil 50% / 25% / 25% dan gaya utama adalah Visual. |
| **AC-06** | REQ-05 | **Given** jawaban menghasilkan 3 visual, 3 auditori, 2 kinestetik, **When** jawaban dikirim, **Then** Visual dan Auditori sama-sama ditampilkan sebagai gaya dominan (seri). |
| **AC-07** | REQ-07 | **Given** siswa A login, **When** siswa A memanggil `GET /api/test-results?userId=<id siswa B>`, **Then** respons ditolak (401/403) dan tidak berisi data siswa B. |
| **AC-08** | REQ-08 | **Given** aplikasi berjalan dalam mode produksi tanpa login, **When** `GET /api/debug/check-demo-accounts` dipanggil, **Then** respons 401/403/404 dan tidak membocorkan email, role, atau UID. |
| **AC-09** | REQ-09 | **Given** siswa yang sah terautentikasi, **When** `POST /api/test-results` dikirim dengan `percentages.visual = 150` atau jumlah persentase ≠ 100, **Then** respons 400 dan tidak ada dokumen baru di `hasilTes`. |

---

## 3. Risk Register (Likelihood × Impact, skala 1–5)

Skala **Likelihood (L)**: 1 = sangat jarang … 5 = hampir pasti terjadi. Skala **Impact (I)**: 1 = dampak kecil … 5 = dampak kritis (data bocor, sistem tidak dapat dipakai). **Skor = L × I.** Tingkat: **Tinggi ≥ 15**, **Sedang 8–14**, **Rendah ≤ 7**.

| ID | Risiko kualitas | Atribut mutu | Req. | L | I | Skor | Tingkat | Dasar penilaian (bukti) |
|---|---|---|---|---|---|---|---|---|
| **R-01** | Eskalasi hak akses: siswa mengubah field `role` pada dokumen `users/{uid}` miliknya menjadi `admin`/`guru`. | Security | REQ-03, REQ-07 | 3 | 5 | **15** | Tinggi | Rancangan rules mengizinkan `update` oleh pemilik dokumen tanpa membatasi field (`lib/firestore-schema.md`). Pengecekan role di UI hanya dilakukan di sisi klien. Akan diverifikasi pada project uji dengan rules rancangan tersebut. |
| **R-02** | Endpoint `/api/test-results` tidak memeriksa autentikasi dan tidak memvalidasi isi data. `userId`, `counts`, dan `percentages` dipercaya dari klien sehingga hasil siswa lain dapat dibaca atau dipalsukan, dan nilai di luar 0–100 dapat tersimpan. | Security, Functional correctness | REQ-07, REQ-09 | 4 | 4 | **16** | Tinggi | `app/api/test-results/route.ts`: tidak ada verifikasi token dan hanya ada pengecekan kehadiran field. |
| **R-03** | Endpoint debug publik membocorkan jumlah pengguna, email akun demo, role, dan UID. Akun demo (`siswa@/guru@/admin@lernova.com`) dapat ditebak. | Security | REQ-08 | 4 | 3 | **12** | Sedang | `app/api/debug/check-firebase/route.ts` dan `check-demo-accounts/route.ts` tanpa autentikasi. |
| **R-04** | Kegagalan build/deploy: `app/api/test-results/route.ts` mengimpor `firebase-admin/auth`, tetapi paket `firebase-admin` tidak ada di `package-lock.json`. | Reliability / Maintainability | semua | 5 | 5 | **25** | Tinggi | **Terbukti.** `npm run build` (Next.js 16.2.6, Turbopack) gagal pada tahap type check: `./app/api/test-results/route.ts:12:25 Type error: Cannot find module 'firebase-admin/auth'`, exit code 1. Lockfile juga tidak memuat `firebase-admin`. Mode `npm run dev` masih dapat berjalan, tetapi build produksi/deploy tidak dapat dibuat. Likelihood dinaikkan dari 4 ke 5 karena kegagalan terjadi pada setiap build. |
| **R-05** | Gaya dominan salah saat nilai seri: pencarian maksimum memakai `a[1] > b[1] ? a : b`, sehingga saat seri gaya yang muncul belakangan dipilih tanpa pemberitahuan. | Functional correctness | REQ-05 | 4 | 3 | **12** | Sedang | `app/tes/page.tsx`. Dengan 8 soal dan 3 kategori, kombinasi seri (mis. 3-3-2) wajar terjadi. |
| **R-06** | Bank soal dan kunci kategori terekspos/dapat diubah: `/api/seed` mengirim kategori tiap opsi ke klien tanpa autentikasi, dan rules mengizinkan pengguna login mana pun menulis koleksi `pertanyaan`. | Security / Integrity | REQ-04, REQ-07 | 3 | 4 | **12** | Sedang | `app/api/seed/route.ts`; komentar rules "Only admin can write" tidak ditegakkan (`allow write: if request.auth != null`). |
| **R-07** | Ketidakkonsistenan rules dan kode: rules `hasilTes` memakai `siswaId`, sedangkan kode menyimpan dan meng-query `userId`. Jika rules seperti rancangan di-deploy, pembacaan hasil di `/hasil` dapat ditolak (halaman tampil kosong) atau rules tidak efektif. Query `where` + `orderBy` pada `/hasil` juga membutuhkan composite index Firestore. | Reliability / Security | REQ-06, REQ-07 | 3 | 4 | **12** | Sedang | Perbandingan `lib/firestore-schema.md` dengan `app/api/test-results/route.ts` dan `app/hasil/page.tsx`. |
| **R-08** | Kebocoran data pribadi lewat log: `useAuth` mencetak email, role, dan objek pengguna ke `console.log`. | Security / Privacy | REQ-01, REQ-02 | 3 | 2 | **6** | Rendah | `hooks/useAuth.ts` (`[useAuth] User authenticated:`, `[signUp] ...`). |
| **R-09** | Jawaban hilang saat pengiriman gagal: jawaban hanya ada di state komponen dan pengguna hanya diberi `alert`. Jika halaman dimuat ulang, seluruh tes harus diulang. Efek `useEffect` pada `/hasil` juga bergantung pada `selectedResult` dan berpotensi membaca Firestore berulang. | Reliability / Usability | REQ-04, REQ-06 | 3 | 3 | **9** | Sedang | `app/tes/page.tsx` (`answers` hanya di `useState`); `app/hasil/page.tsx` (dependensi `useEffect`). |
| **R-10** | Validasi registrasi hanya di klien: konfirmasi password dan pilihan role dikontrol UI, sehingga permintaan langsung ke Firebase dapat melewatinya. Panjang password minimal 6 kemungkinan tetap ditegakkan oleh Firebase Auth (perlu dibuktikan). | Security / Usability | REQ-01 | 2 | 2 | **4** | Rendah | `components/auth/RegisterForm.tsx`, `hooks/useAuth.ts`. Dugaan PM-06 pada P01. |

### 3.1 Matriks Risiko (Likelihood × Impact)

Setiap sel memuat ID risiko dari tabel di atas. 🟥 = Tinggi (skor ≥ 15), 🟧 = Sedang (8–14), 🟩 = Rendah (≤ 7).

| L \ I | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| **5** | | | | | 🟥 R-04 (25) |
| **4** | | | 🟧 R-03 (12), 🟧 R-05 (12) | 🟥 R-02 (16) | |
| **3** | | 🟩 R-08 (6) | 🟧 R-09 (9) | 🟧 R-06 (12), 🟧 R-07 (12) | 🟥 R-01 (15) |
| **2** | | 🟩 R-10 (4) | | | |
| **1** | | | | | |

Tiga sel merah (R-04, R-02, R-01) menjadi risiko prioritas pada bagian 4.

---

## 4. Tiga Risiko Prioritas, Alasan, dan Residual Risk

| Peringkat | Risiko | Alasan pemilihan | Mitigasi yang direncanakan | Residual risk |
|---|---|---|---|---|
| **1** | **R-04** — kegagalan build karena `firebase-admin` | Skor tertinggi (25) dan **sudah terbukti**: build produksi gagal saat ini. Tanpa build, aplikasi tidak dapat dirilis ke Firebase, sehingga semua requirement terdampak. | Hapus impor `getAuth` dari `firebase-admin/auth` yang tidak dipakai, atau pasang `firebase-admin` bila verifikasi token memang akan diimplementasikan (terkait R-02). Ulangi `npm run build` dan `npm run lint` sebagai smoke test (QG-1 kriteria 1). | **Rendah** jika build lulus dan dijalankan ulang pada setiap perubahan. Masih ada risiko dependensi lain yang tidak sinkron. |
| **2** | **R-02** — endpoint hasil tes tanpa autentikasi | Skor 16, dan menyentuh data siswa (hasil tes) yang bersifat pribadi. Mudah dieksploitasi karena hanya perlu mengubah parameter `userId`. | Uji negatif AC-07 pada level API. Rekomendasi perbaikan: verifikasi ID token Firebase di server dan abaikan `userId` dari klien. | **Sedang** selama perbaikan belum diterapkan. Setelah diperbaiki, sisa risiko ada pada konfigurasi token dan rules. |
| **3** | **R-01** — eskalasi role lewat dokumen `users` | Skor 15 dengan dampak maksimum (5): satu siswa dapat memperoleh hak admin sehingga seluruh pembatasan role tidak berarti. | Uji negatif: ubah `role` melalui Firestore SDK dengan akun siswa. Rekomendasi: rules yang melarang perubahan field `role`, atau role diberikan lewat custom claims. | **Sedang** karena rules produksi milik pemilik aplikasi tidak diuji. Pengujian ini hanya membuktikan kondisi lingkungan uji kelompok. |

**Perbedaan dengan dugaan awal (P01).** P01 memperkirakan PM-01 sampai PM-03 sebagai kandidat prioritas. Setelah requirement dan kode ditinjau lebih rinci, peringkat berdasarkan skor berubah: R-04 dan R-01 muncul sebagai temuan baru dengan dampak lebih besar, sedangkan PM-01 (R-05) dan PM-03 (R-06) berada di skor 12. PM-02 tetap menjadi risiko prioritas (R-02). Dugaan PM-04 hanya **sebagian terbukti**: `/hasil` memang memakai `sessionStorage`, tetapi juga membaca Firestore sehingga refresh tidak otomatis menghasilkan halaman kosong. Risikonya bergeser ke R-07 (rules dan index).

**Catatan prioritas berikutnya.** R-05 (gaya dominan saat seri, skor 12) tidak masuk tiga besar menurut skor, tetapi tetap **wajib diuji** karena merupakan fungsi inti aplikasi. Risiko ini ditangani lewat teknik boundary value analysis dan decision table pada Tahap 6. R-03, R-06, dan R-07 tercakup dalam pengujian akses (decision table role × halaman) dan pengujian API.

---

## 5. Keterlacakan Awal (P01 ↔ Risiko ↔ Requirement ↔ Kriteria)

### 5.1 Potensi masalah P01 ke risiko P03

| Potensi masalah (P01) | Risiko (P03) | Status tinjauan |
|---|---|---|
| PM-01 tie-break | R-05 | Didukung kode (`app/tes/page.tsx`) |
| PM-02 endpoint simpan hasil | R-02 | Didukung kode |
| PM-03 `/api/seed` terbuka dan memuat kunci | R-06 | Didukung kode |
| PM-04 `sessionStorage` | R-09, R-07 | Sebagian: ada pembacaan Firestore di `/hasil` |
| PM-05 endpoint debug | R-03 | Didukung kode |
| PM-06 validasi di klien | R-10 | Sebagian: batas password mungkin tetap ditegakkan Firebase Auth |
| (baru di P03) | R-01, R-04, R-07, R-08 | Temuan tambahan dari tinjauan kode dan rules |

### 5.2 Requirement ke risiko dan acceptance criteria

| Requirement | Risiko terkait | Acceptance Criteria |
|---|---|---|
| REQ-01 | R-08, R-10 | AC-01 |
| REQ-02 | R-08 | AC-02 |
| REQ-03 | R-01 | AC-03 |
| REQ-04 | R-06, R-09 | AC-04 |
| REQ-05 | R-05 | AC-05, AC-06 |
| REQ-06 | R-07, R-09 | AC-05 |
| REQ-07 | R-01, R-02, R-06, R-07 | AC-07 |
| REQ-08 | R-03 | AC-08 |
| REQ-09 | R-02 | AC-09 |

## 6. Kesimpulan Tahap 3

- Terdapat 9 requirement yang sudah testable, 9 acceptance criteria (Given-When-Then), dan 10 risiko berskor L×I.
- Tiga risiko prioritas adalah R-04 (build gagal, skor 25, sudah terbukti lewat `npm run build`), R-02 (endpoint hasil tes tanpa autentikasi, 16), dan R-01 (eskalasi role, 15). R-05 tetap wajib diuji karena merupakan fungsi inti.
- Seluruh PM-01 sampai PM-06 dari P01 sudah dipetakan. Dua di antaranya (PM-04 dan PM-06) hanya sebagian terbukti dan akan dikonfirmasi lewat eksekusi.
- Keterlacakan: risiko prioritas menjadi dasar test level dan type pada P04 (`docs/04_Test_Strategy_Plan.md`). Tabel 5.2 menjadi dasar RTM lengkap pada P05 (`test-cases/05_RTM.csv`).
