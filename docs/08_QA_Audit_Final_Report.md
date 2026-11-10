# 08 — Final QA Audit dan Quality Decision (P08)

Praktikum Quality Assurance · Aplikasi: **Lernova Web v0.1.0**
Kelompok: `Kelompok 4`
Anggota:
1. Muhammad Adzkia Madani / 23076012
2. Alim Kurniadi / 23076003
3. Muhammad Hariadi / 23076043
4. Zekia Rahmaddiano / 23076056

## 1. Keputusan

> # **NO-GO**
> Lernova Web 0.1.0 **tidak layak dirilis** pada kondisi saat ini.

Alasan singkat (rinci pada bagian 5 dan 6):
1. Ada **3 defect Critical** yang masih open: DEF-02 (hasil siswa lain terbaca), DEF-03 (siswa bisa menjadi admin), DEF-07 (build gagal).
2. **Kriteria 5 dan 7 QG-1 tidak terpenuhi.** Menurut aturan keputusan P02 dan P04, ini otomatis No-Go.
3. Pass rate 6 dari 12 (50%), jauh di bawah ambang 90%. Test case prioritas tinggi lulus 3 dari 7 (43%), ambang 100%.
4. Hanya 2 dari 8 kriteria QG-1 yang terpenuhi.

Keputusan ini mengikuti hasil eksekusi nyata. Ia tidak berubah oleh catatan konsistensi di bagian 3, karena setiap catatan tersebut tidak mengubah status Fail pada test case yang menjadi dasar keputusan.

## 2. Ringkasan Eksekusi

| Hasil | Jumlah | Test case |
|---|---|---|
| Pass | 6 | TC-01, TC-02, TC-03, TC-10, TC-11, TC-12 |
| Fail | 6 | TC-04, TC-05, TC-06, TC-07, TC-08, TC-09 |
| Belum dieksekusi | 0 | — |
| Pra-uji PRE-01 (build) | — | Hasil dirujuk dari P03 (gagal). Log asli perlu dilampirkan. |

| Prioritas | Total | Pass | Fail |
|---|---|---|---|
| Tinggi | 7 (TC-02, 04, 05, 06, 08, 10, 11) | 3 (TC-02, 10, 11) | 4 (TC-04, 05, 06, 08) |
| Sedang | 5 (TC-01, 03, 07, 09, 12) | 3 (TC-01, 03, 12) | 2 (TC-07, 09) |

Per teknik black-box:

| Teknik | Test case | Pass | Fail |
|---|---|---|---|
| Boundary Value Analysis | TC-01, 02, 03 | 3 | 0 |
| Equivalence Partitioning | TC-04, 05, 06 | 0 | 3 |
| Decision Table | TC-07, 08, 09 | 0 | 3 |
| State Transition | TC-10, 11, 12 | 3 | 0 |

Pola yang terlihat: fitur dasar (registrasi dan alur tes) bekerja, sedangkan semua uji **kontrol akses dan validasi server** gagal.

## 3. Audit Konsistensi (RTM, Requirement, Risiko, Test Case, Evidence)

### 3.1 RTM yang diperbarui

| Requirement | Risiko | AC | Test case | Status | Defect | Evidence |
|---|---|---|---|---|---|---|
| REQ-01 Registrasi | R-08, R-10 | AC-01 | TC-01 Pass, TC-02 Pass, TC-03 Pass | Lulus | — | Testcase-01, 02, 03.png |
| REQ-02 Login | R-08 | AC-02 | TC-10 (langkah login valid) | Sebagian (AC-02 tidak diuji) | — | Testcase-10-1..3.png |
| REQ-03 Akses /tes | R-01 | AC-03 | TC-06 Fail, TC-07 Fail | **Gagal** | DEF-03, DEF-06 | Testcase-06, 07.png |
| REQ-04 Tes lengkap 15 soal | R-06, R-09 | AC-04 | TC-10 Pass, TC-11 Pass, TC-12 Pass | Lulus | — | Testcase-10-1..3, 11, 12.png |
| REQ-05 Persentase dan dominan | R-05 | AC-05, AC-06 | TC-04 Fail, TC-10 Pass | **Gagal** (AC-06) | DEF-05 | Testcase-04, 10-1..3.png |
| REQ-06 Hasil tersimpan dan tampil | R-07, R-09 | AC-05 | TC-10 Pass, TC-12 Pass | Lulus | — | Testcase-10-1..3, 12.png |
| REQ-07 Isolasi data siswa | R-01, R-02, R-06, R-07 | AC-07 | TC-06 Fail, TC-08 Fail | **Gagal** | DEF-02, DEF-03 | Testcase-06, 08.png |
| REQ-08 Endpoint debug tertutup | R-03 | AC-08 | TC-09 Fail | **Gagal** | DEF-04 | Testcase-09-1, 09-2.png |
| REQ-09 Validasi data di server | R-02 | AC-09 | TC-05 Fail | **Gagal** | DEF-01 | Testcase-05.png |
| (Pra-uji) Build dan lint | R-04 | — | PRE-01 | **Gagal** (dari P03) | DEF-07 | `evidence/PRE-01_build_log.txt` (belum terlampir) |

