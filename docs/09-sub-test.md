# 09 — Sub Test

> Video: 00:51:27 — 00:56:44

---

## Masalah yang Diselesaikan

Bayangkan kamu punya fungsi `Tambah` dan ingin mengujinya dengan banyak kasus berbeda:

```go
func TestTambahKasus1(t *testing.T) { ... }
func TestTambahKasus2(t *testing.T) { ... }
func TestTambahKasus3(t *testing.T) { ... }
func TestTambahKasus4(t *testing.T) { ... }
```

Ini berantakan. Semua fungsi test terpisah-pisah padahal semua menguji hal yang sama.

**Sub Test adalah solusinya** — kamu bisa mengelompokkan semua kasus dalam satu fungsi test.

---

## Apa Itu Sub Test?

**Istilah: Sub Test**

Artinya:
Test yang berada di dalam test lain. Disebut juga "child test" atau "nested test".

Analogi:
Bayangkan sebuah folder "Ujian Matematika" yang berisi banyak soal: "Soal 1", "Soal 2", "Soal 3". Folder adalah test utama, setiap soal adalah sub test.

---

## Cara Membuat Sub Test

Gunakan `t.Run("nama", func(t *testing.T) { ... })`:

```go
func TestTambah(t *testing.T) {

    t.Run("penjumlahan positif", func(t *testing.T) {
        hasil := Tambah(2, 3)
        assert.Equal(t, 5, hasil)
    })

    t.Run("penjumlahan dengan nol", func(t *testing.T) {
        hasil := Tambah(0, 5)
        assert.Equal(t, 5, hasil)
    })

    t.Run("penjumlahan negatif", func(t *testing.T) {
        hasil := Tambah(-2, -3)
        assert.Equal(t, -5, hasil)
    })

}
```

### Penjelasan:

```go
t.Run("penjumlahan positif", func(t *testing.T) {
```
- `"penjumlahan positif"` → nama sub test (bebas kamu tentukan)
- `func(t *testing.T) { ... }` → fungsi anonim yang berisi test

Perhatikan: `t` di dalam sub test adalah `t` yang baru, bukan `t` dari fungsi luar.

---

## Output Sub Test

```bash
go test -v
```

```
=== RUN   TestTambah
=== RUN   TestTambah/penjumlahan_positif
--- PASS: TestTambah/penjumlahan_positif (0.00s)
=== RUN   TestTambah/penjumlahan_dengan_nol
--- PASS: TestTambah/penjumlahan_dengan_nol (0.00s)
=== RUN   TestTambah/penjumlahan_negatif
--- PASS: TestTambah/penjumlahan_negatif (0.00s)
--- PASS: TestTambah (0.00s)
PASS
```

Nama sub test muncul sebagai `TestTambah/penjumlahan_positif` (dengan `/` sebagai pemisah).

---

## Menjalankan Sub Test Tertentu

Kamu bisa jalankan hanya sub test tertentu:

```bash
# Jalankan semua sub test di TestTambah
go test -run TestTambah

# Jalankan hanya sub test "penjumlahan positif"
go test -run "TestTambah/penjumlahan_positif"

# Jalankan semua sub test yang mengandung kata "negatif"
go test -run "TestTambah/negatif"
```

---

## Keunggulan Sub Test

### 1. Kegagalan terisolasi

Kalau satu sub test gagal, sub test lain tetap berjalan:

```go
func TestTambah(t *testing.T) {

    t.Run("kasus 1", func(t *testing.T) {
        assert.Equal(t, 5, Tambah(2, 3))  // BERHASIL
    })

    t.Run("kasus 2", func(t *testing.T) {
        assert.Equal(t, 10, Tambah(3, 3)) // GAGAL (3+3=6, bukan 10)
    })

    t.Run("kasus 3", func(t *testing.T) {
        assert.Equal(t, 0, Tambah(0, 0))  // BERHASIL
    })

}
```

Output:
```
=== RUN   TestTambah/kasus_1
--- PASS: TestTambah/kasus_1 (0.00s)
=== RUN   TestTambah/kasus_2
--- FAIL: TestTambah/kasus_2 (0.00s)
=== RUN   TestTambah/kasus_3
--- PASS: TestTambah/kasus_3 (0.00s)
--- FAIL: TestTambah (0.00s)
```

Kasus 1 dan 3 tetap berjalan meski kasus 2 gagal.

### 2. Setup per grup

Kamu bisa melakukan setup untuk semua sub test sekaligus:

