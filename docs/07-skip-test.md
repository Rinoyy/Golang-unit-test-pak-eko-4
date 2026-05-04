# 07 — Skip Test

> Video: 00:43:24 — 00:46:35

---

## Apa Itu Skip Test?

**Skip** artinya **melewati** test — test tidak dijalankan, tidak dianggap gagal, tidak dianggap berhasil.

Test yang di-skip akan muncul dengan status **SKIP** di output.

---

## Kenapa Perlu Skip Test?

Ada situasi di mana kamu ingin test ada di kode, tapi belum siap untuk dijalankan:

### Situasi 1: Fitur belum selesai dibuat

Kamu menulis test untuk fitur yang belum diimplementasi. Kalau dijalankan sekarang, pasti gagal — tapi bukan karena bug, tapi karena memang belum dibuat.

### Situasi 2: Test bergantung ke resource yang tidak selalu ada

Contoh: test yang memerlukan koneksi ke database production. Di komputer lokal tidak ada akses, tapi di server CI ada.

### Situasi 3: Test sementara tidak relevan

Misalnya ada fitur yang sedang direfactor total — test lamanya tidak relevan sementara, tapi kamu tidak mau hapus.

### Situasi 4: Test hanya untuk kondisi tertentu

Test yang hanya boleh dijalankan di sistem operasi tertentu (Linux, Windows, Mac).

---

## Cara Skip Test

### Cara 1: `t.Skip("alasan")`

```go
func TestFiturBaru(t *testing.T) {
    t.Skip("Fitur ini belum diimplementasi")
    
    // kode di bawah ini tidak akan dieksekusi
    hasil := FiturBaru()
    assert.Equal(t, "ok", hasil)
}
```

Output:
```
=== RUN   TestFiturBaru
    calculator_test.go:6: Fitur ini belum diimplementasi
--- SKIP: TestFiturBaru (0.00s)
PASS
```

Perhatikan: test berstatus `SKIP`, bukan `FAIL`. Program tetap exit dengan sukses.

---

### Cara 2: `t.Skipf("format", args...)` — dengan format pesan

```go
func TestKoneksiDatabase(t *testing.T) {
    lingkungan := os.Getenv("TEST_ENV")
    if lingkungan != "production" {
        t.Skipf("Test ini hanya dijalankan di production, sekarang: %s", lingkungan)
    }
    
    // test berjalan hanya jika TEST_ENV = "production"
}
```

---

### Cara 3: Skip berdasarkan kondisi (paling umum dipakai)

```go
import (
    "os"
    "runtime"
    "testing"
)

func TestKhususLinux(t *testing.T) {
    if runtime.GOOS != "linux" {
        t.Skip("Test ini hanya untuk Linux")
    }
    
    // kode yang hanya berjalan di Linux
}
```

**Istilah: `runtime.GOOS`**

Artinya:
Variabel bawaan Go yang berisi nama sistem operasi yang sedang berjalan: "linux", "darwin" (Mac), "windows".

---

## Perbedaan Skip vs Komentar Kode

Mungkin kamu berpikir: "Kenapa tidak hapus saja testnya atau comment saja?"

| Cara | Masalah |
|---|---|
| Hapus test | Kamu kehilangan test itu selamanya |
| Comment kode | Go tidak tahu test itu ada — tidak muncul di laporan |
| `t.Skip` | Test tetap ada, muncul di laporan, bisa diaktifkan kapan saja |

**`t.Skip` adalah cara yang transparan** — semua orang di tim tahu ada test yang di-skip dan alasannya.

---

## Contoh Nyata: Skip Berdasarkan Environment Variable

```go
func TestKirimEmail(t *testing.T) {
    // Ambil konfigurasi dari environment variable
    smtpHost := os.Getenv("SMTP_HOST")
    
    if smtpHost == "" {
        t.Skip("SMTP_HOST tidak diset, skip test pengiriman email")
    }
    
    // Test hanya berjalan kalau SMTP_HOST sudah diset
    err := KirimEmail("test@example.com", "Subject", "Isi email")
    assert.NoError(t, err)
}
```

