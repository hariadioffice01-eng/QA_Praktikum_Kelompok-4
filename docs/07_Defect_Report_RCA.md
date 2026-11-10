# 07 — Defect Report, Triage, dan Root Cause Analysis (P07)

Praktikum Quality Assurance · Aplikasi: **Lernova Web v0.1.0**
Kelompok: `Kelompok 4`
Anggota:
1. Muhammad Adzkia Madani / 23076012
2. Alim Kurniadi / 23076003
3. Muhammad Hariadi / 23076043
4. Zekia Rahmaddiano / 23076056

> Seluruh defect di bawah adalah **temuan aktual** dari eksekusi test case (`test-cases/05_Test_Cases.xlsx`). Tidak ada defect simulasi (seeded). Defect belum diperbaiki karena kode aplikasi milik pihak lain tidak diubah (P04 bagian 5), sehingga **belum ada retest** dan semua status masih **Open**.

## 1. Lingkungan Pengujian (berlaku untuk semua defect)

| Item | Keterangan |
|---|---|
| Aplikasi | Lernova Web 0.1.0 (arsip `Lernova_Web.zip`), `npm run dev`, `http://localhost:3000` |
| Backend | Project Firebase uji milik kelompok, rules rancangan dari `lib/firestore-schema.md` |
| Sistem | Windows 11 (build 22631), Google Chrome |
| Akun uji | siswa1 = `siswa@lernova.com`, siswa2 = `siswa2@lernova.test`, guru = `guru@lernova.com` |
| Data soal | 15 soal pada koleksi `pertanyaan` |
| Reproducibility | Tiap defect diamati pada eksekusi test case. Penyebabnya bersifat deterministik di kode, sehingga **diperkirakan selalu terjadi**. Ulangi sekali lagi sebelum merekam video agar tercatat "3/3" bila berhasil. |

## 2. Ringkasan Defect

| ID | Judul | TC | Risiko | Severity | Priority | Status |
|---|---|---|---|---|---|---|
| DEF-01 | `POST /api/test-results` menerima persentase 150 dan menyimpannya | TC-05 | R-02 | **High** | P1 | Open |
| DEF-02 | `GET /api/test-results` menampilkan hasil siswa lain tanpa verifikasi identitas | TC-08 | R-02 | **Critical** | P1 | Open |
| DEF-03 | Siswa dapat mengubah `role` miliknya menjadi `admin` dan membuka `/dashboard/admin` | TC-06 | R-01 | **Critical** | P1 | Open |
| DEF-04 | Endpoint `/api/debug/*` dapat dibuka tanpa login dan menampilkan data pengguna | TC-09 | R-03 | **High** | P2 | Open |
| DEF-05 | Saat skor seri hanya satu gaya yang ditampilkan sebagai dominan | TC-04 | R-05 | **High** | P2 | Open |
| DEF-06 | `/tes` untuk role guru hanya menampilkan layar loading, tanpa "Akses Dikunci" | TC-07 | R-01 | **Medium** | P3 | Open |
| DEF-07 | `npm run build` gagal karena impor `firebase-admin/auth` | PRE-01 | R-04 | **Critical** | P1 | Open |
| DEF-08 | Jumlah soal tidak konsisten: README 10, `mockData.ts` 8, data uji 15 | (dokumentasi) | — | **Low** | P3 | Open |

Rekap severity: Critical 3, High 3, Medium 1, Low 1. Test case yang tidak menghasilkan defect: TC-01, TC-02, TC-03, TC-10, TC-11, TC-12.

## 3. Dasar Penetapan Severity dan Priority

**Severity** (dampak) memakai panduan P04 bagian 4.4:
- **Critical**: akses data atau role tanpa otorisasi, atau aplikasi tidak dapat dibangun (DEF-02, DEF-03, DEF-07).
- **High**: hasil tes salah atau tidak valid, atau kebocoran yang lebih terbatas (DEF-01, DEF-04, DEF-05).
- **Medium**: perilaku tidak konsisten dengan requirement tetapi ada jalan keluar (DEF-06).
- **Low**: dokumentasi dan teks (DEF-08).