**Hasil audit RTM:** 9 dari 9 requirement terhubung ke minimal satu test case (QG-1 kriteria 6 terpenuhi). 5 requirement gagal, 3 lulus penuh, 1 lulus sebagian.

### 3.2 Catatan konsistensi yang perlu dirapikan sebelum pengumpulan

Berikut temuan audit pada berkas `05_Test_Cases.xlsx` dan `05_RTM.xlsx`. Semuanya adalah masalah **dokumentasi uji**, bukan temuan pada aplikasi, tetapi dosen dapat memeriksanya.

| No | Temuan | Perbaikan yang disarankan |
|---|---|---|
| 1 | **TC-04:** kolom *Data_Uji* masih "3-3-2" dan *Expected* masih "37,5% / 37,5% / 25%", padahal langkah, AC-06, dan Lampiran A memakai pola **6-6-3 (40% / 40% / 20%)**. | Ubah Data_Uji menjadi "6-6-3 (total 15 soal)" dan Expected menjadi "40% / 40% / 20%. Visual dan Auditori sama-sama dominan". Status Fail tetap sah. |
| 2 | **TC-06:** kolom *Expected* berisi "role baru: admin" (tampaknya tertimpa data uji). Expected asli adalah penolakan 403. | Pulihkan Expected: "Perubahan DITOLAK (403). Role tetap siswa. `/dashboard/admin` tidak dapat dibuka." Tanpa ini status Fail tidak punya patokan. |
| 3 | **TC-01 sampai TC-03:** Expected menyebut "Login REST", dan langkah 4 (cek akun lewat login) tidak dilaporkan pada Actual. | Hapus kata "REST" dari Expected, dan tambahkan hasil cek login pada Actual atau catat bahwa pengecekan dilakukan lewat UI saja. |
| 4 | **TC-08:** Precondition masih menyebut "Auth REST", dan Actual menyebut siswa1 "telah berubah rolenya". TC-06 mengubah akun yang dipakai test case lain. | Tulis urutan eksekusi, pastikan role siswa1 dipulihkan sebelum TC-10 sampai TC-12, dan catat pemulihan itu sebagai langkah. |
| 5 | **TC-10 dan TC-12:** Actual belum menyebut angka. TC-10 hanya "persentasi serupa" tanpa nilai 60% / 20% / 20%. TC-12 tidak menyebut jumlah hasil sebelum dan sesudah. | Tambahkan angka yang terlihat di screenshot (contoh TC-10: "60% / 20% / 20%, jumlah 5 menjadi 6"). |
| 6 | **RTM:** sheet `Table1` tidak memiliki kolom Defect, Evidence, dan Catatan. Sheet `05_RTM (1)` memiliki kolom Defect yang kosong dan kolom Evidence berisi pola `evidence/TC-xx`, padahal berkas sebenarnya bernama `Testcase-xx.png`. Tidak ada kolom status. | Gabungkan menjadi satu sheet, isi Defect dengan tabel 3.1, ganti nama evidence sesuai berkas sebenarnya, dan tambahkan kolom Status. |
| 7 | **PRE-01:** tidak ada log build yang dilampirkan di berkas yang diterima, dan README/RTM merujuk `evidence/PRE-01_build_log.txt`. | Jalankan ulang `npm run build` dan `npm run lint`, simpan log. Ini bukti QG-1 kriteria 1. |
| 8 | **Efek samping TC-05:** satu hasil tidak valid (150%) tersimpan pada siswa1 dan memengaruhi angka TC-10 (5 menjadi 6). | Bersihkan data uji, atau catat bahwa jumlah awal TC-10 sudah memuat data TC-05. |

