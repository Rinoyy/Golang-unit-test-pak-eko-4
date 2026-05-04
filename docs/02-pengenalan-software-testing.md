# 02 — Pengenalan Software Testing

> Video: 00:01:57 — 00:10:15

---

## Apa Itu Software Testing?

**Software Testing** adalah proses **memeriksa apakah kode yang kamu tulis bekerja sesuai yang diharapkan**.

Kamu memberikan "pertanyaan" ke kode:
- "Kalau saya kasih angka 2 dan 3, apakah hasilnya 5?"
- "Kalau saya kasih input kosong, apakah tidak crash?"
- "Kalau saya kasih angka negatif, apakah hasilnya masuk akal?"

Kode harus bisa menjawab pertanyaan-pertanyaan itu dengan benar.

---

## Kenapa Testing Ada?

### Masalah yang dialami developer tanpa testing:

**Masalah 1: Bug ditemukan terlambat**

Bayangkan kamu membangun aplikasi belanja online. Setelah 3 bulan pengembangan, baru ketahuan ada bug di halaman pembayaran. User sudah kehilangan uang. Reputasi rusak.

**Masalah 2: Takut mengubah kode**

Kamu punya kode lama yang berjalan. Kamu takut mengubahnya karena tidak tahu dampaknya ke bagian lain. Akibatnya kode makin "busuk" seiring waktu.

**Masalah 3: Uji manual yang melelahkan**

Setiap kali ada perubahan, kamu harus klik-klik manual untuk memastikan semua fitur masih jalan. Ini membuang waktu dan membosankan.

**Testing menyelesaikan semua masalah itu.**

---

## Jenis-Jenis Testing

Ada banyak jenis testing, tapi kita mulai dari yang paling dasar:

---

### 1. Unit Test

**Istilah: Unit Test**

Artinya:
Test yang memeriksa satu fungsi atau satu bagian kecil kode secara terisolasi.

Analogi:
Kamu punya mesin mobil. Unit test seperti memeriksa setiap komponen satu per satu: apakah busi berfungsi? Apakah karburator berfungsi? Tidak perlu nyalakan mobilnya dulu.

Contoh:
```go
// Uji fungsi Tambah saja, tidak perlu jalankan seluruh program
func TestTambah(t *testing.T) {
    hasil := Tambah(2, 3)
    // apakah hasilnya 5?
}
```

Catatan:
Ini yang paling sering dibuat dan yang paling mudah dipahami.

---

### 2. Integration Test

**Istilah: Integration Test**

Artinya:
Test yang memeriksa apakah beberapa bagian kode bisa bekerja sama dengan benar.

Analogi:
Setelah setiap komponen mobil diuji satu per satu, sekarang kita uji apakah mesin + transmisi + roda bisa bekerja bersama.

Catatan:
Kita belum fokus ke sini dulu. Pahami unit test terlebih dahulu.

---

### 3. End-to-End Test

**Istilah: End-to-End Test (E2E Test)**

Artinya:
Test yang mensimulasikan penggunaan nyata dari awal sampai akhir.

Analogi:
Uji coba mobil dengan cara mengendarainya di jalan asli.

Catatan:
Ini yang paling kompleks. Kita tidak bahas di materi ini.

---

## Cara Berpikir Saat Testing

Ini adalah **cara berpikir paling penting** yang harus kamu kuasai.

Setiap kali kamu mau membuat test, tanya tiga pertanyaan ini:

### Pertanyaan 1: Apa yang ingin diuji?

Pilih satu hal spesifik. Bukan "saya mau uji fungsi login" tapi:
- "Saya mau uji: apakah login berhasil ketika username dan password benar?"
- "Saya mau uji: apakah login gagal ketika password salah?"

Satu test = satu hal yang diuji.

### Pertanyaan 2: Kenapa diuji?

Karena ini adalah kondisi yang mungkin terjadi di dunia nyata. Kalau tidak diuji, dan ternyata salah — user yang kena dampaknya.

### Pertanyaan 3: Apa kemungkinan gagal?

Coba bayangkan: dalam kondisi apa fungsi ini bisa rusak?
- Data kosong?
- Angka negatif?
- String yang sangat panjang?
- Angka nol?

---

## Expected vs Actual — Konsep Paling Dasar

Ini adalah inti dari semua testing:

**Istilah: Expected**

Artinya:
Hasil yang KAMU HARAPKAN dari fungsi tersebut.

Analogi:
Kamu pesan nasi goreng. Kamu HARAPKAN mendapat nasi goreng.

---

**Istilah: Actual**

Artinya:
Hasil yang SEBENARNYA keluar dari fungsi tersebut.

Analogi:
Nasi goreng yang BENAR-BENAR disajikan ke mejamu.

---

Sebuah test **BERHASIL** jika: `Expected == Actual`

Sebuah test **GAGAL** jika: `Expected != Actual`

### Contoh Nyata:

```
Fungsi: Tambah(2, 3)

Expected: 5      <-- yang kita harapkan
Actual:   5      <-- yang keluar dari fungsi

Hasil: TEST BERHASIL ✓
```

```
Fungsi: Tambah(2, 3)

Expected: 5      <-- yang kita harapkan
Actual:   6      <-- ada bug di fungsi!

Hasil: TEST GAGAL ✗
```