**Priority** (urutan perbaikan) mengikuti risiko P03 dan biaya perbaikan:
- **P1** (segera, sebelum pengujian ulang apa pun): DEF-07, DEF-02, DEF-03, DEF-01. DEF-01 dan DEF-02 berada di berkas yang sama, sehingga diperbaiki bersamaan.
- **P2** (sebelum rilis): DEF-04, DEF-05.
- **P3** (dijadwalkan): DEF-06, DEF-08.

**Mengapa severity dan priority bisa berbeda.** DEF-05 severity High (hasil tes memang salah menurut REQ-05), tetapi priority P2 karena aturan seri adalah *usulan kelompok* yang perlu dikonfirmasi pemilik produk (P03 REQ-05). DEF-04 severity High, bukan Critical, karena yang terbuka adalah data akun (bukan hasil tes siswa), akun yang diuji adalah akun demo, dan endpoint tidak dapat mengubah data. Jika aplikasi dipakai dengan pengguna nyata, severity DEF-04 perlu dinaikkan menjadi Critical.

## 4. Laporan Defect

### DEF-01 — `POST /api/test-results` menerima persentase di luar 0–100

| Kolom | Isi |
|---|---|
| Test case / AC | TC-05 · AC-09 · REQ-09 · R-02 |
| Precondition | Aplikasi berjalan. UID siswa1 diketahui. Jumlah hasil siswa1 sebelum uji dicatat. |
| Langkah reproduksi | 1) Hitung hasil siswa1 lewat `GET /api/test-results?userId=<UID siswa1>`. 2) Dari tab `localhost:3000`, kirim `POST /api/test-results` lewat Console browser dengan `percentages.visual = 150`, `auditori = 0`, `kinestetik = 0`, `dominantStyle = visual`. 3) Catat status HTTP. 4) Ulangi langkah 1. |
| Expected | Status 400 dan tidak ada data baru. |
| Actual | Status **201**. Sistem menerima nilai di atas 100. Data tersimpan (hasil siswa1 bertambah, dan angka ini konsisten dengan TC-10 yang mulai dari 5 hasil). |
| Evidence | `Testcase-05.png` |
| Severity / Priority | High / P1 |
| Dampak | Data hasil palsu atau rusak dapat masuk ke statistik guru. |
| Reproducibility | Selalu (diperkirakan) |
| Status | Open |

### DEF-02 — Hasil tes siswa lain dapat dibaca tanpa verifikasi identitas

| Kolom | Isi |
|---|---|
| Test case / AC | TC-08 · AC-07 · REQ-07 · R-02 |
| Precondition | siswa2 sudah punya minimal 1 hasil tes. UID siswa2 diketahui. Pengguna login sebagai siswa1 di browser. |
| Langkah reproduksi | 1) Buka `GET /api/test-results?userId=<UID siswa2>`. 2) Catat status dan isi respons. |
| Expected | Status 401/403 dan tidak ada data siswa2. |
| Actual | Data tes siswa2 tampil lengkap untuk pemanggil lain. |
| Evidence | `Testcase-08.png` |
| Severity / Priority | **Critical** / P1 |
| Dampak | Hasil tes tiap siswa dapat dibaca siapa saja yang mengetahui `userId`. Berpotensi pelanggaran privasi data siswa. |
| Catatan | Rute `GET` pada `app/api/test-results/route.ts` hanya membaca parameter `userId` dan tidak memeriksa token (hasil tinjauan kode di P03). |
| Reproducibility | Selalu (diperkirakan) |
| Status | Open |

### DEF-03 — Siswa dapat menaikkan role menjadi admin