## 4. Coverage dan Blind Spot

### 4.1 Coverage terhadap acceptance criteria

| AC | Diuji oleh | Hasil |
|---|---|---|
| AC-01 | TC-02 | Pass |
| AC-02 | — | **Tidak diuji** |
| AC-03 | TC-07 | Fail |
| AC-04 | TC-11 | Pass |
| AC-05 | TC-10 | Pass |
| AC-06 | TC-04 | Fail |
| AC-07 | TC-08 | Fail |
| AC-08 | TC-09 | Fail |
| AC-09 | TC-05 | Fail |

Coverage AC: **8 dari 9 (89%)** dieksekusi. Coverage requirement: **9 dari 9 (100%)** ada test case.

### 4.2 Coverage terhadap risiko

| Risiko | Skor | Diuji oleh | Keterangan |
|---|---|---|---|
| R-04 Build gagal | 25 | PRE-01 | Terbukti gagal (P03). Log perlu dilampirkan. |
| R-02 API tanpa auth/validasi | 16 | TC-05, TC-08 | **Terbukti** |
| R-01 Eskalasi role | 15 | TC-06, TC-07 | **Terbukti** (pada rules rancangan) |
| R-03 Endpoint debug | 12 | TC-09 | **Terbukti** |
| R-05 Gaya dominan saat seri | 12 | TC-04 | **Terbukti** |
| R-06 Bank soal terekspos | 12 | — | **Tidak diuji khusus** |
| R-07 Rules dan index tidak sinkron | 12 | TC-10 (tidak langsung) | Tidak teramati masalah, tetapi tidak diuji khusus |
| R-09 Jawaban hilang | 9 | TC-11, TC-12 | Tidak terbukti: jawaban tidak hilang saat offline |
| R-08 Log berisi data pribadi | 6 | — | **Tidak diuji** |
| R-10 Validasi registrasi di klien | 4 | TC-01 sampai TC-03 | Hanya lewat UI |

Risiko yang **terbukti**: 5 dari 10 (R-04, R-02, R-01, R-03, R-05). Risiko yang **tidak punya test case**: R-06 dan R-08.

### 4.3 Blind spot

1. **AC-02** (login dengan password salah) tidak diuji. Risiko rendah karena ditangani Firebase Auth.
2. **`/api/seed`** (R-06) tidak diuji dengan uji negatif, padahal termasuk dalam kriteria 5 QG-1. Fakta bahwa kriteria 5 sudah gagal lewat TC-05, TC-08, dan TC-09 membuat keputusan tidak berubah.
3. **R-08** (log berisi email dan role) tidak diuji.
4. **R-10:** validasi registrasi langsung ke Firebase Auth tidak diuji (hanya UI).
5. **Halaman admin lanjutan dan analisis guru** di luar scope (P04).
6. **Rules produksi** milik pemilik aplikasi tidak diuji. Hasil R-01, R-06, dan R-07 hanya berlaku untuk rules rancangan pada project uji.
7. **Kompatibilitas browser dan performa** di luar scope.
8. **Bukti tidak langsung:** data diperiksa lewat API dan UI (jumlah sebelum dan sesudah), bukan dengan melihat dokumen Firestore.
9. **Setiap test case dieksekusi satu kali.** Belum ada pengulangan untuk memastikan reproducibility.

## 5. Perbandingan dengan Exit Criteria dan Quality Gate

### 5.1 Exit criteria (P04 bagian 4.2)

