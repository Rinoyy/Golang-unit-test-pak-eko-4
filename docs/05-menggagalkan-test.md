# 05 — Menggagalkan Test

> Video: 00:25:37 — 00:33:31

---

## Apa Artinya "Menggagalkan Test"?

Pada materi sebelumnya, kita sudah tahu bahwa test bisa **BERHASIL** atau **GAGAL**.

Di materi ini, kita fokus pada: **bagaimana cara memberitahu Go bahwa sebuah test gagal**.

Ada beberapa cara — dan masing-masing punya perbedaan penting yang harus kamu pahami.

---

## Kenapa Ini Penting?

Kalau kamu tidak tahu cara yang tepat untuk melaporkan kegagalan, kamu bisa:
- Melaporkan kegagalan tapi test tetap jalan terus (padahal harusnya berhenti)
- Berhenti terlalu cepat dan kehilangan informasi penting
- Mendapat pesan error yang membingungkan

---

## Semua Cara Menggagalkan Test

### 1. `t.Fail()`

**Apa yang dilakukan:**
Menandai test sebagai GAGAL, tapi **tetap lanjut** menjalankan sisa kode dalam fungsi test.

**Tidak ada pesan.**

```go
func TestContoh(t *testing.T) {
    hasil := Tambah(2, 3)
    if hasil != 5 {
        t.Fail()  // tandai gagal, tidak ada pesan, lanjut terus
    }
    fmt.Println("Baris ini tetap dieksekusi")
}
```

Output saat gagal:
```
--- FAIL: TestContoh (0.00s)
FAIL
```

Kekurangan: tidak ada pesan, susah tahu kenapa gagal.

---

### 2. `t.Error("pesan")`

**Apa yang dilakukan:**
Menandai test sebagai GAGAL, menampilkan pesan, lalu **tetap lanjut**.

```go
func TestContoh(t *testing.T) {
    hasil := Tambah(2, 3)
    if hasil != 5 {
        t.Error("Hasil tidak sesuai!")  // tandai gagal + pesan, lanjut terus
    }
    fmt.Println("Baris ini tetap dieksekusi")
}
```

Output saat gagal:
```
--- FAIL: TestContoh (0.00s)
    calculator_test.go:8: Hasil tidak sesuai!
```

---

### 3. `t.Errorf("format", args...)`

**Apa yang dilakukan:**
Sama seperti `t.Error`, tapi bisa format pesan seperti `fmt.Sprintf`.

```go
func TestContoh(t *testing.T) {
    hasil := Tambah(2, 3)
    expected := 5
    if hasil != expected {
        t.Errorf("Tambah(2, 3) = %d, tapi expected %d", hasil, expected)
    }
}
```

Output saat gagal:
```
--- FAIL: TestContoh (0.00s)
    calculator_test.go:8: Tambah(2, 3) = 6, tapi expected 5
```

**Ini yang paling sering dipakai** karena pesannya informatif.

---

### 4. `t.FailNow()`

**Apa yang dilakukan:**
Menandai test sebagai GAGAL, lalu **langsung berhenti**. Tidak ada pesan.

```go
func TestContoh(t *testing.T) {
    hasil := Tambah(2, 3)
    if hasil != 5 {
        t.FailNow()  // tandai gagal, BERHENTI sekarang
    }
    fmt.Println("Baris ini TIDAK dieksekusi kalau di atas gagal")
}
```

---

### 5. `t.Fatal("pesan")`

**Apa yang dilakukan:**
Menandai test sebagai GAGAL, menampilkan pesan, lalu **langsung berhenti**.

```go
func TestContoh(t *testing.T) {
    hasil := Tambah(2, 3)
    if hasil != 5 {
        t.Fatal("Fungsi Tambah rusak!")  // tandai gagal + pesan, BERHENTI
    }
    fmt.Println("Baris ini TIDAK dieksekusi kalau di atas gagal")
}
```

---