| Kolom | Isi |
|---|---|
| Test case / AC | TC-06 · AC-03 · REQ-03, REQ-07 · R-01 |
| Precondition | Akun siswa1 ada dengan role `siswa`. Rules rancangan dari `lib/firestore-schema.md` terpasang di project uji. |
| Langkah reproduksi | 1) Login siswa1 lewat Console browser (Blok 1). 2) Ubah field `role` di `users/{UID}` menjadi `admin` lewat Console (Blok 3). 3) Catat status HTTP. 4) Baca `users/{UID}` lagi. 5) Login siswa1 di browser dan buka `/dashboard/admin`. |
| Expected | Perubahan ditolak (403), role tetap `siswa`, `/dashboard/admin` tidak dapat dibuka. |
| Actual | Status **200**. Perubahan role diterima dan `/dashboard/admin` dapat diakses. |
| Evidence | `Testcase-06.png` |
| Severity / Priority | **Critical** / P1 |
| Dampak | Seluruh pembatasan role tidak berarti. Siswa dapat memperoleh hak admin. |
| Batasan | Hanya berlaku untuk rules rancangan di project uji kelompok. Rules produksi milik pemilik aplikasi tidak diuji (P04 bagian 5). |
| Reproducibility | Selalu (diperkirakan) |
| Status | Open |

### DEF-04 — Endpoint debug terbuka tanpa login

| Kolom | Isi |
|---|---|
| Test case / AC | TC-09 · AC-08 · REQ-08 · R-03 |
| Precondition | Tidak ada sesi login. |
| Langkah reproduksi | 1) Buka `GET /api/debug/check-demo-accounts`. 2) Buka `GET /api/debug/check-firebase`. 3) Catat status dan isi respons. |
| Expected | Status 401/403/404 dan tidak ada data pengguna. |
| Actual | Halaman berhasil tampil. Data pengguna dan jumlah pengguna terlihat. Rincian field yang terlihat ada pada screenshot. |
| Evidence | `Testcase-09-1.png`, `Testcase-09-2.png` |
| Severity / Priority | High / P2 (alasan di bagian 3) |
| Dampak | Informasi akun dan konfigurasi dapat dipakai untuk menebak akun demo. |
| Reproducibility | Selalu (diperkirakan) |
| Status | Open |

### DEF-05 — Aturan seri: hanya satu gaya dominan yang ditampilkan

| Kolom | Isi |
|---|---|
| Test case / AC | TC-04 · AC-06 · REQ-05 · R-05 |
| Precondition | Login siswa1. 15 soal tersedia. |
| Langkah reproduksi | 1) Buka `/tes`. 2) Jawab 6 soal Visual, 6 soal Auditori, 3 soal Kinestetik (Lampiran A di `docs/06`). 3) Kirim. 4) Lihat `/hasil`. |
| Expected | Persentase 40% / 40% / 20%. Visual **dan** Auditori sama-sama tampil sebagai gaya dominan. |
| Actual | Hasil akhir menampilkan siswa sebagai tipe **Auditori** saja. |
| Evidence | `Testcase-04.png` |
| Severity / Priority | High / P2 |
| Dampak | Label gaya belajar bias ke kategori yang berada belakangan. Rekomendasi belajar tidak lengkap. |
| Catatan | Hasil lama pada akun yang sama (33,3% / 33,3% / 33,3% tersimpan sebagai "kinestetik", dan 40% / 40% / 20% sebagai "auditori") konsisten dengan pola ini. Ini data pendukung, bukan pengganti test case. |
| Reproducibility | Selalu (diperkirakan) |
| Status | Open |

### DEF-06 — `/tes` untuk guru hanya menampilkan layar loading

| Kolom | Isi |
|---|---|
| Test case / AC | TC-07 · AC-03 · REQ-03 · R-01 |
| Precondition | Login sebagai `guru@lernova.com`. |
| Langkah reproduksi | 1) Ketik `/tes` di address bar. 2) Amati tampilan. |
| Expected | Halaman "Akses Dikunci" dengan tombol kembali. |
| Actual | Hanya layar loading, tanpa pesan dan tanpa tombol kembali. Dari hasil uji tidak teramati bahwa soal ditampilkan. |
| Evidence | `Testcase-07.png` |
| Severity / Priority | Medium / P3 |
| Dampak | Pengguna tertahan tanpa petunjuk. Akses ke soal tidak teramati terbuka. |
| Catatan | Permintaan `/api/seed` pada tab Network tidak diperiksa saat eksekusi, sehingga bagian kedua AC-03 belum terbukti. |
| Reproducibility | Selalu (diperkirakan) |
| Status | Open |

