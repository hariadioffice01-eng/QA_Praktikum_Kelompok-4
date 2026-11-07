# 01 — Software Quality Assessment (P01)

Praktikum Quality Assurance · Aplikasi: **Lernova Web v0.1.0**
Kelompok: `Kelompok 4` · 
Anggota: `1.Muhammad adzkia Madani /23076012
          2.Alim Kurniadi / 23076003
          3.Muhammad Hariadi / 23076043
          4.Zekia Rahmaddiano / 23076056`. 
> Semua potensi masalah di bawah berstatus **hipotesis dari tinjauan kode** dan akan diverifikasi lewat eksekusi pada tahap 5-7.

## 1. Identitas Aplikasi

| Item | Keterangan |
|---|---|
| Nama | Lernova Web |
| Versi | 0.1.0 (`package.json`) |
| Stack | Next.js 16.2.6, React 19.2.4, TypeScript, Tailwind CSS 4, Firebase 12 (Auth + Firestore) |
| Lingkungan uji | Lokal (`npm run dev`, http://localhost:3000), data dummy |
| Jenis aplikasi | Web |

## 2. Tujuan Sistem dan Pengguna

**Tujuan.** Lernova membantu mengidentifikasi gaya belajar siswa (Visual, Auditori, Kinestetik) lewat tes 10 soal pilihan A/B/C. Hasilnya berupa persentase tiap gaya, gaya dominan, dan rekomendasi metode belajar, sehingga guru dapat menyesuaikan pengajaran.

| Pengguna | Kebutuhan utama | Fitur terkait |
|---|---|---|
| Siswa | Mengerjakan tes, melihat hasil dan riwayat | `/tes`, `/hasil`, `/dashboard/siswa`, riwayat |
| Guru | Memantau hasil siswa dan analisis kelas | `/dashboard/guru`, hasil-siswa, analisis |
| Admin | Mengelola pengguna, kelas, dan bank pertanyaan | `/dashboard/admin/*` |

**Alur utama yang diuji:** Login siswa → buka tes → jawab 10 soal → submit → hasil dihitung dan disimpan → tampil di `/hasil`.

## 3. Atribut Kualitas yang Dipilih

Atribut dipilih berdasarkan ISO/IEC 25010 dan relevansinya pada alur utama.

| No | Atribut | Alasan dipilih | Contoh indikator terukur |
|---|---|---|---|
| 1 | **Functional correctness** | Hasil tes adalah nilai inti produk. Salah hitung berarti rekomendasi belajar salah. | Persentase dan gaya dominan sesuai perhitungan manual oracle (100% kasus uji) |
| 2 | **Security** | Data siswa dan integritas hasil tes harus terlindungi. | Tidak ada endpoint yang menerima data tanpa otorisasi |
| 3 | **Reliability** | Hasil tes tidak boleh hilang atau tersimpan ganda. | Hasil tetap tersedia setelah refresh. Submit ulang tidak membuat duplikat. |
| 4 | **Usability** | Pengguna utama adalah siswa. Pesan galat harus jelas. | Pengguna menyelesaikan tes tanpa bantuan. Pesan galat dapat dipahami. |
| 5 | **Maintainability** | Banyak kode debug, contoh, dan dokumen sisa di repo. | Tidak ada halaman/endpoint debug di build produksi |

## 4. Potensi Masalah Kualitas dan Dampaknya

| ID | Potensi masalah (hipotesis) | Atribut | Dampak | Bukti kode |
|---|---|---|---|---|
| PM-01 | **Tie-break tidak adil.** Jika dua gaya memiliki skor sama (mis. 4-4-2), gaya dominan selalu jatuh ke kategori yang urutannya lebih akhir, karena perbandingan `a[1] > b[1] ? a : b` mengembalikan `b` saat seri. | Functional correctness | Siswa mendapat label gaya belajar yang bias. Rekomendasi guru kurang tepat. | `app/tes/page.tsx` (perhitungan `dominantStyle`) |
| PM-02 | **Endpoint simpan hasil tanpa otorisasi dan validasi server.** `userId`, `counts`, dan `percentages` dipercaya dari klien. Nilai di luar 0-100 atau `userId` pengguna lain kemungkinan diterima. | Security, Functional correctness | Data hasil dapat dipalsukan atau ditulis atas nama siswa lain. Statistik guru menjadi tidak valid. | `app/api/test-results/route.ts` |
| PM-03 | **`/api/seed` (GET) terbuka tanpa login dan memuat kunci kategori jawaban.** | Security | Siswa dapat mengetahui pemetaan jawaban ke gaya belajar lalu memanipulasi hasil. | `app/api/seed/route.ts` |
| PM-04 | **Hasil tes bertumpu pada `sessionStorage`** lalu dihapus setelah dibaca. | Reliability | Refresh atau buka tab baru di `/hasil` dapat menampilkan hasil kosong. | `app/hasil/page.tsx` |
| PM-05 | **Endpoint dan halaman debug** (`/api/debug/*`, `/debug-console`, `/quick-setup`) ikut dalam aplikasi. | Security, Maintainability | Permukaan serangan membesar. Informasi konfigurasi dapat bocor. | `app/api/debug/*`, `app/debug-console`, `app/quick-setup` |
| PM-06 | **Validasi registrasi hanya di klien** (password minimal 6 karakter, konfirmasi sama). | Security, Usability | Validasi dapat dilewati lewat permintaan langsung ke Firebase. | `components/auth/RegisterForm.tsx` |

Catatan: PM-01 sampai PM-03 diperkirakan paling berdampak pada alur utama dan menjadi kandidat risiko prioritas di P03.

## 5. Perbedaan QA dan QC pada Lernova

| Aspek | Quality Assurance (QA) | Quality Control (QC) |
|---|---|---|
| Fokus | Proses: mencegah cacat | Produk: mendeteksi cacat |
| Orientasi | Proaktif | Reaktif |
| Contoh pada Lernova | Mereview requirement "gaya dominan saat skor seri" sebelum implementasi. Menyusun quality gate dan checklist code review untuk endpoint API. | Menjalankan 12+ test case, membandingkan hasil aktual vs ekspektasi, mencatat defect PM-01 sampai PM-06 |
| Artefak | Strategi uji, kriteria entry/exit, RTM | Test case ter-eksekusi, laporan defect, evidence |
| Pelaku | Seluruh tim, terutama QA engineer | Penguji/QA engineer |

Contoh konkret: meninjau aturan tie-break pada tahap requirement adalah **aktivitas QA**. Menjalankan tes dengan jawaban 4-4-2 dan membandingkan hasil dengan oracle adalah **aktivitas QC**.

## 6. Cost of Quality

Kategori: **Cost of Conformance** (pencegahan dan penilaian) dan **Cost of Non-Conformance** (kegagalan internal dan eksternal).

| Kategori | Contoh pada Lernova | Perkiraan dampak |
|---|---|---|
| Prevention | Menetapkan aturan tie-break di requirement, review kontrak API | Biaya kecil di awal |
| Appraisal | Eksekusi test case, review kode API | Waktu penguji |
| Internal failure | Defect PM-01 ditemukan saat uji lalu diperbaiki sebelum rilis | Biaya perbaikan dan retest |
| **External failure** | Siswa menerima gaya belajar yang keliru, atau data hasil dipalsukan setelah dipakai kelas | Hilang kepercayaan guru, data kelas harus dihitung ulang |

**Cost of quality yang diidentifikasi: external failure akibat hasil tes yang salah atau dapat dipalsukan.** Jika ditemukan setelah dipakai satu kelas, biaya meliputi pengumpulan ulang hasil tes seluruh siswa, perbaikan data di Firestore, dan hilangnya kepercayaan pengguna. Biaya ini jauh lebih besar daripada biaya pencegahan di awal.

## 7. Kesimpulan Tahap 1

- Aplikasi memenuhi kriteria objek praktikum (alur utama, input valid/invalid, aturan bisnis, state).
- Lima atribut kualitas dipilih, dengan *functional correctness*, *security*, dan *reliability* sebagai prioritas.
- Enam potensi masalah teridentifikasi dan akan dibuktikan atau dibantah lewat eksekusi uji.
- Keterlacakan: PM-01 sampai PM-06 menjadi masukan untuk daftar risiko di P03 (`docs/03_Requirement_Risk_Register.md`).