### 6. `t.Fatalf("format", args...)`

**Apa yang dilakukan:**
Sama seperti `t.Fatal`, tapi bisa format pesan.

```go
func TestContoh(t *testing.T) {
    hasil := Tambah(2, 3)
    expected := 5
    if hasil != expected {
        t.Fatalf("Tambah(2, 3) = %d, tapi expected %d", hasil, expected)
    }
    fmt.Println("Baris ini TIDAK dieksekusi kalau di atas gagal")
}
```

---

## Ringkasan Perbandingan

| Fungsi | Ada Pesan? | Lanjut Setelah Gagal? |
|---|---|---|
| `t.Fail()` | Tidak | Ya |
| `t.Error("...")` | Ya | Ya |
| `t.Errorf("...")` | Ya (format) | Ya |
| `t.FailNow()` | Tidak | Tidak |
| `t.Fatal("...")` | Ya | Tidak |
| `t.Fatalf("...")` | Ya (format) | Tidak |

**Rekomendasi:**
- Pakai `t.Errorf` jika kamu mau periksa beberapa hal dan tetap ingin lihat semua kegagalannya
- Pakai `t.Fatalf` jika kalau satu hal gagal, tidak perlu lanjut

---

## Kapan Pakai Error vs Fatal?

### Contoh kasus pakai `t.Error` (cek banyak hal):

```go
func TestUser(t *testing.T) {
    user := BuatUser("Budi", 25, "budi@email.com")

    if user.Nama != "Budi" {
        t.Errorf("Nama = %s, expected Budi", user.Nama)
    }
    // meski nama salah, tetap cek umur dan email
    if user.Umur != 25 {
        t.Errorf("Umur = %d, expected 25", user.Umur)
    }
    if user.Email != "budi@email.com" {
        t.Errorf("Email = %s, expected budi@email.com", user.Email)
    }
}
```

Dengan `t.Error`, kalau Nama salah — kamu masih bisa tahu Umur dan Email juga salah atau tidak. Berguna untuk menemukan semua masalah sekaligus.

---

### Contoh kasus pakai `t.Fatal` (tidak ada gunanya lanjut):

```go
func TestBukaDatabaseDanAmbilData(t *testing.T) {
    db, err := BukaDatabase()
    if err != nil {
        t.Fatalf("Gagal buka database: %v", err)
        // kalau database tidak bisa dibuka, tidak perlu lanjut
        // tidak ada gunanya coba ambil data
    }

    // ini hanya dieksekusi kalau database berhasil dibuka
    data := db.AmbilData()
    if data == nil {
        t.Error("Data seharusnya tidak nil")
    }
}
```

---

## Cara Berpikir Memilih Error vs Fatal

Tanya diri sendiri:

> "Kalau langkah ini gagal, apakah langkah berikutnya masih masuk akal untuk dijalankan?"

- **Ya** → pakai `t.Error`
- **Tidak** → pakai `t.Fatal`

---

## Contoh Lengkap: Semua Cara dalam Satu File

```go
package main

import (
    "fmt"
    "testing"
)

// Demonstrasi t.Error — lanjut meski gagal
func TestDemonstrasiError(t *testing.T) {
    hasil := Tambah(2, 3)
    if hasil != 5 {
        t.Errorf("Tambah(2,3) harusnya 5, tapi dapat %d", hasil)
    }
    fmt.Println("Baris ini selalu dieksekusi")
}

// Demonstrasi t.Fatal — berhenti saat gagal
func TestDemonstrasiFatal(t *testing.T) {
    hasil := Tambah(2, 3)
    if hasil != 5 {
        t.Fatalf("Tambah(2,3) harusnya 5, tapi dapat %d", hasil)
    }
    fmt.Println("Baris ini hanya dieksekusi jika test di atas berhasil")
}
```

---

## Percobaan: Lihat Perbedaannya Sendiri

Ubah fungsi `Tambah` menjadi salah:

