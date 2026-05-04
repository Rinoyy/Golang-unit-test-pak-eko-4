# 13 — Membuat Benchmark

> Video: 01:30:23 — 01:38:19

---

## Saatnya Praktek Benchmark!

Di materi sebelumnya kita tahu **apa** itu benchmark. Sekarang kita **buat** benchmark pertama kita.

---

## Aturan Membuat Benchmark di Go

Sama seperti unit test, benchmark punya aturan:

1. **File** harus berakhiran `_test.go`
2. **Nama fungsi** harus dimulai dengan `Benchmark` (huruf B kapital)
3. **Parameter** adalah `b *testing.B` (bukan `t *testing.T`)

```go
func BenchmarkNamaFungsi(b *testing.B) {
    // isi benchmark di sini
}
```

---

## Struktur Benchmark

```go
func BenchmarkTambah(b *testing.B) {
    // Kode yang ingin diukur diletakkan di dalam loop b.N
    for i := 0; i < b.N; i++ {
        Tambah(2, 3)
    }
}
```

### Penjelasan Paling Penting:

**Istilah: `b.N`**

Artinya:
Jumlah iterasi yang ditentukan oleh Go secara otomatis. Go akan menentukan `b.N` berdasarkan seberapa lama benchmark perlu berjalan untuk mendapatkan hasil yang akurat.

Analogi:
Bayangkan kamu mengukur kecepatan atlet. Kamu tidak hanya suruh dia lari sekali — kamu suruh dia lari berkali-kali lalu ambil rata-ratanya. `b.N` adalah jumlah "berapa kali lari" yang ditentukan Go.

**Kamu tidak boleh mengubah `b.N`** — biarkan Go yang menentukan nilainya.

---

## Cara Menjalankan Benchmark

```bash
# Jalankan semua benchmark
go test -bench=.

# Jalankan benchmark tertentu
go test -bench=BenchmarkTambah

# Jalankan dengan info memori
go test -bench=. -benchmem
```

**Penjelasan flag:**
- `-bench=.` → jalankan semua benchmark (`.` = semua)
- `-bench=BenchmarkTambah` → hanya benchmark ini
- `-benchmem` → tampilkan info alokasi memori

---

## Contoh Benchmark Pertama

```go
// File: calculator_test.go

func BenchmarkTambah(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Tambah(2, 3)
    }
}
```

Jalankan:
```bash
go test -bench=BenchmarkTambah
```

Output:
```
goos: darwin
goarch: arm64
pkg: belajar_testing
BenchmarkTambah-8    1000000000    0.2801 ns/op
PASS
ok      belajar_testing    0.573s
```

### Membaca Output:

```
BenchmarkTambah-8    1000000000    0.2801 ns/op
```

| Bagian | Artinya |
|---|---|
| `BenchmarkTambah` | Nama benchmark |
| `-8` | Jumlah CPU yang dipakai (8 core) |
| `1000000000` | Go menjalankan 1 miliar iterasi |
| `0.2801 ns/op` | Setiap operasi butuh 0.28 nanosecond |

Angka `0.2801 ns/op` berarti fungsi `Tambah` sangat cepat — masuk akal karena hanya melakukan penjumlahan sederhana.

---

## Benchmark dengan `-benchmem`

```bash
go test -bench=BenchmarkTambah -benchmem
```

Output:
```
BenchmarkTambah-8    1000000000    0.2801 ns/op    0 B/op    0 allocs/op
```

Kolom tambahan:
| Bagian | Artinya |
|---|---|
| `0 B/op` | Tidak ada alokasi memori per operasi |
| `0 allocs/op` | Tidak ada heap allocation per operasi |

Alokasi memori yang rendah itu bagus — artinya kode tidak membebani garbage collector.

---

## Contoh: Bandingkan Dua Implementasi

Misalkan ada dua cara untuk menggabungkan string:

```go
// Cara A: pakai + (concatenation)
func GabungStringA(kata []string) string {
    hasil := ""
    for _, k := range kata {
        hasil += k
    }
    return hasil
}

// Cara B: pakai strings.Builder
func GabungStringB(kata []string) string {
    var sb strings.Builder
    for _, k := range kata {
        sb.WriteString(k)
    }
    return sb.String()
}
```

Benchmark keduanya:

```go
var kataKata = []string{"hello", " ", "world", "!", " ", "ini", " ", "test"}

func BenchmarkGabungStringA(b *testing.B) {
    for i := 0; i < b.N; i++ {
        GabungStringA(kataKata)
    }
}

func BenchmarkGabungStringB(b *testing.B) {
    for i := 0; i < b.N; i++ {
        GabungStringB(kataKata)
    }
}
```

Jalankan:
```bash
go test -bench=. -benchmem
```

Output (contoh):
```
BenchmarkGabungStringA-8    3000000    450 ns/op    248 B/op    7 allocs/op
BenchmarkGabungStringB-8    8000000    180 ns/op     64 B/op    2 allocs/op
```