| No | Exit criteria | Hasil | Status |
|---|---|---|---|
| 1 | Seluruh test case dieksekusi dengan actual, status, dan evidence | 12 dari 12 dieksekusi | **Terpenuhi** (setelah catatan 3.2 dirapikan) |
| 2 | 100% prioritas tinggi lulus dan minimal 90% lulus keseluruhan | 3 dari 7 (43%) dan 6 dari 12 (50%) | **Tidak terpenuhi** |
| 3 | Semua defect terdokumentasi dengan severity, priority, status | 8 defect tercatat di P07 | **Terpenuhi** |
| 4 | Tidak ada Critical open, High open punya mitigasi tertulis | 3 Critical open, 3 High tanpa mitigasi | **Tidak terpenuhi** |
| 5 | RTM tanpa baris kosong | Semua requirement terhubung. Kolom Defect dan Evidence perlu dilengkapi. | **Sebagian** |

### 5.2 Quality Gate QG-1 (8 kriteria)

| No | Kriteria | Ambang | Hasil | Status |
|---|---|---|---|---|
| 1 | Lint dan build | Lint bebas error, build sukses | Build gagal (R-04, DEF-07). Hasil lint belum ada. | **Tidak terpenuhi** |
| 2 | Eksekusi test case | 100% dieksekusi | 12 dari 12 | **Terpenuhi** |
| 3 | Pass rate | 100% tinggi, minimal 90% total | 43% tinggi, 50% total | **Tidak terpenuhi** |
| 4 | Defect terbuka | Tanpa Critical, tanpa High tanpa mitigasi | 3 Critical, 3 High | **Tidak terpenuhi** |
| 5 | Keamanan dasar | `/api/seed`, `/api/test-results`, debug tertutup tanpa otorisasi | TC-05, TC-08, TC-09 gagal | **Tidak terpenuhi** |
| 6 | Keterlacakan | Tiap requirement punya test case | 9 dari 9 | **Terpenuhi** |
| 7 | Isolasi hak akses | Siswa tidak dapat mengubah role atau membuka dashboard lain | TC-06 gagal | **Tidak terpenuhi** |
| 8 | Validasi data hasil | Server menolak data di luar rentang | TC-05 gagal | **Tidak terpenuhi** |

**Terpenuhi: 2 dari 8** (kriteria 2 dan 6).

### 5.3 Residual risk setelah audit

| Risiko | Tingkat residual | Penjelasan |
|---|---|---|
| R-04 Build gagal | **Tinggi** | Belum diperbaiki. Aplikasi tidak dapat dirilis. |
| R-02 API tanpa auth | **Tinggi** | Terbukti, belum diperbaiki. Data siswa dan hasil dapat dibaca dan dipalsukan. |
| R-01 Eskalasi role | **Tinggi** pada rules rancangan | Rules produksi belum diketahui. |
| R-03 Debug terbuka | **Sedang** | Terbukti, perbaikannya sederhana. |
| R-05 Seri | **Sedang** | Terbukti, bergantung pada konfirmasi aturan seri. |
| R-06 dan R-08 | **Tidak diketahui** | Tidak diuji. |
| R-09, R-10, R-07 | **Rendah** | Tidak terbukti, atau hanya sebagian diuji. |

## 6. Argumentasi Keputusan

**Aturan keputusan** (P02 dan P04): *Go* bila semua kriteria terpenuhi. *Conditional Go* bila hanya kriteria yang dapat dimitigasi yang tidak terpenuhi. *No-Go* bila ada defect Critical, atau kriteria 5 atau 7 tidak terpenuhi.

Ketiga syarat No-Go terpenuhi sekaligus:
- **Ada defect Critical.** DEF-02, DEF-03, DEF-07 (bagian 5.2 kriteria 4).
- **Kriteria 5 tidak terpenuhi.** Endpoint hasil tes (TC-05, TC-08) dan endpoint debug (TC-09) dapat diakses tanpa otorisasi.
- **Kriteria 7 tidak terpenuhi.** TC-06 menunjukkan siswa dapat mengubah role menjadi admin.

**Mengapa bukan Conditional Go.** Conditional Go mensyaratkan kegagalan yang bisa dimitigasi. Defect yang ada menyangkut kebocoran dan pemalsuan data siswa serta eskalasi hak akses. Mitigasi sementara (misalnya hanya merahasiakan alamat endpoint) tidak mencegah pengguna yang mengetahui `userId` atau mengubah parameter URL. Build yang gagal juga membuat rilis tidak mungkin.

