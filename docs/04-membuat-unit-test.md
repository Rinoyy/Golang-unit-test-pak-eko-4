# 04 — Membuat Unit Test

> Video: 00:13:27 — 00:25:37

---

## Saatnya Praktek!

Di sini kita akan membuat unit test pertama kamu. Dari nol. Satu langkah demi satu langkah.

---

## Langkah 1: Buat Folder dan File

Buat folder project:

```bash
mkdir golang_unit_test
cd golang_unit_test
go mod init belajar_testing
```

**Istilah: `go mod init`**

Artinya:
Mendaftarkan folder ini sebagai project Go. Go perlu tahu ini adalah sebuah project agar bisa mengelola dependensi.

Analogi:
Seperti membuat kartu nama untuk projectmu. "Hei Go, project ini namanya `belajar_testing`."

---

## Langkah 2: Buat Kode yang Akan Ditest

Buat file `calculator.go`:

```go
package main

// Tambah menjumlahkan dua bilangan bulat
func Tambah(a int, b int) int {
    return a + b
}

// Kurang mengurangkan b dari a
func Kurang(a int, b int) int {
    return a - b
}

// Kali mengalikan dua bilangan bulat
func Kali(a int, b int) int {
    return a * b
}

// Bagi membagi a dengan b
// Kembalikan 0 jika b adalah 0 (hindari crash)
func Bagi(a int, b int) int {
    if b == 0 {
        return 0
    }
    return a / b
}
```

### Penjelasan setiap baris:

```go
package main
```
Ini adalah "nama kelompok" dari file ini. Semua file di folder yang sama harus punya package yang sama.

```go
func Tambah(a int, b int) int {
```
Membuat fungsi bernama `Tambah`. Menerima dua parameter `a` dan `b` bertipe `int` (bilangan bulat). Mengembalikan `int`.

```go
    return a + b
```
Kembalikan hasil penjumlahan `a` dan `b`.

```go
if b == 0 {
    return 0
}
```
Kondisi khusus: jika `b` adalah 0, kembalikan 0. Ini mencegah error "division by zero" yang bisa crash program.

---

## Langkah 3: Buat File Test

Buat file `calculator_test.go`:

```go
package main

import "testing"

func TestTambah(t *testing.T) {
    hasil := Tambah(2, 3)
    expected := 5

    if hasil != expected {
        t.Errorf("Tambah(2, 3) = %d, expected %d", hasil, expected)
    }
}
```

### Penjelasan setiap baris:

```go
package main
```
Harus sama dengan package di `calculator.go` agar bisa mengakses fungsi-fungsinya.

```go
import "testing"
```
Import package testing bawaan Go. Wajib ada.

```go
func TestTambah(t *testing.T) {
```
Fungsi test. Nama harus mulai `Test`. Parameter `t *testing.T` adalah alat untuk laporan test.

```go
    hasil := Tambah(2, 3)
```
**Bagian ACT**: Jalankan fungsi yang ingin diuji. Simpan hasilnya di variabel `hasil`.

```go
    expected := 5
```
**Bagian ARRANGE**: Tentukan apa hasil yang kamu harapkan.

```go
    if hasil != expected {
```
**Bagian ASSERT**: Bandingkan `hasil` dengan `expected`. Kalau berbeda, berarti ada masalah.

```go
        t.Errorf("Tambah(2, 3) = %d, expected %d", hasil, expected)
```
Lapor ke `t` bahwa test gagal. `Errorf` seperti `Error` tapi bisa format string dengan `%d` (digit/angka).

**Istilah: `%d`**

Artinya:
Placeholder untuk angka (digit) dalam format string. Seperti tempat kosong yang diisi angka.

Contoh: `"Hasil: %d" dengan angka 5` → `"Hasil: 5"`

---

## Langkah 4: Jalankan Test

```bash
go test -v
```

Output yang seharusnya muncul:

```
=== RUN   TestTambah
--- PASS: TestTambah (0.00s)
PASS
ok      belajar_testing    0.002s
```

Artinya test berhasil!

---

## Tambahkan Test Lainnya

Sekarang lengkapi `calculator_test.go` dengan test untuk semua fungsi:

```go
package main

import "testing"

func TestTambah(t *testing.T) {
    hasil := Tambah(2, 3)
    expected := 5

    if hasil != expected {
        t.Errorf("Tambah(2, 3) = %d, expected %d", hasil, expected)
    }
}

func TestKurang(t *testing.T) {
    hasil := Kurang(10, 3)
    expected := 7

    if hasil != expected {
        t.Errorf("Kurang(10, 3) = %d, expected %d", hasil, expected)
    }
}

func TestKali(t *testing.T) {
    hasil := Kali(4, 5)
    expected := 20

    if hasil != expected {
        t.Errorf("Kali(4, 5) = %d, expected %d", hasil, expected)
    }
}

func TestBagi(t *testing.T) {
    hasil := Bagi(10, 2)
    expected := 5

    if hasil != expected {
        t.Errorf("Bagi(10, 2) = %d, expected %d", hasil, expected)
    }
}

func TestBagiDenganNol(t *testing.T) {
    hasil := Bagi(10, 0)
    expected := 0  // kita sepakat mengembalikan 0 jika dibagi 0

    if hasil != expected {
        t.Errorf("Bagi(10, 0) = %d, expected %d", hasil, expected)
    }
}
```

