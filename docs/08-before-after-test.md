# 08 — Before dan After Test

> Video: 00:46:35 — 00:51:27

---

## Masalah yang Diselesaikan

Bayangkan kamu punya 10 test, dan setiap test butuh:
- Membuka koneksi database dulu
- Menjalankan test
- Menutup koneksi database

Tanpa fitur "before dan after", kamu harus tulis kode buka/tutup koneksi di setiap test:

```go
func TestA(t *testing.T) {
    db := BukaDatabase()          // duplikat!
    defer db.Tutup()              // duplikat!
    // isi test A
}

func TestB(t *testing.T) {
    db := BukaDatabase()          // duplikat!
    defer db.Tutup()              // duplikat!
    // isi test B
}
```

Ini tidak efisien dan rentan kesalahan.

---

## Solusi: TestMain

Go menyediakan fungsi khusus bernama `TestMain` untuk menjalankan kode **sebelum** dan **sesudah** semua test berjalan.

**Istilah: TestMain**

Artinya:
Fungsi spesial di Go yang menjadi "pintu masuk" semua test. Kamu bisa menjalankan kode setup sebelum test dan cleanup setelah semua test selesai.

Analogi:
Seperti pembukaan dan penutupan di sebuah acara. Sebelum acara dimulai, panitia menyiapkan ruangan (setup). Setelah acara selesai, panitia membereskan ruangan (teardown). Peserta (test) hanya fokus pada acaranya saja.

---

## Struktur TestMain

```go
func TestMain(m *testing.M) {
    // === SEBELUM SEMUA TEST (SETUP) ===
    fmt.Println("Menyiapkan semua yang dibutuhkan...")

    // Jalankan semua test
    exitCode := m.Run()

    // === SETELAH SEMUA TEST (TEARDOWN) ===
    fmt.Println("Membersihkan semua resource...")

    // Keluar dengan exit code dari test
    os.Exit(exitCode)
}
```

**Penjelasan setiap bagian:**

```go
func TestMain(m *testing.M) {
```
Nama fungsi ini wajib persis `TestMain`. Parameter `m *testing.M` adalah objek yang mengontrol jalannya semua test.

```go
exitCode := m.Run()
```
**`m.Run()`** adalah perintah untuk menjalankan semua test. Wajib dipanggil! Kalau tidak dipanggil, tidak ada test yang berjalan.

`exitCode` adalah kode keluar: `0` berarti semua berhasil, bukan `0` berarti ada yang gagal.

```go
os.Exit(exitCode)
```
Keluar dari program dengan kode yang sesuai. Ini penting agar CI/CD system tahu apakah test berhasil atau gagal.

---

## Contoh Nyata: Setup dan Teardown Database

```go
package main

import (
    "fmt"
    "os"
    "testing"
)

// Variabel yang dibagi antar test
var testDB *Database

func TestMain(m *testing.M) {
    // SETUP: siapkan database test
    fmt.Println("Membuka koneksi database test...")
    testDB = BukaDatabase("test_db")

    // Isi database test dengan data awal
    testDB.IsiDataAwal()

    // Jalankan semua test
    exitCode := m.Run()

    // TEARDOWN: bersihkan setelah semua test selesai
    fmt.Println("Menutup dan membersihkan database test...")
    testDB.HapusSemua()
    testDB.Tutup()

    os.Exit(exitCode)
}

func TestAmbilUser(t *testing.T) {
    // testDB sudah siap, langsung pakai
    user := testDB.AmbilUser("user_1")
    assert.Equal(t, "Budi", user.Nama)
}

func TestHapusUser(t *testing.T) {
    // testDB sudah siap, langsung pakai
    err := testDB.HapusUser("user_2")
    assert.NoError(t, err)
}
```

---

## Cara Lain: Setup per Test dengan `defer`

Selain `TestMain` yang untuk semua test, kamu juga bisa setup per test menggunakan `defer`.

**Istilah: `defer`**

Artinya:
Perintah kepada Go untuk menjalankan sebuah fungsi **saat fungsi yang sedang berjalan selesai** (baik berhasil maupun gagal).

Analogi:
Kamu memasak. Sebelum mulai masak, kamu bilang ke asisten: "Nanti kalau saya sudah selesai masak, tolong cuci semua piring." Kamu tidak perlu ingat-ingat untuk cuci piring — asisten akan melakukannya otomatis.

```go
func TestKoneksi(t *testing.T) {
    db := BukaDatabase()
    defer db.Tutup()  // akan dieksekusi saat TestKoneksi selesai

    // isi test
    // tidak perlu ingat tutup database — defer yang urus
}
```