**Cara menjalankan dengan environment variable:**
```bash
SMTP_HOST=smtp.gmail.com go test -v
```

**Cara menjalankan tanpa (skip otomatis):**
```bash
go test -v
```

---

## Contoh Nyata: Skip Karena Fitur Belum Selesai

```go
func TestFiturPremium(t *testing.T) {
    t.Skip("TODO: Implementasi fitur premium di sprint berikutnya")
    
    hasil := AktifkanFiturPremium("user123")
    assert.True(t, hasil.Aktif)
    assert.Equal(t, "premium", hasil.Tipe)
}
```

Test ini akan selalu di-skip sampai kamu hapus baris `t.Skip`.

---

## Melihat Test yang Di-skip

Jalankan dengan flag `-v`:

```bash
go test -v
```

```
=== RUN   TestTambah
--- PASS: TestTambah (0.00s)
=== RUN   TestFiturBaru
    calculator_test.go:6: Fitur ini belum diimplementasi
--- SKIP: TestFiturBaru (0.00s)
=== RUN   TestKurang
--- PASS: TestKurang (0.00s)
PASS
```

Test yang di-skip terlihat jelas — tidak tersembunyi, tidak menghalangi test lain.

---

## Analogi: Melewati Pos di Marathon

Bayangkan lomba lari marathon. Di tengah jalan ada **pos pemeriksaan** (checkpoint).

- Kalau pos normal → kamu lewati dan lanjut berlari
- Kalau pos sedang direnovasi → panitia berikan tanda "SKIP pos ini, lanjut ke pos berikutnya"

Kamu tidak dinyatakan kalah karena melewati pos yang sedang renovasi. Kamu tetap melanjutkan lomba.

Skip test persis seperti itu — test yang tidak siap dilewati sementara, tapi lomba (proses testing) tetap berjalan.

---

## Aturan Penting tentang Skip

1. **Selalu tulis alasan** — `t.Skip("alasan kenapa dilewati")` agar orang lain (atau dirimu sendiri di masa depan) tahu
2. **Skip adalah sementara** — jangan biarkan skip menumpuk. Segera aktifkan kembali setelah kondisinya siap
3. **Skip bukan untuk menyembunyikan bug** — kalau kode ada bug, perbaiki — jangan di-skip

---

## Kesalahan Umum Pemula

1. **Lupa tulis alasan** — `t.Skip()` tanpa pesan membuat orang bingung kenapa dilewati
2. **Pakai skip untuk sembunyikan bug** — ini berbahaya, bug tetap ada dan bisa muncul kapan saja
3. **Skip yang tidak pernah diaktifkan** — test menumpuk dalam kondisi skip berbulan-bulan

---

## Latihan

**Tugas:**

Buat sebuah test `TestKoneksiBD` yang:
1. Cek apakah environment variable `DB_HOST` sudah diset
2. Kalau belum diset → skip dengan pesan yang jelas
3. Kalau sudah diset → jalankan test (untuk latihan ini, test isinya bisa `assert.NotEmpty(t, dbHost)`)

---

### Jawaban

```go
func TestKoneksiBD(t *testing.T) {
    dbHost := os.Getenv("DB_HOST")
    
    if dbHost == "" {
        t.Skip("DB_HOST tidak diset, skip test koneksi database")
    }
    
    // Test ini hanya berjalan kalau DB_HOST sudah diset
    assert.NotEmpty(t, dbHost, "DB_HOST harus diisi")
}
```

Jalankan tanpa environment variable → SKIP.
Jalankan dengan `DB_HOST=localhost go test -v` → PASS.

---

## Selanjutnya

Lanjut ke: [08 — Before dan After Test](08-before-after-test.md)

Di sana kita belajar cara menyiapkan (setup) dan membersihkan (teardown) sebelum dan sesudah test berjalan.
