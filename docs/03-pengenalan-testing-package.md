# 03 — Pengenalan Testing Package

> Video: 00:10:15 — 00:13:27

---

## Apa Itu Testing Package?

Di Go (Golang), sudah ada **alat bawaan** untuk membuat test.

Alat itu adalah **package `testing`**.

**Istilah: Package**

Artinya:
Kumpulan kode yang sudah disiapkan oleh Go dan siap kamu gunakan.

Analogi:
Seperti kotak peralatan tukang. Di dalam ada palu, obeng, meteran. Kamu tidak perlu membuat sendiri — tinggal ambil dan pakai.

---

## Kenapa Go Punya Testing Bawaan?

Banyak bahasa pemrograman tidak punya alat testing bawaan — kamu harus install alat dari luar.

Go memilih untuk **menyertakan testing sejak awal** karena tim Go percaya bahwa testing adalah bagian wajib dari menulis kode yang baik.

Ini artinya:
- Kamu tidak perlu install apa-apa
- Cara membuat test sudah ada standarnya
- Semua developer Go menggunakan cara yang sama

---

## Cara Kerja Testing di Go

Go punya **konvensi** (aturan tidak tertulis yang wajib diikuti):

### Aturan 1: Nama File Test

File test harus berakhiran `_test.go`.

```
calculator.go       ← kode program biasa
calculator_test.go  ← kode test (harus diakhiri _test.go)
```

Kalau tidak diakhiri `_test.go`, Go tidak akan mengenalinya sebagai file test.

### Aturan 2: Nama Fungsi Test

Fungsi test harus dimulai dengan kata `Test` (huruf T kapital).

```go
func TestTambah(t *testing.T) { ... }   ← BENAR
func testTambah(t *testing.T) { ... }   ← SALAH (t kecil)
func UjiTambah(t *testing.T) { ... }    ← SALAH (tidak mulai dengan Test)
```

### Aturan 3: Parameter Fungsi Test

Fungsi test harus menerima satu parameter: `t *testing.T`

```go
func TestTambah(t *testing.T) {
    // t adalah "alat" yang kamu gunakan untuk
    // memberitahu Go bahwa test berhasil atau gagal
}
```

**Istilah: `*testing.T`**

Artinya:
Sebuah "objek alat" yang diberikan Go ke dalam fungsi testmu. Kamu pakai `t` ini untuk:
- Memberitahu Go bahwa test gagal: `t.Fail()`
- Mencatat pesan error: `t.Error("pesan")`
- Menghentikan test: `t.Fatal("pesan")`

Analogi:
Bayangkan `t` seperti wasit di pertandingan sepak bola. Kalau ada pelanggaran, kamu lapor ke wasit (`t`). Wasit yang kemudian menentukan test gagal atau berhasil.

---

## Cara Menjalankan Test

Buka terminal di folder project, lalu ketik:

```bash
go test
```

Atau kalau mau lebih detail:

```bash
go test -v
```

**Istilah: `-v` (verbose)**

Artinya:
Tampilkan detail semua test yang berjalan, bukan hanya hasil akhirnya.

Analogi:
Kalau `go test` seperti laporan akhir "LULUS/TIDAK LULUS", maka `go test -v` seperti laporan lengkap per mata pelajaran.

---

## Contoh Output Test

Tanpa `-v`:
```
ok      github.com/namauser/project    0.002s
```

Artinya: semua test berhasil, selesai dalam 0.002 detik.

Dengan `-v`:
```
=== RUN   TestTambah
--- PASS: TestTambah (0.00s)
=== RUN   TestKurang
--- PASS: TestKurang (0.00s)
PASS
ok      github.com/namauser/project    0.002s
```

Artinya: TestTambah berhasil, TestKurang berhasil.

Kalau ada yang gagal:
```
=== RUN   TestTambah
--- FAIL: TestTambah (0.00s)
    calculator_test.go:10: Expected 5, got 6
FAIL
```

---

## Struktur Dasar File Test

```go
// File: calculator_test.go

package main  // harus sama dengan package yang ditest

import "testing"  // import package testing bawaan Go

func TestTambah(t *testing.T) {
    // isi test di sini
}

func TestKurang(t *testing.T) {
    // isi test di sini
}
```

Perhatikan:
- `package main` — harus sama dengan file yang ditest
- `import "testing"` — wajib di setiap file test
- Nama fungsi dimulai `Test`
- Parameter `t *testing.T`