**Hal yang berjalan baik.** Registrasi dengan batas password 6 karakter (TC-01 sampai TC-03) berfungsi. Alur tes normal, penolakan pengiriman saat soal belum lengkap, dan pemulihan setelah gangguan jaringan tanpa duplikasi data (TC-10 sampai TC-12) juga lulus. Fondasi fungsional baik, tetapi lapisan keamanannya belum siap.

**Batasan kesimpulan.** Keputusan berlaku untuk versi 0.1.0 pada arsip yang diterima 7 November 2026, dengan project Firebase uji dan rules rancangan. Temuan R-01, R-06, dan R-07 belum membuktikan kondisi pada rules produksi pemilik aplikasi. Karena dieksekusi sekali dan dengan bukti tidak langsung, hasil perlu dikonfirmasi ulang pada pengujian ulang.

## 7. Alternatif Tindakan dan Trade-off

| Opsi | Isi | Keuntungan | Trade-off dan risiko | Penilaian |
|---|---|---|---|---|
| **A. Perbaiki dulu, uji ulang, nilai ulang gate** (**direkomendasikan**) | Perbaiki urutan: DEF-07 (build), DEF-02 dan DEF-01 (verifikasi ID token dan validasi di `route.ts`), DEF-03 (rules `role`), DEF-04 (debug), DEF-05, DEF-06, DEF-08. Ulangi TC-04 sampai TC-09 dan PRE-01. | Menutup risiko inti. Gate dapat dinilai ulang dengan bukti. | Butuh waktu perbaikan dan retest. Perlu kerja sama pemilik aplikasi karena kode dan rules miliknya. | Paling aman |
| **B. Conditional Go terbatas** | Rilis hanya untuk demo internal tanpa data siswa nyata, dengan endpoint debug dimatikan. | Cepat. Dapat memberi umpan balik pengguna. | DEF-02, DEF-03, DEF-07 tetap ada. Build harus tetap diperbaiki. Risiko data nyata bocor bila demo meluas. | Hanya jika data benar-benar dummy |
| **C. Mitigasi infrastruktur** | Perketat rules Firestore dan batasi akses jaringan ke API tanpa mengubah kode. | Tidak perlu mengubah kode aplikasi. | Rules tidak memperbaiki DEF-01 dan DEF-07. Mudah salah konfigurasi. Tidak menggantikan verifikasi di server. | Pelengkap, bukan pengganti A |

**Kondisi untuk berubah menjadi Conditional Go atau Go:**
1. DEF-02, DEF-03, DEF-07 berstatus *Closed* setelah retest.
2. Defect High (DEF-01, DEF-04, DEF-05) berstatus *Closed* atau memiliki mitigasi tertulis.
3. Pass rate memenuhi QG-1 (100% prioritas tinggi, minimal 90% total).
4. Tambahkan test case untuk celah di bagian 4.3, minimal AC-02 dan `/api/seed`.

## 8. Umpan Balik (Feedback Loop)

Gate gagal, sehingga jalur **Loop gate** dari P02 aktif: laporan gate → keputusan No-Go → perbaikan oleh pengembang → evaluasi ulang. Setelah perbaikan, artefak yang diperbarui adalah test case dan RTM (status dan defect), register risiko (skor R-01, R-02, R-04 setelah mitigasi), dan laporan defect (status Closed atau Reopened). Rekomendasi perbaikan di P07 bagian 5 disampaikan kepada pemilik aplikasi.

## 9. Kesimpulan Tahap 8

- **Keputusan: No-Go.** 3 defect Critical open, kriteria QG-1 nomor 5 dan 7 tidak terpenuhi, dan hanya 2 dari 8 kriteria terpenuhi.
- Pass rate 6 dari 12 (50%). Semua kegagalan terkait keamanan dan validasi server, sedangkan fungsi dasar lulus.
- Coverage: 9 dari 9 requirement dan 8 dari 9 acceptance criteria diuji. Blind spot utama adalah AC-02, `/api/seed`, R-08, dan rules produksi.
- Rekomendasi: Opsi A, yaitu memperbaiki dulu lalu menguji ulang.
- Keterlacakan: P01 (potensi masalah) → P03 (risiko dan requirement) → P04 (strategi dan gate) → P05/P06 (test case) → P07 (defect) → keputusan di dokumen ini.
