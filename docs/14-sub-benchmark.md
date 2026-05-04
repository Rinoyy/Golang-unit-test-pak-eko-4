# 14 — Sub Benchmark

> Video: 01:38:19 — 01:40:52

---

## Apa Itu Sub Benchmark?

Sama seperti **Sub Test** yang menggunakan `t.Run(...)`, **Sub Benchmark** menggunakan `b.Run(...)` untuk membuat benchmark di dalam benchmark.

Ini memungkinkan kamu mengelompokkan benchmark yang berkaitan dan menjalankan masing-masing secara terpisah.

---

## Kenapa Perlu Sub Benchmark?

Bayangkan kamu mau benchmark fungsi `Tambah` dengan berbagai ukuran input:

**Tanpa sub benchmark:**
```go
func BenchmarkTambahKecil(b *testing.B) { ... }
func BenchmarkTambahSedang(b *testing.B) { ... }
func BenchmarkTambahBesar(b *testing.B) { ... }
```

Ini terpisah-pisah dan tidak jelas hubungannya.

**Dengan sub benchmark:**
```go
func BenchmarkTambah(b *testing.B) {
    b.Run("kecil", func(b *testing.B) { ... })
    b.Run("sedang", func(b *testing.B) { ... })
    b.Run("besar", func(b *testing.B) { ... })
}
```

Lebih terorganisir dan jelas bahwa ketiganya menguji hal yang sama dengan kondisi berbeda.

---

## Cara Membuat Sub Benchmark

```go
func BenchmarkGabungString(b *testing.B) {

    b.Run("10 kata", func(b *testing.B) {
        kata := buatSliceKata(10)
        for i := 0; i < b.N; i++ {
            GabungString(kata)
        }
    })

    b.Run("100 kata", func(b *testing.B) {
        kata := buatSliceKata(100)
        for i := 0; i < b.N; i++ {
            GabungString(kata)
        }
    })

    b.Run("1000 kata", func(b *testing.B) {
        kata := buatSliceKata(1000)
        for i := 0; i < b.N; i++ {
            GabungString(kata)
        }
    })

}
```

### Penjelasan:

```go
b.Run("10 kata", func(b *testing.B) {
```
- `"10 kata"` → nama sub benchmark
- `func(b *testing.B)` → fungsi anonim yang berisi benchmark
- `b` di dalam sub benchmark adalah `b` yang baru — parameter lokal, bukan yang di luar

---

## Output Sub Benchmark

```bash
go test -bench=BenchmarkGabungString -benchmem
```

```
BenchmarkGabungString/10_kata-8      5000000    280 ns/op    160 B/op    1 allocs/op
BenchmarkGabungString/100_kata-8      500000   2800 ns/op   1600 B/op    1 allocs/op
BenchmarkGabungString/1000_kata-8      50000  28000 ns/op  16000 B/op    1 allocs/op
```

Nama sub benchmark muncul sebagai `BenchmarkGabungString/10_kata`.

Dari output di atas kita bisa lihat:
- Input 10x lebih besar → waktu juga ~10x lebih lama (linear)
- Ini adalah tanda implementasi yang "baik" — kompleksitas O(n)

---

## Menjalankan Sub Benchmark Tertentu

```bash
# Jalankan semua sub benchmark di BenchmarkGabungString
go test -bench=BenchmarkGabungString

# Jalankan hanya sub benchmark "100 kata"
go test -bench="BenchmarkGabungString/100_kata"

# Jalankan semua yang mengandung "kata"
go test -bench="BenchmarkGabungString/kata"
```

---

## Contoh Nyata: Benchmark dengan Setup per Sub

Keunggulan sub benchmark: setiap sub bisa punya setup sendiri.

```go
func buatSliceKata(n int) []string {
    kata := make([]string, n)
    for i := range kata {
        kata[i] = fmt.Sprintf("kata%d", i)
    }
    return kata
}

func BenchmarkGabungStringBuilder(b *testing.B) {

    ukuran := []int{10, 100, 1000, 10000}

    for _, n := range ukuran {
        // Buat nama dari ukuran
        nama := fmt.Sprintf("%d_kata", n)

        b.Run(nama, func(b *testing.B) {
            // Setup: buat data di luar loop b.N
            kata := buatSliceKata(n)

            b.ResetTimer() // jangan hitung waktu setup

            for i := 0; i < b.N; i++ {
                GabungStringB(kata)
            }
        })
    }
}
```