---

## Apa yang Bisa Dilakukan dengan `t`

| Perintah | Artinya |
|---|---|
| `t.Log("pesan")` | Cetak pesan (hanya muncul kalau test gagal atau pakai `-v`) |
| `t.Error("pesan")` | Tandai test sebagai GAGAL, tapi lanjutkan |
| `t.Fatal("pesan")` | Tandai test sebagai GAGAL, lalu hentikan |
| `t.Fail()` | Tandai test sebagai GAGAL tanpa pesan |
| `t.FailNow()` | Tandai test sebagai GAGAL dan hentikan sekarang |

Perbedaan `t.Error` vs `t.Fatal`:

```go
func TestContoh(t *testing.T) {
    t.Error("ini gagal")     // test ditandai gagal, tapi LANJUT
    fmt.Println("masih jalan") // ini tetap dieksekusi
}
```

```go
func TestContoh(t *testing.T) {
    t.Fatal("ini gagal")     // test ditandai gagal dan BERHENTI
    fmt.Println("tidak dieksekusi") // ini TIDAK dieksekusi
}
```

---

## Kenapa Ada Dua Cara (Error vs Fatal)?

**Gunakan `t.Error`** ketika kamu ingin lanjutkan memeriksa kondisi lain meskipun ada yang gagal.

**Gunakan `t.Fatal`** ketika kalau satu kondisi gagal, tidak masuk akal untuk lanjut ke kondisi berikutnya.

Analogi:
- `t.Error` = Kamu menemukan lubang di jalan. Kamu catat, tapi lanjut berjalan untuk cek kondisi jalan lainnya.
- `t.Fatal` = Kamu menemukan jembatan putus. Tidak perlu lanjut — tidak bisa lewat sama sekali.

---

## Cara Menjalankan Test Satu Fungsi Saja

Kalau kamu hanya mau jalankan `TestTambah`:

```bash
go test -run TestTambah
```

Kalau mau jalankan semua test yang namanya mengandung kata "Tambah":

```bash
go test -run Tambah
```

---

## Analogi Keseluruhan: Formulir Laporan

Package `testing` adalah seperti formulir laporan standar.

- Semua test punya format yang sama
- Semua developer mengisi formulir dengan cara yang sama
- Go tahu cara membaca formulir itu dan memberikan hasil

`t` adalah pulpen yang kamu gunakan untuk mengisi formulir:
- `t.Error` = tulis di kolom "ada masalah, tapi lanjut diperiksa"
- `t.Fatal` = tulis di kolom "masalah fatal, hentikan pemeriksaan"

---

## Kesalahan Umum Pemula

1. **Lupa `_test.go`** — file tidak dikenali sebagai test
2. **Nama fungsi tidak mulai `Test`** — fungsi tidak dijalankan
3. **Lupa `import "testing"`** — error saat compile
4. **Package berbeda** — Go tidak bisa akses fungsi yang ditest
5. **Tidak pakai parameter `t`** — Go tidak tahu cara menerima laporan dari test

---

## Latihan

Tanpa kode dulu. Jawab pertanyaan ini:

**Pertanyaan 1:**
Kamu punya file `user.go` dengan `package main`. Bagaimana nama file test-nya? Dan apa isi baris pertama file test tersebut?

**Pertanyaan 2:**
Kamu mau test fungsi `HitungDiskon`. Bagaimana nama fungsi test-nya?

**Pertanyaan 3:**
Apa bedanya `t.Error` dan `t.Fatal`? Kapan kamu pakai yang mana?

---

### Jawaban

**Jawaban 1:**
- Nama file: `user_test.go`
- Baris pertama: `package main`

**Jawaban 2:**
- `func TestHitungDiskon(t *testing.T) { ... }`

**Jawaban 3:**
- `t.Error` → test ditandai gagal tapi tetap lanjut
- `t.Fatal` → test ditandai gagal dan berhenti di situ
- Pakai `t.Error` ketika mau periksa banyak hal sekaligus
- Pakai `t.Fatal` ketika kalau satu hal gagal, tidak ada gunanya lanjut

---

## Selanjutnya

Lanjut ke: [04 — Membuat Unit Test](04-membuat-unit-test.md)

Di sana kita langsung praktek membuat unit test pertama kamu!
