# 15 — Table Benchmark

> Video: 01:40:52 — Selesai

---

## Apa Itu Table Benchmark?

**Table Benchmark** adalah gabungan dari dua konsep yang sudah kamu pelajari:

- **Table Test** — data kasus uji diletakkan dalam tabel (slice of struct)
- **Sub Benchmark** — benchmark di dalam benchmark dengan `b.Run`

Hasilnya: cara yang paling rapi dan terorganisir untuk membenchmark banyak kasus sekaligus.

---

## Perbandingan Cara-Cara Benchmark

### Cara 1: Benchmark Terpisah (tidak efisien)

```go
func BenchmarkGabung10(b *testing.B) { ... }
func BenchmarkGabung100(b *testing.B) { ... }
func BenchmarkGabung1000(b *testing.B) { ... }
```

Banyak duplikasi, susah dikelola.

### Cara 2: Sub Benchmark Manual

```go
func BenchmarkGabung(b *testing.B) {
    b.Run("10", func(b *testing.B) { ... })
    b.Run("100", func(b *testing.B) { ... })
    b.Run("1000", func(b *testing.B) { ... })
}
```

Lebih baik, tapi masih ada duplikasi.

### Cara 3: Table Benchmark ✓ (paling rapi)

```go
func BenchmarkGabung(b *testing.B) {
    kasusUji := []struct {
        nama  string
        n     int
    }{
        {"10_kata", 10},
        {"100_kata", 100},
        {"1000_kata", 1000},
    }

    for _, kasus := range kasusUji {
        b.Run(kasus.nama, func(b *testing.B) {
            kata := buatSliceKata(kasus.n)
            b.ResetTimer()
            for i := 0; i < b.N; i++ {
                GabungString(kata)
            }
        })
    }
}
```

---

## Contoh Lengkap: Table Benchmark

```go
package main

import (
    "fmt"
    "strings"
    "testing"
)

// Dua implementasi yang ingin dibandingkan
func GabungDenganPlus(kata []string) string {
    hasil := ""
    for _, k := range kata {
        hasil += k
    }
    return hasil
}

func GabungDenganBuilder(kata []string) string {
    var sb strings.Builder
    for _, k := range kata {
        sb.WriteString(k)
    }
    return sb.String()
}

// Helper
func buatSliceKata(n int) []string {
    kata := make([]string, n)
    for i := range kata {
        kata[i] = fmt.Sprintf("kata%d", i)
    }
    return kata
}

// Table Benchmark
func BenchmarkGabungString(b *testing.B) {
    kasusUji := []struct {
        nama  string
        n     int
        fungsi func([]string) string
    }{
        {"plus/10", 10, GabungDenganPlus},
        {"plus/100", 100, GabungDenganPlus},
        {"plus/1000", 1000, GabungDenganPlus},
        {"builder/10", 10, GabungDenganBuilder},
        {"builder/100", 100, GabungDenganBuilder},
        {"builder/1000", 1000, GabungDenganBuilder},
    }

    for _, kasus := range kasusUji {
        b.Run(kasus.nama, func(b *testing.B) {
            // Setup di luar loop b.N
            data := buatSliceKata(kasus.n)
            b.ResetTimer()

            for i := 0; i < b.N; i++ {
                kasus.fungsi(data)
            }
        })
    }
}
```

Output:
```
BenchmarkGabungString/plus/10-8        2000000    650 ns/op    400 B/op    9 allocs/op
BenchmarkGabungString/plus/100-8        100000   12000 ns/op  28000 B/op   99 allocs/op
BenchmarkGabungString/plus/1000-8         5000  270000 ns/op 2800000 B/op  999 allocs/op
BenchmarkGabungString/builder/10-8     5000000    280 ns/op    128 B/op    2 allocs/op
BenchmarkGabungString/builder/100-8     800000   1800 ns/op    896 B/op    2 allocs/op
BenchmarkGabungString/builder/1000-8     80000  18000 ns/op   8192 B/op    2 allocs/op
```

### Membaca Output

Bandingkan `plus/1000` vs `builder/1000`:
- `plus` butuh 270.000 ns, `builder` hanya 18.000 ns → **builder 15x lebih cepat**
- `plus` alokasikan 2.8 MB, `builder` hanya 8 KB → **builder 342x lebih hemat memori**
- `plus` lakukan 999 alokasi, `builder` hanya 2 → **builder jauh lebih efisien**

Data ini sangat berharga untuk mengambil keputusan teknis.

---

## Pola Table Benchmark yang Umum