Jalankan lagi:

```bash
go test -v
```

Output:

```
=== RUN   TestTambah
--- PASS: TestTambah (0.00s)
=== RUN   TestKurang
--- PASS: TestKurang (0.00s)
=== RUN   TestKali
--- PASS: TestKali (0.00s)
=== RUN   TestBagi
--- PASS: TestBagi (0.00s)
=== RUN   TestBagiDenganNol
--- PASS: TestBagiDenganNol (0.00s)
PASS
ok      belajar_testing    0.003s
```

Semua berhasil!

---

## Cara Berpikir Saat Membuat Test

Sebelum menulis satu baris kode test, tanyakan:

**Apa yang diuji?**
→ Fungsi `Tambah` dengan input 2 dan 3

**Kenapa diuji?**
→ Karena ini fungsi dasar yang dipakai banyak tempat — kalau ini salah, semuanya salah

**Apa kemungkinan gagal?**
→ Kalau ada bug di operator `+`, hasilnya tidak akan 5
→ Kalau tipe data salah, bisa overflow

---

## Percobaan Menarik: Buat Fungsi yang Salah

Coba ubah fungsi `Tambah` menjadi salah:

```go
func Tambah(a int, b int) int {
    return a + b + 1  // sengaja ditambah 1 (ada bug!)
}
```

Jalankan test:

```bash
go test -v
```

Output:

```
=== RUN   TestTambah
    calculator_test.go:10: Tambah(2, 3) = 6, expected 5
--- FAIL: TestTambah (0.00s)
FAIL
exit status 1
```

**Test berhasil mendeteksi bug!**

Ini adalah kekuatan testing — bug terdeteksi otomatis, tanpa perlu kamu jalankan program dan coba manual.

Kembalikan kode ke yang benar setelah mencoba ini.

---

## Cara Berpikir: Test Apa Saja yang Perlu Dibuat?

Untuk setiap fungsi, pikirkan:

1. **Happy path** — kondisi normal yang diharapkan berhasil
2. **Edge case** — kondisi batas/tepi yang mungkin bermasalah
3. **Error case** — kondisi yang seharusnya menghasilkan error

Untuk fungsi `Bagi`:

| Kasus | Input | Expected | Alasan |
|---|---|---|---|
| Happy path | 10, 2 | 5 | Pembagian normal |
| Edge case | 0, 5 | 0 | Angka 0 dibagi sesuatu |
| Error case | 10, 0 | 0 | Dibagi nol, harus ditangani |

---

## Analogi: Dokter yang Memeriksa Pasien

Membuat unit test seperti dokter yang memeriksa pasien:

1. **Dokter tahu apa yang seharusnya normal** (expected) — tekanan darah normal adalah 120/80
2. **Dokter mengukur kondisi sebenarnya** (actual) — tekanan darah pasien terukur 150/90
3. **Dokter bandingkan** — 150/90 ≠ 120/80 → ada masalah!
4. **Dokter lapor** — "pasien mengalami hipertensi"

Test kamu persis seperti dokter itu — tahu nilai normal, ukur nilai aktual, bandingkan, laporkan kalau ada masalah.

---

## Kesalahan Umum Pemula

1. **Hanya test satu kasus** — padahal ada banyak edge case yang perlu ditest
2. **Tidak beri pesan yang informatif** — `t.Error("salah")` tidak berguna. Tulis pesan yang jelas seperti `t.Errorf("Tambah(2,3) = %d, expected 5", hasil)`
3. **Bergantung ke urutan test** — setiap test harus bisa berjalan sendiri
4. **Variabel `expected` tidak didefinisikan** — langsung bandingkan dengan angka di if (boleh, tapi lebih jelas kalau pakai variabel)

---

## Latihan

**Tugas:**

Buat fungsi `Kuadrat(n int) int` yang mengembalikan `n * n`.

Lalu buat test untuk fungsi tersebut dengan kasus:
1. `Kuadrat(3)` → expected 9
2. `Kuadrat(0)` → expected 0
3. `Kuadrat(-4)` → expected 16

---

### Jawaban

Fungsi (`calculator.go`):
```go
func Kuadrat(n int) int {
    return n * n
}
```

Test (`calculator_test.go`):
```go
func TestKuadrat(t *testing.T) {
    hasil := Kuadrat(3)
    if hasil != 9 {
        t.Errorf("Kuadrat(3) = %d, expected 9", hasil)
    }
}

func TestKuadratNol(t *testing.T) {
    hasil := Kuadrat(0)
    if hasil != 0 {
        t.Errorf("Kuadrat(0) = %d, expected 0", hasil)
    }
}

func TestKuadratNegatif(t *testing.T) {
    hasil := Kuadrat(-4)
    if hasil != 16 {
        t.Errorf("Kuadrat(-4) = %d, expected 16", hasil)
    }
}
```

---

## Selanjutnya

Lanjut ke: [05 — Menggagalkan Test](05-menggagalkan-test.md)

Di sana kita belajar lebih dalam tentang berbagai cara untuk "melaporkan kegagalan" test.
