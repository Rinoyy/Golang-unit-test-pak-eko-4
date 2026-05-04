# 06 — Assertion

> Video: 00:33:31 — 00:43:24

---

## Masalah dengan Cara Sebelumnya

Di materi sebelumnya, kita menulis test seperti ini:

```go
if hasil != expected {
    t.Errorf("Tambah(2, 3) = %d, expected %d", hasil, expected)
}
```

Ini bekerja. Tapi bayangkan kalau kamu punya 50 fungsi dengan 200 test — kamu harus menulis `if hasil != expected` berulang-ulang. Itu melelahkan dan membuat kode jadi berantakan.

**Assertion adalah solusinya.**

---

## Apa Itu Assertion?

**Istilah: Assertion**

Artinya:
Pernyataan tegas bahwa "ini HARUS benar". Kalau tidak benar → test gagal secara otomatis.

Analogi:
Seperti kontrak: "Saya jamin hasilnya adalah 5. Kalau bukan 5, saya salah dan harus dilaporkan."

---

## Go Tidak Punya Assertion Bawaan

Berbeda dengan bahasa lain, Go tidak menyertakan assertion di package `testing` bawaan.

Kenapa? Karena tim Go menganggap `if + t.Error` sudah cukup eksplisit.

Tapi komunitas Go membuat library **testify** yang sangat populer untuk assertion.

---

## Mengenal Testify

**Istilah: Testify**

Artinya:
Library (kumpulan kode) pihak ketiga yang menyediakan fungsi-fungsi assertion untuk membuat test lebih mudah dibaca.

GitHub: `github.com/stretchr/testify`

---

## Install Testify

Di terminal, di folder project kamu:

```bash
go get github.com/stretchr/testify
```

Ini akan mengunduh dan mendaftarkan testify sebagai dependensi project kamu.

---

## Cara Pakai Testify

```go
package main

import (
    "testing"
    "github.com/stretchr/testify/assert"  // import sub-package assert
)

func TestTambah(t *testing.T) {
    hasil := Tambah(2, 3)
    assert.Equal(t, 5, hasil, "Tambah(2, 3) harus menghasilkan 5")
}
```

Perhatikan perbedaannya dengan cara lama:

**Cara lama:**
```go
if hasil != expected {
    t.Errorf("Tambah(2, 3) = %d, expected %d", hasil, expected)
}
```

**Cara dengan testify:**
```go
assert.Equal(t, 5, hasil, "Tambah(2, 3) harus menghasilkan 5")
```

Lebih pendek, lebih mudah dibaca!

---

## Penjelasan `assert.Equal`

```go
assert.Equal(t, expected, actual, "pesan opsional")
```

| Parameter | Artinya |
|---|---|
| `t` | Selalu parameter pertama — alat laporan test |
| `expected` | Nilai yang kamu harapkan (yang benar) |
| `actual` | Nilai yang sebenarnya keluar dari fungsi |
| `"pesan"` | Pesan tambahan (opsional) untuk memperjelas |

**Penting:** `expected` selalu di kiri, `actual` di kanan. Ini konvensi yang harus diikuti agar pesan error masuk akal.

Kalau salah urutan:
```
expected: 6   ← ini sebenarnya actual
actual:   5   ← ini sebenarnya expected
```
Pesan errornya jadi membingungkan.

---

## Fungsi-Fungsi Assert yang Sering Dipakai

### `assert.Equal` — nilai harus sama

```go
assert.Equal(t, 5, hasil)
assert.Equal(t, "Budi", nama)
assert.Equal(t, true, aktif)
```

### `assert.NotEqual` — nilai harus berbeda

```go
assert.NotEqual(t, 0, hasil)  // hasil tidak boleh 0
```

### `assert.Nil` — nilai harus nil

```go
assert.Nil(t, err)  // tidak boleh ada error
```

### `assert.NotNil` — nilai tidak boleh nil

```go
assert.NotNil(t, user)  // user harus ada
```

### `assert.True` — kondisi harus true

```go
assert.True(t, hasil > 0)
```

### `assert.False` — kondisi harus false

```go
assert.False(t, aktif)
```

### `assert.NoError` — tidak boleh ada error

```go
assert.NoError(t, err)
```

### `assert.Error` — harus ada error

```go
assert.Error(t, err)
```

---

## Perbedaan `assert` dan `require`

Testify punya dua sub-package: `assert` dan `require`.

**Istilah: `assert`**

Sama seperti `t.Error` — kalau gagal, test ditandai gagal tapi **tetap lanjut**.

**Istilah: `require`**

Sama seperti `t.Fatal` — kalau gagal, test ditandai gagal dan **langsung berhenti**.