**Kesimpulan:**
- Cara B (strings.Builder) **2.5x lebih cepat**
- Cara B mengalokasikan memori **4x lebih sedikit**
- Cara B jelas lebih baik untuk string panjang

---

## Setup di Benchmark: ResetTimer

Kadang kamu perlu setup sebelum benchmark, tapi tidak ingin setup dihitung sebagai waktu benchmark:

```go
func BenchmarkProsesBesar(b *testing.B) {
    // Setup yang membutuhkan waktu — tidak ingin dihitung
    data := buatDataBesar(10000)
    
    // Reset timer — waktu setup tidak dihitung
    b.ResetTimer()
    
    // Baru mulai benchmark yang sesungguhnya
    for i := 0; i < b.N; i++ {
        ProsesBesar(data)
    }
}
```

**Istilah: `b.ResetTimer()`**

Artinya:
Reset timer benchmark ke 0. Waktu yang terpakai sebelum `ResetTimer()` tidak dihitung dalam hasil benchmark.

Analogi:
Seperti stopwatch yang di-reset setelah kamu selesai pemanasan. Waktu pemanasan tidak dihitung, hanya waktu lomba yang sebenarnya.

---

## StopTimer dan StartTimer

Kalau kamu perlu melakukan sesuatu di tengah loop yang tidak ingin dihitung:

```go
func BenchmarkDenganPersiapan(b *testing.B) {
    for i := 0; i < b.N; i++ {
        // Hentikan timer saat mempersiapkan data
        b.StopTimer()
        data := buatDataUntukIterasiIni()
        b.StartTimer()
        
        // Baru jalankan kode yang diukur
        Proses(data)
    }
}
```

---

## Contoh Lengkap: Benchmark Kalkulator

```go
// File: calculator_test.go

func BenchmarkTambah(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Tambah(100, 200)
    }
}

func BenchmarkKurang(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Kurang(500, 100)
    }
}

func BenchmarkKali(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Kali(12, 34)
    }
}

func BenchmarkBagi(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Bagi(1000, 7)
    }
}
```

Jalankan semua:
```bash
go test -bench=. -benchmem -v
```

---

## Cara Berpikir: Apa yang Diukur?

Sebelum membuat benchmark, tanyakan:

**Apa yang diukur?**
→ Hanya kode di dalam loop `b.N`. Pastikan itu adalah kode yang kamu ingin ukur, bukan setup.

**Kenapa diukur?**
→ Kode ini dipanggil sering? Ada kandidat yang lebih cepat?

**Apa yang dioptimasi?**
→ Waktu (ns/op) atau memori (B/op)? Kadang keduanya trade-off.

---

## Analogi: Mencatat Rekor Lari

Benchmark seperti pencatat waktu di lomba lari:

- `b.N` = jumlah putaran yang harus dilari
- Waktu per operasi = waktu per putaran
- `b.ResetTimer()` = jangan hitung waktu pemanasan

Tujuannya bukan menang — tujuannya tahu **berapa kecepatan saat ini** agar bisa diperbaiki.

---

## Kesalahan Umum Pemula

1. **Tidak loop dengan `b.N`** — kode yang diukur harus di dalam `for i := 0; i < b.N; i++`, bukan di luar

2. **Menaruh setup yang berat di dalam loop** — ini akan terukur dan menghasilkan angka yang menyesatkan

3. **Lupa `-bench` flag** — `go test` saja tidak menjalankan benchmark!

4. **Mengubah `b.N`** — jangan ubah nilai `b.N`, biarkan Go yang menentukan

5. **Membandingkan benchmark di mesin yang berbeda** — angkanya bisa sangat berbeda. Bandingkan di mesin yang sama.

---

## Latihan

**Tugas:**

Buat dua fungsi untuk menghitung faktorial:

```go
// Cara A: iteratif (loop)
func FaktorialIteratif(n int) int { ... }

// Cara B: rekursif
func FaktorialRekursif(n int) int { ... }
```

Buat benchmark untuk keduanya dan bandingkan hasilnya.

---

### Jawaban

```go
// Fungsi
func FaktorialIteratif(n int) int {
    hasil := 1
    for i := 2; i <= n; i++ {
        hasil *= i
    }
    return hasil
}

func FaktorialRekursif(n int) int {
    if n <= 1 {
        return 1
    }
    return n * FaktorialRekursif(n-1)
}

// Benchmark
func BenchmarkFaktorialIteratif(b *testing.B) {
    for i := 0; i < b.N; i++ {
        FaktorialIteratif(10)
    }
}

func BenchmarkFaktorialRekursif(b *testing.B) {
    for i := 0; i < b.N; i++ {
        FaktorialRekursif(10)
    }
}
```

Biasanya versi iteratif lebih cepat karena tidak ada overhead pemanggilan fungsi rekursif.

---

## Selanjutnya

Lanjut ke: [14 — Sub Benchmark](14-sub-benchmark.md)

Di sana kita belajar cara membuat benchmark di dalam benchmark, mirip seperti sub test.