```go
func TestOperasiKalkulator(t *testing.T) {
    // Setup: buat kalkulator
    kalk := BuatKalkulator()

    t.Run("tambah", func(t *testing.T) {
        assert.Equal(t, 5, kalk.Tambah(2, 3))
    })

    t.Run("kurang", func(t *testing.T) {
        assert.Equal(t, 7, kalk.Kurang(10, 3))
    })

    t.Run("kali", func(t *testing.T) {
        assert.Equal(t, 12, kalk.Kali(3, 4))
    })
}
```

`kalk` dibuat sekali, dipakai oleh semua sub test.

---

## Sub Test Bersarang (Nested)

Sub test bisa diisi sub test lagi:

```go
func TestKalkulator(t *testing.T) {

    t.Run("operasi dasar", func(t *testing.T) {

        t.Run("tambah", func(t *testing.T) {
            assert.Equal(t, 5, Tambah(2, 3))
        })

        t.Run("kurang", func(t *testing.T) {
            assert.Equal(t, 7, Kurang(10, 3))
        })

    })

    t.Run("operasi lanjutan", func(t *testing.T) {

        t.Run("pangkat", func(t *testing.T) {
            assert.Equal(t, 8, Pangkat(2, 3))
        })

    })

}
```

Output:
```
TestKalkulator/operasi_dasar/tambah
TestKalkulator/operasi_dasar/kurang
TestKalkulator/operasi_lanjutan/pangkat
```

---

## Contoh Real: Test Fungsi Validasi

```go
func ValidasiEmail(email string) bool {
    // cek apakah email valid
    return strings.Contains(email, "@") && strings.Contains(email, ".")
}

func TestValidasiEmail(t *testing.T) {

    t.Run("email valid", func(t *testing.T) {
        assert.True(t, ValidasiEmail("budi@gmail.com"))
    })

    t.Run("email tanpa @", func(t *testing.T) {
        assert.False(t, ValidasiEmail("budigmail.com"))
    })

    t.Run("email tanpa domain", func(t *testing.T) {
        assert.False(t, ValidasiEmail("budi@"))
    })

    t.Run("email kosong", func(t *testing.T) {
        assert.False(t, ValidasiEmail(""))
    })

}
```

---

## Cara Berpikir: Kapan Pakai Sub Test?

Gunakan sub test ketika:
- Kamu menguji satu fungsi dengan banyak kasus berbeda
- Kamu ingin mengelompokkan test yang berhubungan
- Kamu ingin bisa menjalankan subset dari test dengan mudah

---

## Analogi: Ujian dengan Beberapa Soal

**Tanpa sub test:** Setiap soal adalah ujian terpisah. Kalau soal 2 gagal, kamu tidak tahu soal 3 berhasil atau tidak — ujian berhenti.

**Dengan sub test:** Semua soal dalam satu ujian. Kalau soal 2 gagal, soal 3 tetap dinilai. Kamu mendapat gambaran lengkap: berhasil 8 dari 10 soal.

---

## Kesalahan Umum Pemula

1. **Nama sub test mengandung spasi banyak** — `t.Run("test kasus 1 ini", ...)` → Go akan ubah spasi jadi `_`, tapi lebih baik gunakan nama yang singkat dan jelas

2. **Tidak memanfaatkan kegagalan terisolasi** — kalau pakai `t.Fatal` di luar sub test, semua sub test berhenti. Gunakan `t.Fatal` hanya di dalam sub test jika ingin berhenti di sub test itu saja.

3. **Sub test terlalu dalam** — maksimal 2-3 level. Lebih dari itu jadi susah dibaca.

---

## Latihan

**Tugas:**

Buat fungsi `Absolut(n int) int` yang mengembalikan nilai absolut (positif) dari `n`.

Buat satu fungsi test `TestAbsolut` yang berisi sub test:
1. "angka positif" — input 5 → expected 5
2. "angka negatif" — input -8 → expected 8
3. "angka nol" — input 0 → expected 0

---

### Jawaban

Fungsi:
```go
func Absolut(n int) int {
    if n < 0 {
        return -n
    }
    return n
}
```

Test:
```go
func TestAbsolut(t *testing.T) {

    t.Run("angka positif", func(t *testing.T) {
        assert.Equal(t, 5, Absolut(5))
    })

    t.Run("angka negatif", func(t *testing.T) {
        assert.Equal(t, 8, Absolut(-8))
    })

    t.Run("angka nol", func(t *testing.T) {
        assert.Equal(t, 0, Absolut(0))
    })

}
```

---

## Selanjutnya

Lanjut ke: [10 — Table Test](10-table-test.md)

Di sana kita belajar cara yang lebih elegan untuk menguji banyak kasus sekaligus menggunakan tabel data.