Output:
```
BenchmarkGabungStringBuilder/10_kata-8     10000000    120 ns/op
BenchmarkGabungStringBuilder/100_kata-8     1000000   1200 ns/op
BenchmarkGabungStringBuilder/1000_kata-8     100000  12000 ns/op
BenchmarkGabungStringBuilder/10000_kata-8     10000 120000 ns/op
```

Pola ini sangat berguna untuk memahami bagaimana performa berubah seiring bertambahnya ukuran data.

---

## Perbandingan Dua Implementasi dengan Sub Benchmark

```go
func BenchmarkGabungStringPerbandingan(b *testing.B) {

    kata := buatSliceKata(100)

    b.Run("concatenation", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            GabungStringA(kata)  // pakai +
        }
    })

    b.Run("strings_builder", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            GabungStringB(kata)  // pakai strings.Builder
        }
    })

}
```

Output:
```
BenchmarkGabungStringPerbandingan/concatenation-8     500000   2500 ns/op   5000 B/op   99 allocs/op
BenchmarkGabungStringPerbandingan/strings_builder-8  2000000    600 ns/op    512 B/op    2 allocs/op
```

Sekarang perbandingannya sangat jelas dalam satu tempat.

---

## `b.ResetTimer()` di dalam Sub Benchmark

Kalau ada setup di dalam sub benchmark yang tidak ingin dihitung:

```go
b.Run("dengan data besar", func(b *testing.B) {
    // Setup yang berat — tidak ingin dihitung
    data := muatDataDariFile("data_besar.json")

    // Reset timer setelah setup
    b.ResetTimer()

    for i := 0; i < b.N; i++ {
        Proses(data)
    }
})
```

---

## Cara Berpikir: Sub Benchmark untuk Memahami Scaling

Sub benchmark sangat bagus untuk menjawab pertanyaan:

> "Bagaimana performa berubah seiring bertambahnya input?"

Dengan benchmark berbagai ukuran input, kamu bisa melihat apakah algoritmamu:
- **O(1)** — waktu konstan, tidak peduli ukuran input
- **O(n)** — waktu linear, 10x input → 10x lebih lambat
- **O(n²)** — waktu kuadratik, 10x input → 100x lebih lambat (berbahaya!)

---

## Analogi: Uji Kecepatan Mobil di Berbagai Kondisi

Sub benchmark seperti uji kecepatan mobil dalam kondisi berbeda:

```
BenchmarkMobil/jalan_lurus      → 200 km/jam
BenchmarkMobil/jalan_berkelok   → 80 km/jam
BenchmarkMobil/tanjakan         → 60 km/jam
BenchmarkMobil/hujan_deras      → 40 km/jam
```

Setiap sub benchmark adalah kondisi yang berbeda. Hasilnya memberikan gambaran lengkap tentang performa di berbagai situasi.

---

## Kesalahan Umum Pemula

1. **Setup berat di dalam loop `b.N`** — data test harus dibuat di luar loop, lalu `b.ResetTimer()` dipanggil

2. **Tidak memakai `b.ResetTimer()`** — waktu setup ikut terhitung, hasil menjadi tidak akurat

3. **Sub benchmark terlalu banyak** — pertahankan yang paling informatif, jangan benchmark setiap kombinasi yang mungkin

---

## Latihan

**Tugas:**

Buat fungsi `CariAngka(slice []int, target int) int` yang mengembalikan index dari `target` di slice, atau `-1` kalau tidak ditemukan (linear search).

Buat sub benchmark dengan ukuran slice berbeda: 100, 1000, dan 10000. Untuk setiap kasus, cari angka yang ada di **akhir** slice (kasus terburuk).

---

### Jawaban

```go
// Fungsi
func CariAngka(slice []int, target int) int {
    for i, v := range slice {
        if v == target {
            return i
        }
    }
    return -1
}

// Helper
func buatSliceAngka(n int) []int {
    s := make([]int, n)
    for i := range s {
        s[i] = i
    }
    return s
}

// Benchmark
func BenchmarkCariAngka(b *testing.B) {
    ukuran := []int{100, 1000, 10000}

    for _, n := range ukuran {
        nama := fmt.Sprintf("ukuran_%d", n)
        b.Run(nama, func(b *testing.B) {
            slice := buatSliceAngka(n)
            target := n - 1  // angka di akhir (kasus terburuk)

            b.ResetTimer()

            for i := 0; i < b.N; i++ {
                CariAngka(slice, target)
            }
        })
    }
}
```

---

## Selanjutnya

Lanjut ke: [15 — Table Benchmark](15-table-benchmark.md)

Materi terakhir! Di sana kita gabungkan konsep Table Test dengan Benchmark.