---

## Apa Itu Unit Test Secara Lebih Detail

Unit test punya tiga bagian utama yang sering disebut **AAA**:

### A — Arrange (Siapkan)

Siapkan semua yang dibutuhkan untuk menjalankan test.

```go
// Siapkan angka yang akan dijumlahkan
angka1 := 2
angka2 := 3
```

### A — Act (Lakukan)

Jalankan fungsi yang ingin diuji.

```go
// Jalankan fungsi Tambah
hasil := Tambah(angka1, angka2)
```

### A — Assert (Periksa)

Periksa apakah hasilnya sesuai yang diharapkan.

```go
// Periksa apakah hasilnya 5
if hasil != 5 {
    // test gagal!
}
```

---

## Isolasi Test — Kenapa Penting

**Istilah: Isolasi Test**

Artinya:
Setiap test harus bisa berjalan sendiri tanpa bergantung ke test lain, database, internet, atau sistem eksternal.

Analogi:
Bayangkan kamu menguji kualitas garam. Kamu tidak perlu masak seluruh makanan dulu. Kamu cukup ambil garamnya, cicipi langsung. Terisolasi, tidak bergantung ke bahan lain.

Kenapa penting?
- Test yang bergantung ke database bisa gagal bukan karena kode salah, tapi karena databasenya mati
- Test yang bergantung ke internet bisa gagal karena koneksi putus
- Itu bukan informasi yang berguna — kita tidak tahu apakah kode kita benar atau tidak

Contoh masalah:
```
TestLoginUser bergantung ke database
→ Database mati
→ TestLoginUser gagal
→ Kita bingung: kode salah? Atau database mati?
```

Dengan isolasi:
```
TestLoginUser menggunakan data palsu (mock)
→ Database mati tidak masalah
→ TestLoginUser tetap berjalan
→ Kalau gagal, pasti karena kode salah
```

---

## Kapan Test Dianggap Berhasil dan Gagal

| Kondisi | Status |
|---|---|
| Output fungsi sesuai yang diharapkan | BERHASIL (PASS) |
| Output fungsi tidak sesuai yang diharapkan | GAGAL (FAIL) |
| Fungsi crash / panic saat dijalankan | GAGAL (FAIL) |
| Test sengaja dilewati | SKIP |

---

## Analogi Keseluruhan: Tukang Kue

Bayangkan kamu adalah tukang kue yang membuat resep baru.

**Tanpa testing:**
Kamu buat kue, langsung jual ke customer. Ternyata kue terlalu asin. Customer kecewa.

**Dengan testing:**
1. Kamu buat adonan kecil dulu (unit test)
2. Kamu cicipi adonannya (assertion)
3. Kalau rasanya pas → lanjut
4. Kalau rasanya tidak pas → perbaiki resep dulu
5. Baru setelah semua oke, kue dijual ke customer

Testing adalah proses "cicip sebelum jual".

---

## Kesalahan Umum Pemula

1. **Hanya test yang "happy path"** — hanya uji kondisi sukses, lupa uji kondisi gagal
2. **Test terlalu banyak hal sekaligus** — satu test harusnya satu hal saja
3. **Test bergantung ke urutan** — test A harus dijalankan sebelum test B (ini salah)
4. **Tidak pernah jalankan test** — menulis test tapi tidak pernah dijalankan
5. **Test terlalu mirip dengan kode aslinya** — ini tidak menguji apa-apa

---

## Ringkasan

| Konsep | Penjelasan Singkat |
|---|---|
| Software Testing | Memeriksa apakah kode bekerja sesuai harapan |
| Unit Test | Test untuk satu fungsi kecil, terisolasi |
| Expected | Hasil yang kamu harapkan |
| Actual | Hasil yang benar-benar keluar |
| AAA | Arrange → Act → Assert |
| Isolasi | Test tidak bergantung ke sistem lain |

---

## Latihan

Sebelum lanjut, jawab pertanyaan ini (tanpa lihat jawaban dulu!):

**Pertanyaan 1:**
Kamu punya fungsi `Kali(a, b int) int`. Kamu mau test fungsi ini dengan input 3 dan 4.
- Apa yang kamu harapkan hasilnya?
- Bagaimana kamu tahu test berhasil?

**Pertanyaan 2:**
Kamu punya fungsi `Bagi(a, b int) int`. Kamu mau test dengan input 10 dan 2.
- Apa yang kamu harapkan?
- Apa yang terjadi kalau `b` adalah 0? Apakah perlu ditest juga?

---

### Jawaban

**Jawaban 1:**
- Expected: 12 (karena 3 × 4 = 12)
- Test berhasil jika `Kali(3, 4)` menghasilkan 12

**Jawaban 2:**
- Expected: 5 (karena 10 ÷ 2 = 5)
- YA, perlu ditest! Pembagian dengan 0 adalah kondisi berbahaya (bisa crash). Test harus memastikan fungsi menangani kondisi ini dengan benar.

---

## Selanjutnya

Lanjut ke: [03 — Pengenalan Testing Package](03-pengenalan-testing-package.md)

Di sana kamu akan kenal dengan "alat" yang dipakai Go untuk membuat test.