---

## Kombinasi TestMain + defer

Keduanya bisa dipakai bersama:

```go
// TestMain: setup SEKALI untuk semua test
func TestMain(m *testing.M) {
    testDB = BukaDatabase("test")
    exitCode := m.Run()
    testDB.Tutup()
    os.Exit(exitCode)
}

// defer: cleanup per test
func TestTambahUser(t *testing.T) {
    // Buat user baru untuk test ini
    user := testDB.TambahUser("Siti", 30)
    defer testDB.HapusUser(user.ID)  // hapus setelah test ini selesai

    // Test apakah user berhasil dibuat
    assert.Equal(t, "Siti", user.Nama)
}
```

---

## Contoh Lebih Sederhana (Tanpa Database)

Untuk latihan, kita tidak perlu database. Ini contoh dengan counter:

```go
package main

import (
    "fmt"
    "os"
    "testing"
)

var counter int

func TestMain(m *testing.M) {
    // Setup
    fmt.Println("=== MULAI TESTING ===")
    counter = 100  // inisialisasi counter

    // Jalankan test
    exitCode := m.Run()

    // Teardown
    fmt.Println("=== SELESAI TESTING ===")
    fmt.Printf("Counter akhir: %d\n", counter)

    os.Exit(exitCode)
}

func TestTambahCounter(t *testing.T) {
    counter++
    assert.Equal(t, 101, counter)
}

func TestKurangCounter(t *testing.T) {
    counter--
    assert.Equal(t, 100, counter)
}
```

Output:
```
=== MULAI TESTING ===
=== RUN   TestTambahCounter
--- PASS: TestTambahCounter (0.00s)
=== RUN   TestKurangCounter
--- PASS: TestKurangCounter (0.00s)
=== SELESAI TESTING ===
Counter akhir: 100
PASS
```

---

## Urutan Eksekusi

```
TestMain dipanggil
    |
    ├── [SETUP] kode sebelum m.Run()
    |
    ├── m.Run() dipanggil
    |       |
    |       ├── Test1 berjalan
    |       ├── Test2 berjalan
    |       ├── Test3 berjalan
    |       └── ... semua test
    |
    ├── [TEARDOWN] kode setelah m.Run()
    |
    └── os.Exit(exitCode)
```

---

## Analogi: Pertandingan Olahraga

**TestMain** adalah panitia pertandingan:

1. **Sebelum pertandingan (setup):**
   - Siapkan lapangan
   - Pasang gawang
   - Siapkan wasit

2. **`m.Run()` = Pertandingan berlangsung:**
   - Semua atlet (test) bertanding

3. **Setelah pertandingan (teardown):**
   - Bersihkan lapangan
   - Simpan peralatan
   - Buat laporan hasil

Setiap atlet (test) tidak perlu khawatir tentang lapangan — panitia yang mengurus.

---

## Kesalahan Umum Pemula

1. **Lupa `m.Run()`** — tidak ada test yang berjalan, output: `PASS` tapi semua test dilewati (ini berbahaya!)

2. **Lupa `os.Exit(exitCode)`** — exit code selalu 0 meski ada test yang gagal (CI/CD akan mengira semua berhasil)

3. **`TestMain` lebih dari satu** — hanya boleh ada satu `TestMain` per package

4. **Menaruh logic test di `TestMain`** — `TestMain` hanya untuk setup/teardown, bukan untuk test itu sendiri

---

## Latihan

**Tugas:**

Buat `TestMain` yang:
1. Sebelum test: cetak pesan "Memulai test suite kalkulator"
2. Jalankan semua test
3. Setelah test: cetak pesan "Test suite selesai"

Tambahkan juga dua test sederhana: `TestPenjumlahan` dan `TestPengurangan`.

---

### Jawaban

```go
package main

import (
    "fmt"
    "os"
    "testing"
    "github.com/stretchr/testify/assert"
)

func TestMain(m *testing.M) {
    fmt.Println("Memulai test suite kalkulator")
    
    exitCode := m.Run()
    
    fmt.Println("Test suite selesai")
    os.Exit(exitCode)
}

func TestPenjumlahan(t *testing.T) {
    assert.Equal(t, 7, Tambah(3, 4))
}

func TestPengurangan(t *testing.T) {
    assert.Equal(t, 3, Kurang(7, 4))
}
```

---

## Selanjutnya

Lanjut ke: [09 — Sub Test](09-sub-test.md)

Di sana kita belajar cara membuat test di dalam test — berguna untuk mengelompokkan test yang berkaitan.