### DEF-07 — Build produksi gagal

| Kolom | Isi |
|---|---|
| Test case / AC | PRE-01 (pra-uji) · R-04 · QG-1 kriteria 1 |
| Precondition | Dependensi terpasang dengan `npm ci`. |
| Langkah reproduksi | 1) Jalankan `npm run build`. 2) Catat keluaran terminal. |
| Expected | Build lulus. |
| Actual | Gagal pada tahap type check: `./app/api/test-results/route.ts:12:25 Type error: Cannot find module 'firebase-admin/auth'`, exit code 1 (tercatat di P03). |
| Evidence | `evidence/PRE-01_build_log.txt` **(lampirkan log asli sebelum mengumpulkan)** |
| Severity / Priority | **Critical** / P1 |
| Dampak | Aplikasi tidak dapat dibangun dan dirilis ke Firebase. `npm run dev` masih berjalan. |
| Reproducibility | Selalu (terjadi pada tiap build) |
| Status | Open |

### DEF-08 — Jumlah soal tidak konsisten antar sumber

| Kolom | Isi |
|---|---|
| Test case / AC | Temuan dokumentasi dari tinjauan P03 (A3) dan P04 |
| Langkah reproduksi | 1) Baca README (menyebut 10 soal). 2) Hitung soal di `lib/mockData.ts`. 3) Hitung item di `GET /api/seed` pada project uji. |
| Expected | Satu angka yang konsisten. |
| Actual | README 10, `mockData.ts` 8, data uji 15. |
| Evidence | Respons `GET /api/seed`; salin potongan `mockData.ts` dan README ke `evidence/` bila perlu. |
| Severity / Priority | Low / P3 |
| Dampak | Pengembang dan penguji bisa memakai angka berbeda untuk acceptance criteria. |
| Status | Open |

## 5. Triage

| ID | Tindakan triage | Pemilik tindakan |
|---|---|---|
| DEF-07 | Perbaiki lebih dulu. Hapus impor `getAuth` yang tidak dipakai, atau pasang `firebase-admin` bila verifikasi token akan dibuat. | Pengembang aplikasi |
| DEF-02, DEF-01 | Perbaiki bersamaan di `route.ts`: verifikasi ID token Firebase di server, abaikan `userId` dari klien, validasi rentang dan jumlah persentase. | Pengembang |
| DEF-03 | Perbaiki rules: larang perubahan field `role` oleh pemilik dokumen, atau pindahkan role ke custom claims. | Pengembang / pemilik Firebase |
| DEF-04 | Hapus atau lindungi endpoint debug dan halaman penyiapan akun demo di lingkungan produksi. | Pengembang |
| DEF-05 | Konfirmasi aturan seri ke pemilik produk, lalu ubah perbandingan maksimum agar menangani seri. | Pemilik produk, lalu pengembang |
| DEF-06 | Tampilkan pesan "Akses Dikunci" untuk role selain siswa dan hentikan loading. | Pengembang |
| DEF-08 | Samakan jumlah soal di README, `mockData.ts`, dan data seed. | Pengembang / dokumentasi |

## 6. Analisis Akar Masalah (mini 5-Why)

Analisis ini adalah **dugaan penyebab** berdasarkan tinjauan kode dan dokumen P03, bukan hasil wawancara pengembang.

**DEF-01 dan DEF-02 (API tanpa validasi dan tanpa autentikasi)**