```go
func BenchmarkNamaFungsi(b *testing.B) {
    tests := []struct {
        name  string
        // parameter-parameter lain yang bervariasi
    }{
        { name: "kasus kecil", ... },
        { name: "kasus sedang", ... },
        { name: "kasus besar", ... },
    }

    for _, tt := range tests {
        b.Run(tt.name, func(b *testing.B) {
            // Setup yang tidak ingin diukur
            setup := ...
            b.ResetTimer()

            // Loop benchmark yang sesungguhnya
            for i := 0; i < b.N; i++ {
                NamaFungsi(setup)
            }
        })
    }
}
```

---

## Menjalankan Subset Table Benchmark

```bash
# Semua benchmark
go test -bench=.

# Hanya BenchmarkGabungString
go test -bench=BenchmarkGabungString

# Hanya kasus "builder"
go test -bench="BenchmarkGabungString/builder"

# Hanya kasus "1000"
go test -bench="BenchmarkGabungString/.*/1000"

# Dengan info memori
go test -bench=. -benchmem
```

---

## Menjalankan Benchmark Lebih Lama untuk Hasil Lebih Akurat

```bash
# Jalankan setiap benchmark minimal 5 detik (default 1 detik)
go test -bench=. -benchtime=5s

# Jalankan tepat 1000 iterasi
go test -bench=. -benchtime=1000x
```

Semakin lama benchmark berjalan, hasilnya semakin stabil dan akurat.

---

## Kombinasi Unit Test + Table Benchmark

File test yang baik punya keduanya:

```go
package main

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

// === UNIT TEST ===

func TestGabungString(t *testing.T) {
    tests := []struct {
        name     string
        input    []string
        expected string
    }{
        {"dua kata", []string{"hello", "world"}, "helloworld"},
        {"satu kata", []string{"go"}, "go"},
        {"kosong", []string{}, ""},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            assert.Equal(t, tt.expected, GabungDenganBuilder(tt.input))
        })
    }
}

// === BENCHMARK ===

func BenchmarkGabungString(b *testing.B) {
    tests := []struct {
        name string
        n    int
    }{
        {"10_kata", 10},
        {"100_kata", 100},
        {"1000_kata", 1000},
    }

    for _, tt := range tests {
        b.Run(tt.name, func(b *testing.B) {
            data := buatSliceKata(tt.n)
            b.ResetTimer()
            for i := 0; i < b.N; i++ {
                GabungDenganBuilder(data)
            }
        })
    }
}
```

Ini adalah pola ideal: test membuktikan **kebenaran**, benchmark mengukur **kecepatan**.

---

## Cara Berpikir: Kapan Pakai Tiap Pola

| Situasi | Rekomendasi |
|---|---|
| Satu fungsi, satu kasus | Benchmark biasa |
| Satu fungsi, banyak ukuran input | Sub Benchmark atau Table Benchmark |
| Beberapa fungsi berbeda, banyak kasus | Table Benchmark |
| Bandingkan dua implementasi | Table Benchmark dengan fungsi sebagai field |

---

## Analogi: Laporan Uji Performa Mobil

Laporan benchmark yang baik seperti laporan uji performa mobil:

| Model | Kecepatan 0-100 km/h | Konsumsi BBM | Kapasitas Kargo |
|---|---|---|---|
| Mobil A | 8 detik | 15 L/100km | 500 kg |
| Mobil B | 6 detik | 18 L/100km | 300 kg |
| Mobil C | 10 detik | 10 L/100km | 800 kg |

Table Benchmark memberikan laporan seperti ini — perbandingan yang jelas dan terstruktur sehingga kamu bisa membuat keputusan yang tepat.

---

## Kesalahan Umum Pemula

1. **Benchmark tanpa unit test** — pastikan kode benar dulu sebelum mengukur kecepatannya
2. **Terlalu fokus pada angka kecil** — optimasi dari 10ns ke 8ns di kode yang jarang dipanggil tidak berarti banyak
3. **Tidak pakai `b.ResetTimer()`** — setup yang berat akan merusak hasil benchmark
4. **Membandingkan benchmark di mesin berbeda** — selalu bandingkan di kondisi yang sama
5. **Tidak jalankan beberapa kali** — jalankan benchmark 2-3 kali untuk memastikan hasilnya konsisten

---

## Ringkasan Perjalanan Belajar

Selamat! Kamu sudah menyelesaikan seluruh materi. Ini yang sudah kamu pelajari:

| Topik | Kemampuan |
|---|---|
| Software Testing | Memahami apa, kenapa, dan bagaimana testing |
| Unit Test | Membuat test untuk setiap fungsi |
| Menggagalkan Test | Menggunakan t.Error, t.Fatal dengan tepat |
| Assertion | Memakai testify untuk assertion yang rapi |
| Skip Test | Melewati test yang belum siap |
| Before/After | Menyiapkan dan membersihkan resource test |
| Sub Test | Mengelompokkan test dengan t.Run |
| Table Test | Testing banyak kasus secara efisien |
| Mock | Mengisolasi test dari dependensi eksternal |
| Benchmark | Mengukur kecepatan kode |
| Sub Benchmark | Benchmark terorganisir dengan b.Run |
| Table Benchmark | Benchmark banyak kasus secara efisien |

---

## Langkah Selanjutnya

Setelah menguasai semua materi ini, langkah berikutnya:

1. **Terapkan di project nyata** — mulai dengan menulis test untuk satu fungsi di project yang sudah ada
2. **Pelajari coverage** — jalankan `go test -cover` untuk tahu berapa persen kode yang sudah ditest
3. **Pelajari `go generate` + mockery** — tool untuk generate mock secara otomatis dari interface
4. **Pelajari CI/CD** — integrasikan `go test` ke GitHub Actions agar test berjalan otomatis setiap ada commit

---

## Latihan Akhir

**Tugas:**

Buat dua fungsi pencarian:

```go
// Linear search - O(n)
func CariLinear(slice []int, target int) int { ... }

// Binary search - O(log n), hanya untuk slice yang sudah diurutkan
func CariBiner(slice []int, target int) int { ... }
```

Buat:
1. Table Test untuk memverifikasi kedua fungsi benar
2. Table Benchmark untuk membandingkan keduanya dengan input 100, 1000, dan 10000 elemen

---

### Jawaban

```go
// Fungsi
func CariLinear(slice []int, target int) int {
    for i, v := range slice {
        if v == target {
            return i
        }
    }
    return -1
}

func CariBiner(slice []int, target int) int {
    kiri, kanan := 0, len(slice)-1
    for kiri <= kanan {
        tengah := (kiri + kanan) / 2
        if slice[tengah] == target {
            return tengah
        } else if slice[tengah] < target {
            kiri = tengah + 1
        } else {
            kanan = tengah - 1
        }
    }
    return -1
}

// Helper
func buatSliceTerurut(n int) []int {
    s := make([]int, n)
    for i := range s {
        s[i] = i * 2 // 0, 2, 4, 6, ...
    }
    return s
}

// Unit Test
func TestCariLinear(t *testing.T) {
    slice := []int{10, 20, 30, 40, 50}
    tests := []struct {
        name     string
        target   int
        expected int
    }{
        {"ada di tengah", 30, 2},
        {"ada di awal", 10, 0},
        {"ada di akhir", 50, 4},
        {"tidak ada", 99, -1},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            assert.Equal(t, tt.expected, CariLinear(slice, tt.target))
        })
    }
}

func TestCariBiner(t *testing.T) {
    slice := []int{10, 20, 30, 40, 50}
    tests := []struct {
        name     string
        target   int
        expected int
    }{
        {"ada di tengah", 30, 2},
        {"ada di awal", 10, 0},
        {"ada di akhir", 50, 4},
        {"tidak ada", 99, -1},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            assert.Equal(t, tt.expected, CariBiner(slice, tt.target))
        })
    }
}

// Table Benchmark
func BenchmarkCari(b *testing.B) {
    tests := []struct {
        name   string
        n      int
        fungsi func([]int, int) int
    }{
        {"linear/100", 100, CariLinear},
        {"linear/1000", 1000, CariLinear},
        {"linear/10000", 10000, CariLinear},
        {"biner/100", 100, CariBiner},
        {"biner/1000", 1000, CariBiner},
        {"biner/10000", 10000, CariBiner},
    }

    for _, tt := range tests {
        b.Run(tt.name, func(b *testing.B) {
            slice := buatSliceTerurut(tt.n)
            target := slice[tt.n-1] // cari elemen terakhir (kasus terburuk)
            b.ResetTimer()

            for i := 0; i < b.N; i++ {
                tt.fungsi(slice, target)
            }
        })
    }
}
```

Benchmark akan menunjukkan bahwa binary search jauh lebih cepat pada input besar — itulah keindahan O(log n) vs O(n).

---

## Selanjutnya

Kamu sudah selesai! Lihat [KAMUS-ISTILAH.md](KAMUS-ISTILAH.md) sebagai referensi kapan pun kamu lupa arti suatu istilah.