```go
import (
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestContoh(t *testing.T) {
    db, err := BukaDatabase()
    require.NoError(t, err)  // kalau gagal → berhenti (tidak ada gunanya lanjut)

    data := db.AmbilData("user_1")
    assert.NotNil(t, data)  // kalau gagal → catat tapi lanjut
    assert.Equal(t, "Budi", data.Nama)
}
```

---

## Contoh Lengkap dengan Testify

```go
package main

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

func TestTambah(t *testing.T) {
    hasil := Tambah(2, 3)
    assert.Equal(t, 5, hasil, "2 + 3 harus 5")
}

func TestKurang(t *testing.T) {
    hasil := Kurang(10, 3)
    assert.Equal(t, 7, hasil, "10 - 3 harus 7")
}

func TestKali(t *testing.T) {
    hasil := Kali(4, 5)
    assert.Equal(t, 20, hasil, "4 x 5 harus 20")
}

func TestBagi(t *testing.T) {
    hasil := Bagi(10, 2)
    assert.Equal(t, 5, hasil, "10 / 2 harus 5")
}

func TestBagiDenganNol(t *testing.T) {
    hasil := Bagi(10, 0)
    assert.Equal(t, 0, hasil, "dibagi 0 harus return 0")
}
```

---

## Pesan Error yang Lebih Informatif

Kalau test gagal, testify menampilkan pesan yang sangat jelas:

```
--- FAIL: TestTambah (0.00s)
    calculator_test.go:9:
        Error Trace: calculator_test.go:9
        Error:       Not equal:
                     expected: 5
                     actual  : 6
        Test:        TestTambah
        Messages:    2 + 3 harus 5
```

Bandingkan dengan cara manual:
```
--- FAIL: TestTambah (0.00s)
    calculator_test.go:9: Tambah(2, 3) = 6, expected 5
```

Testify lebih terstruktur dan mudah dibaca.

---

## Cara Berpikir: Test Behavior, Bukan Implementation

Ketika kamu menulis assertion, pikirkan **behavior** (perilaku) yang kamu inginkan, bukan cara kerjanya.

**Salah (terlalu fokus ke implementasi):**
```go
// Ini mengasumsikan implementasi pakai loop
assert.Equal(t, 3, len(hasil.Items))
```

**Benar (fokus ke behavior):**
```go
// Ini menyatakan behavior: harus ada 3 item
assert.Len(t, hasil.Items, 3)
```

---

## Analogi: Assertion seperti Syarat Kontrak

Bayangkan kamu menyewa jasa desainer:

**Kontrak:**
- Logo harus berwarna biru → `assert.Equal(t, "biru", logo.Warna)`
- Ukuran harus minimal 500x500px → `assert.True(t, logo.Lebar >= 500)`
- File harus berformat PNG → `assert.Equal(t, "png", logo.Format)`

Kalau desainer menyerahkan logo yang warnanya merah, kontrak dilanggar → test gagal.

Assertion adalah kontrak antara kamu dan kode yang kamu tulis.

---

## Kesalahan Umum Pemula

1. **Tukar expected dan actual** — `assert.Equal(t, hasil, 5)` → seharusnya `assert.Equal(t, 5, hasil)`. Expected selalu di kiri.

2. **Lupa import yang benar** — harus import `assert` atau `require`, bukan hanya `testify`

3. **Selalu pakai `assert` padahal perlu `require`** — kalau langkah pertama gagal dan langkah berikutnya bergantung padanya, harusnya pakai `require`

4. **Pesan assertion terlalu umum** — "gagal" tidak membantu. Tulis "hasil penjumlahan harus 5"

---

## Latihan

**Tugas:**

Buat fungsi `RataRata(angka []int) float64` yang menghitung rata-rata dari slice angka.

Buat test menggunakan `assert.Equal` untuk:
1. Input `[2, 4, 6]` → expected `4.0`
2. Input `[10, 20]` → expected `15.0`
3. Input `[5]` → expected `5.0`

---

### Jawaban

Fungsi:
```go
func RataRata(angka []int) float64 {
    if len(angka) == 0 {
        return 0
    }
    total := 0
    for _, a := range angka {
        total += a
    }
    return float64(total) / float64(len(angka))
}
```

Test:
```go
package main

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

func TestRataRata(t *testing.T) {
    assert.Equal(t, 4.0, RataRata([]int{2, 4, 6}), "rata-rata [2,4,6] harus 4.0")
    assert.Equal(t, 15.0, RataRata([]int{10, 20}), "rata-rata [10,20] harus 15.0")
    assert.Equal(t, 5.0, RataRata([]int{5}), "rata-rata [5] harus 5.0")
}
```

---

## Selanjutnya

Lanjut ke: [07 — Skip Test](07-skip-test.md)

Di sana kamu akan belajar cara melewati (skip) test tertentu sementara waktu.
