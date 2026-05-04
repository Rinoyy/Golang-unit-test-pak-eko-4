# 12 — Pengenalan Benchmark

> Video: 01:26:25 — 01:30:23

---

## Apa Itu Benchmark?

Sejauh ini kita belajar tentang **kebenaran kode** — apakah fungsi menghasilkan output yang benar?

**Benchmark** adalah tentang hal yang berbeda: **kecepatan kode** — seberapa cepat fungsi berjalan?

**Istilah: Benchmark**

Artinya:
Proses mengukur performa (kecepatan) kode. Go menjalankan kode berkali-kali dan menghitung rata-rata waktu yang dibutuhkan per operasi.

Analogi:
Kamu membeli mobil baru. Kamu tidak hanya cek apakah mobilnya bisa jalan (unit test), tapi kamu juga ukur berapa waktu yang dibutuhkan untuk mencapai 100 km/jam (benchmark).

---

## Kenapa Perlu Benchmark?

### Situasi 1: Kamu punya dua cara untuk melakukan hal yang sama

```go
// Cara A: pakai loop biasa
func JumlahA(angka []int) int {
    total := 0
    for _, a := range angka {
        total += a
    }
    return total
}

// Cara B: pakai rekursi
func JumlahB(angka []int) int {
    if len(angka) == 0 {
        return 0
    }
    return angka[0] + JumlahB(angka[1:])
}
```

Keduanya benar. Tapi mana yang lebih cepat? Benchmark bisa menjawab.

### Situasi 2: Kamu mau tahu dampak perubahan kode

Sebelum refactor → benchmark. Setelah refactor → benchmark lagi. Bandingkan hasilnya.

### Situasi 3: Aplikasi lambat dan kamu mau tahu di mana

Benchmark bagian-bagian berbeda dari kode untuk menemukan mana yang paling lambat.

---

## Apa yang Diukur Benchmark?

Go mengukur:

1. **Waktu per operasi** (ns/op) — berapa nanosecond per satu kali pemanggilan fungsi
2. **Jumlah alokasi memori per operasi** (allocs/op)
3. **Jumlah byte dialokasikan per operasi** (B/op)

**Istilah: nanosecond (ns)**

Artinya:
Satuan waktu. 1 nanosecond = 0.000000001 detik (satu per satu miliar detik). Sangat kecil!

Analogi perbandingan:
- 1 detik = kamu berkedip sekali
- 1 milidetik = sangat cepat (1/1000 detik)
- 1 mikrodetik = lebih cepat lagi (1/1.000.000 detik)
- 1 nanodetik = sangat sangat cepat (1/1.000.000.000 detik)

Komputer modern sangat cepat — fungsi sederhana bisa selesai dalam hitungan nanodetik.

---

## Perbedaan Test dan Benchmark

| Aspek | Unit Test | Benchmark |
|---|---|---|
| Tujuan | Verifikasi kebenaran | Ukur kecepatan |
| Dijalankan dengan | `go test` | `go test -bench` |
| Nama fungsi | `TestXxx` | `BenchmarkXxx` |
| Parameter | `*testing.T` | `*testing.B` |
| Hasil | PASS / FAIL | ns/op, B/op, allocs/op |

---

## Kapan Sebaiknya Membuat Benchmark?

**Buat benchmark ketika:**
- Ada algoritma yang berat dan performa kritis
- Kamu mau bandingkan dua implementasi
- Kamu perlu membuktikan bahwa perubahan tidak membuat kode lebih lambat
- Aplikasi terasa lambat dan perlu investigasi

**Jangan benchmark setiap fungsi:**
- Benchmark butuh waktu untuk dijalankan
- Terlalu banyak benchmark membuat proses testing lambat
- Fokus pada bagian yang kritis terhadap performa

---

## Prinsip: "Make it work, then make it fast"

Urutan yang benar:

1. **Make it work** — buat kodenya benar dulu (unit test)
2. **Make it right** — buat kodenya rapi dan maintainable
3. **Make it fast** — baru optimasi performa (benchmark)

Jangan optimasi sebelum kodenya benar!

---

## Contoh Kasus Nyata: Optimasi di Dunia Nyata

Sebuah aplikasi e-commerce menerima 10.000 request per detik. Fungsi `HitungDiskon` dipanggil di setiap request.

Kalau `HitungDiskon` butuh **1 ms** per panggilan:
- 10.000 request/detik × 1 ms = 10 detik waktu CPU hanya untuk diskon!

Kalau kita optimasi menjadi **0.01 ms**:
- 10.000 request/detik × 0.01 ms = 0.1 detik
- Penghematan: 99%!

Benchmark membantu menemukan dan mengukur perbedaan seperti ini.

---

## Analogi: Lomba Lari vs Tes Kesehatan

**Unit Test** = tes kesehatan. Memverifikasi bahwa atlet sehat dan tidak ada yang salah dengan tubuhnya.

**Benchmark** = lomba lari. Mengukur seberapa cepat atlet bisa berlari.

Kamu bisa sehat tapi lambat. Kamu bisa cepat tapi tidak sehat. Idealnya: sehat (unit test berhasil) DAN cepat (benchmark bagus).

---

## Ringkasan

| Konsep | Penjelasan |
|---|---|
| Benchmark | Mengukur kecepatan kode |
| ns/op | Nanosecond per satu operasi — satuan utama benchmark |
| allocs/op | Jumlah alokasi memori per operasi |
| B/op | Jumlah byte yang dialokasikan per operasi |
| `go test -bench` | Perintah untuk menjalankan benchmark |

---

## Yang Perlu Diingat

- Benchmark bukan pengganti unit test — keduanya melengkapi
- Jalankan benchmark hanya untuk kode yang performa-kritis
- Selalu benchmark sebelum dan sesudah optimasi untuk membuktikan perubahannya
- Angka benchmark di komputermu bisa berbeda di server — yang penting adalah perbandingan relatif

---

## Selanjutnya

Lanjut ke: [13 — Membuat Benchmark](13-membuat-benchmark.md)

Di sana kita langsung praktek membuat benchmark pertama kamu!