1. Mengapa data 150% tersimpan dan hasil siswa lain terbaca? Karena rute `/api/test-results` hanya memeriksa kehadiran field dan tidak memeriksa siapa pemanggilnya.
2. Mengapa tidak memeriksa pemanggil? Karena rute memakai Firebase client SDK dan tidak memverifikasi ID token di server. Impor `firebase-admin/auth` ada tetapi tidak dipakai.
3. Mengapa verifikasi server tidak dibuat? Karena validasi hanya dibangun di sisi klien (halaman `/tes`) dengan asumsi klien jujur.
4. Mengapa asumsi itu tidak diuji? Tidak ada bukti bahwa kontrak API pernah ditinjau dengan checklist keamanan (kegiatan SL-2 di P02 adalah rencana kelompok QA, bukan proses pemilik aplikasi).
5. **Akar masalah (proses):** tidak ada requirement keamanan dan review desain API sebelum implementasi, sehingga kebutuhan autentikasi dan validasi server tidak pernah menjadi acceptance criteria.

**DEF-03 (eskalasi role)**

1. Mengapa siswa bisa mengubah role? Karena rules mengizinkan pemilik dokumen `users/{uid}` memperbarui seluruh field.
2. Mengapa seluruh field? Karena rules rancangan tidak membatasi field per operasi.
3. Mengapa tidak dibatasi? Karena pengecekan role dikerjakan di UI (klien), sehingga rules dianggap cukup.
4. Mengapa UI dianggap cukup? Karena tidak ada threat check pada desain data pengguna.
5. **Akar masalah (proses dan teknis):** role disimpan pada dokumen yang bisa ditulis penggunanya sendiri, tanpa review keamanan pada desain skema.

**DEF-05 (aturan seri)**

1. Mengapa hanya satu gaya tampil? Karena pencarian maksimum memakai perbandingan `a[1] > b[1] ? a : b` yang mengembalikan elemen belakangan saat seri.
2. Mengapa seri tidak ditangani? Karena kode hanya mencari satu pemenang.
3. Mengapa hanya satu pemenang? Karena tidak ada aturan tertulis tentang seri pada requirement awal (README).
4. Mengapa tidak ada aturan? Karena kasus seri tidak dibahas di review requirement.
5. **Akar masalah (proses):** requirement tidak menyebut kasus batas (seri) sehingga pengembang tidak menulis kode maupun tes untuknya.

**DEF-07 (build gagal)**

1. Mengapa build gagal? Karena modul `firebase-admin/auth` tidak ditemukan.
2. Mengapa tidak ada? Karena paket `firebase-admin` tidak tercantum di `package.json` dan lockfile.
3. Mengapa impornya ada? Karena sisa kode untuk verifikasi token yang tidak pernah selesai.
4. Mengapa lolos? Karena `npm run dev` tidak menjalankan type check penuh.
5. **Akar masalah (proses):** tidak ada bukti pemeriksaan `build` dan `lint` sebelum perubahan digabung (SL-3 di P02 adalah rencana kelompok QA, bukan proses pemilik aplikasi).

## 7. Retest

Belum ada perbaikan pada aplikasi, sehingga **belum ada retest**. Rencana retest setelah pemilik atau kelompok memperbaiki salinan: TC-05 untuk DEF-01, TC-08 untuk DEF-02, TC-06 untuk DEF-03, TC-09 untuk DEF-04, TC-04 untuk DEF-05, TC-07 untuk DEF-06, dan PRE-01 untuk DEF-07. Status diubah menjadi *Closed* bila Actual sesuai Expected, atau *Reopened* bila belum.

## 8. Efek Samping Pengujian pada Data Uji

- TC-05 menyimpan satu hasil tidak valid (persentase 150) pada akun siswa1. Hasil ini ikut terhitung pada jumlah hasil TC-10 (5 menjadi 6). Bersihkan atau catat data ini pada project uji.
- TC-06 mengubah role siswa1 menjadi `admin` (Actual TC-08 menyebut "siswa1 yang telah berubah rolenya"). Pulihkan role siswa1 menjadi `siswa` dan catat urutan eksekusi agar tidak mengacaukan TC-10 sampai TC-12.