```go
func Tambah(a int, b int) int {
    return a + b + 1  // bug: ditambah 1
}
```

Coba test dengan `t.Error`:
```go
func TestDenganError(t *testing.T) {
    if Tambah(2, 3) != 5 {
        t.Error("Test 1 gagal")
    }
    if Tambah(1, 1) != 2 {
        t.Error("Test 2 gagal")
    }
    fmt.Println("Sampai akhir fungsi")
}
```

Output:
```
--- FAIL: TestDenganError (0.00s)
    calculator_test.go:7: Test 1 gagal
    calculator_test.go:10: Test 2 gagal
Sampai akhir fungsi
```

**Kamu lihat kedua error dan "Sampai akhir fungsi".**

---

Coba test dengan `t.Fatal`:
```go
func TestDenganFatal(t *testing.T) {
    if Tambah(2, 3) != 5 {
        t.Fatal("Test 1 gagal")
    }
    if Tambah(1, 1) != 2 {
        t.Fatal("Test 2 gagal")  // tidak pernah sampai sini
    }
    fmt.Println("Sampai akhir fungsi")  // tidak pernah sampai sini
}
```

Output:
```
--- FAIL: TestDenganFatal (0.00s)
    calculator_test.go:7: Test 1 gagal
```

**Hanya error pertama yang muncul. Test berhenti di situ.**

---

## Analogi: Petugas Quality Control

Bayangkan kamu adalah petugas QC (Quality Control) di pabrik.

**Pakai `t.Error`** = kamu periksa semua bagian produk satu per satu, catat semua yang rusak, baru setelah semua selesai kamu buat laporan lengkap. Cocok kalau kamu mau tahu semua kerusakan sekaligus.

**Pakai `t.Fatal`** = kamu temukan bagian yang sangat rusak parah. Tidak perlu periksa bagian lain — produk ini harus langsung dibuang. Tidak ada gunanya teruskan pemeriksaan.

---

## Kesalahan Umum Pemula

1. **Selalu pakai `t.Fatal`** — kehilangan informasi karena test berhenti terlalu awal
2. **Pesan yang tidak informatif** — `t.Error("error")` tidak berguna. Tulis apa yang diexpected dan apa yang didapat
3. **Tidak pakai `t.Fatal` sama sekali** — kode setelah kegagalan dieksekusi padahal hasilnya tidak valid
4. **Lupa `return` setelah `t.Error`** — kalau kamu mau berhenti setelah error, `t.Error` + `return` bisa dipakai sebagai alternatif `t.Fatal`

---

## Latihan

**Tugas:**

Buat fungsi `Maksimal(a, b int) int` yang mengembalikan nilai terbesar dari dua angka.

Buat test-nya dengan ketentuan:
1. Test `Maksimal(10, 5)` → expected 10
2. Test `Maksimal(3, 7)` → expected 7
3. Test `Maksimal(4, 4)` → expected 4

Untuk poin 1 dan 2, pakai `t.Errorf`.
Untuk poin 3, kalau gagal berarti ada bug serius → pakai `t.Fatalf`.

---

### Jawaban

Fungsi:
```go
func Maksimal(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

Test:
```go
func TestMaksimal(t *testing.T) {
    hasil := Maksimal(10, 5)
    if hasil != 10 {
        t.Errorf("Maksimal(10, 5) = %d, expected 10", hasil)
    }

    hasil = Maksimal(3, 7)
    if hasil != 7 {
        t.Errorf("Maksimal(3, 7) = %d, expected 7", hasil)
    }

    hasil = Maksimal(4, 4)
    if hasil != 4 {
        t.Fatalf("Maksimal(4, 4) = %d, expected 4 — bug serius!", hasil)
    }
}
```

---

## Selanjutnya

Lanjut ke: [06 — Assertion](06-assertion.md)

Di sana kita belajar cara yang lebih rapi dan elegan untuk membandingkan expected vs actual.
