# Kamus Istilah — Software Testing di Golang

> Referensi cepat. Buka kapan saja kamu menemukan istilah yang tidak dimengerti.

---

## A

---

**Actual**

Artinya:
Nilai yang benar-benar dihasilkan oleh kode saat dijalankan.

Analogi:
Makanan yang benar-benar disajikan ke mejamu (bisa berbeda dari yang kamu pesan).

Contoh:
```go
actual := Tambah(2, 3)  // actual = 5 (dari fungsi)
```

Catatan:
Selalu letakkan `actual` di sebelah kanan di fungsi assert: `assert.Equal(t, expected, actual)`.

---

**AAA (Arrange, Act, Assert)**

Artinya:
Pola tiga langkah dalam menulis unit test:
- **Arrange** — siapkan data dan kondisi
- **Act** — jalankan fungsi yang diuji
- **Assert** — periksa hasilnya

Contoh:
```go
// Arrange
a, b := 2, 3

// Act
hasil := Tambah(a, b)

// Assert
assert.Equal(t, 5, hasil)
```

---

**Assertion**

Artinya:
Pernyataan tegas bahwa suatu kondisi harus benar. Kalau tidak benar, test otomatis gagal.

Analogi:
Kontrak: "Saya jamin hasilnya 5. Kalau bukan, saya yang salah."

Contoh:
```go
assert.Equal(t, 5, hasil)   // "hasil harus sama dengan 5"
assert.NoError(t, err)      // "tidak boleh ada error"
assert.True(t, aktif)       // "aktif harus bernilai true"
```

---

**allocs/op**

Artinya:
Jumlah alokasi heap memory per satu operasi dalam benchmark.

Analogi:
Berapa kali kamu minta "lembar kerja baru" ke kantor setiap mengerjakan satu tugas.

Catatan:
Semakin sedikit allocs/op, semakin efisien kode kamu.

---

## B

---

**b.N**

Artinya:
Jumlah iterasi yang ditentukan Go secara otomatis dalam benchmark. Go menyesuaikan nilainya agar hasil benchmark cukup akurat.

Analogi:
Jumlah putaran yang harus dilari atlet untuk mendapatkan waktu rata-rata yang akurat.

Catatan:
Jangan ubah nilai `b.N`. Selalu pakai di dalam `for i := 0; i < b.N; i++`.

---

**b.ResetTimer()**

Artinya:
Mereset timer benchmark ke 0. Waktu yang terpakai sebelum perintah ini tidak dihitung dalam hasil.

Analogi:
Stopwatch yang di-reset setelah pemanasan, sebelum lomba dimulai.

Contoh:
```go
func BenchmarkContoh(b *testing.B) {
    data := setupBerat()  // ini tidak dihitung
    b.ResetTimer()        // timer mulai dari sini
    for i := 0; i < b.N; i++ {
        Proses(data)
    }
}
```

---

**Benchmark**

Artinya:
Proses mengukur kecepatan eksekusi kode. Go menjalankan kode berkali-kali dan melaporkan waktu rata-rata per operasi.

Analogi:
Stopwatch yang mengukur berapa lama atlet berlari.

Contoh:
```go
func BenchmarkTambah(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Tambah(2, 3)
    }
}
```

Cara jalankan: `go test -bench=.`

---

**B/op**

Artinya:
Jumlah byte memori yang dialokasikan per operasi dalam benchmark.

Catatan:
Muncul saat menjalankan dengan `-benchmem`. Semakin kecil semakin baik.

---

## D

---

**defer**

Artinya:
Perintah kepada Go untuk menjalankan sebuah fungsi saat fungsi yang sedang berjalan selesai (baik berhasil maupun error).

Analogi:
"Tolong cuci piring nanti setelah saya selesai makan" — kamu tidak perlu ingat-ingat, asisten yang melakukannya otomatis.

Contoh:
```go
func TestKoneksi(t *testing.T) {
    db := BukaDatabase()
    defer db.Tutup()  // dieksekusi saat TestKoneksi selesai
    // ... isi test
}
```

---

## E

---

**Edge Case**

Artinya:
Kondisi batas atau ekstrem yang jarang terjadi tapi mungkin menyebabkan bug. Contoh: angka nol, string kosong, nilai negatif, input sangat besar.

Analogi:
Uji ban mobil tidak hanya di jalan mulus, tapi juga di jalan berbatu, di hujan deras, di suhu ekstrem.

Catatan:
Edge case sering dilupakan dan menjadi sumber bug yang paling berbahaya.

---

**End-to-End Test (E2E)**

Artinya:
Test yang mensimulasikan penggunaan nyata dari sudut pandang pengguna, dari awal sampai akhir.

Analogi:
Uji coba mobil dengan mengendarainya di jalan asli (bukan di laboratorium).

Catatan:
Paling lambat dan paling kompleks. Tidak dibahas dalam materi ini.

---

**Expected**

Artinya:
Nilai yang kamu harapkan sebagai output dari fungsi yang diuji.

Analogi:
Makanan yang kamu pesan (yang kamu harapkan datang ke meja).

Contoh:
```go
expected := 5
actual := Tambah(2, 3)
assert.Equal(t, expected, actual)
```

Catatan:
Di testify, `expected` selalu di sebelah kiri: `assert.Equal(t, expected, actual)`.

---

## F

---

**`t.Fail()`**

Artinya:
Menandai test sebagai GAGAL tanpa pesan, tapi tetap lanjutkan eksekusi test.

Catatan:
Jarang dipakai langsung. Lebih baik pakai `t.Error` yang menyertakan pesan.

---

**`t.FailNow()`**

Artinya:
Menandai test sebagai GAGAL tanpa pesan, dan langsung berhenti.

Catatan:
Jarang dipakai langsung. Lebih baik pakai `t.Fatal` yang menyertakan pesan.

---

**`t.Fatal("pesan")`**

Artinya:
Menandai test sebagai GAGAL dengan pesan, lalu langsung berhenti.

Analogi:
Menemukan jembatan putus — tidak perlu lanjut berjalan.

Contoh:
```go
if err != nil {
    t.Fatalf("Gagal buka database: %v", err)
    // kode setelah ini tidak dieksekusi
}
```

---

## G

---

**go test**

Artinya:
Perintah di terminal untuk menjalankan semua test di folder saat ini.

Contoh:
```bash
go test          # jalankan semua test
go test -v       # tampilkan detail setiap test
go test -bench=. # jalankan semua benchmark
go test -cover   # tampilkan code coverage
```

---

## H

---

**Happy Path**

Artinya:
Kondisi normal yang diharapkan berhasil. Input valid, semua berjalan sesuai rencana.

Analogi:
Pelanggan memesan makanan yang tersedia, membayar dengan uang yang cukup, makan, selesai. Tidak ada masalah.

Catatan:
Jangan hanya test happy path — selalu test juga error case dan edge case.

---

## I

---

**Integration Test**

Artinya:
Test yang memeriksa apakah beberapa komponen bisa bekerja bersama dengan benar.

Analogi:
Setelah setiap komponen mobil diuji sendiri, uji apakah mesin + transmisi + roda bisa bekerja bersama.

---

**Interface**

Artinya:
Kontrak yang mendefinisikan "apa yang bisa dilakukan" tanpa menentukan "bagaimana caranya". Tipe apapun yang memenuhi kontrak bisa digunakan.

Analogi:
Standar colokan listrik — apapun perangkatnya, kalau colokannya cocok, bisa dipakai.

Contoh:
```go
type EmailSender interface {
    Kirim(to, subject, body string) error
}
// Implementasi nyata dan mock keduanya bisa memenuhi interface ini
```

---

**Isolasi Test**

Artinya:
Prinsip bahwa setiap test harus bisa berjalan sendiri tanpa bergantung pada test lain, database nyata, internet, atau sistem eksternal.

Analogi:
Menguji kualitas garam langsung — tidak perlu masak seluruh makanan dulu.

Catatan:
Mock adalah alat utama untuk mencapai isolasi test.

---

## M

---

**m.Run()**

Artinya:
Perintah di dalam `TestMain` untuk menjalankan semua test. Wajib dipanggil, kalau tidak, tidak ada test yang berjalan.

Contoh:
```go
func TestMain(m *testing.M) {
    setup()
    exitCode := m.Run()  // jalankan semua test di sini
    teardown()
    os.Exit(exitCode)
}
```

---

**Mock**

Artinya:
Objek palsu yang meniru perilaku objek asli untuk keperluan testing, tanpa melakukan operasi yang sebenarnya.

Analogi:
Simulator penerbangan — terasa seperti pesawat nyata tapi tidak ada risiko nyata kalau "crash".

Contoh:
```go
type MockEmailSender struct { mock.Mock }

func (m *MockEmailSender) Kirim(to, subject, body string) error {
    args := m.Called(to, subject, body)
    return args.Error(0)
}
```

---

**mock.Anything**

Artinya:
Placeholder di testify/mock yang cocok dengan argumen apapun.

Contoh:
```go
// Cocok dengan SimpanUser yang dipanggil dengan argumen APAPUN
mockRepo.On("SimpanUser", mock.Anything).Return(nil)
```

---

## N

---

**ns/op**

Artinya:
Nanosecond per operasi — satuan utama hasil benchmark. Menunjukkan berapa nanodetik yang dibutuhkan untuk satu kali eksekusi kode.

Analogi:
Satuan kecepatan: km/jam. Semakin kecil ns/op, semakin cepat kodenya.

Catatan:
1 ns = 0.000000001 detik. Komputer modern sangat cepat!

---

## P

---

**Package**

Artinya:
Kumpulan kode yang dikelompokkan bersama dengan nama yang sama. Go punya banyak package bawaan (`fmt`, `testing`, `strings`, dll.).

Analogi:
Kotak peralatan bertabel. "testing" berisi alat-alat testing, "fmt" berisi alat-alat format teks.

---

**PASS / FAIL / SKIP**

Artinya:
Tiga kemungkinan status hasil test:
- **PASS** — test berhasil, kode berjalan sesuai harapan
- **FAIL** — test gagal, ada sesuatu yang tidak sesuai harapan
- **SKIP** — test dilewati dengan sengaja

---

## R

---

**`require`**

Artinya:
Sub-package testify yang seperti `assert` tapi langsung berhenti jika gagal (seperti `t.Fatal`).

Contoh:
```go
require.NoError(t, err)    // gagal → berhenti
assert.Equal(t, 5, hasil)  // gagal → catat tapi lanjut
```

Catatan:
Pakai `require` untuk kondisi yang kalau gagal membuat langkah berikutnya tidak masuk akal.

---

## S

---

**Setup**

Artinya:
Persiapan yang dilakukan sebelum test berjalan. Contoh: membuka koneksi database, menyiapkan data test.

Analogi:
Panitia menyiapkan lapangan sebelum pertandingan dimulai.

---

**Slice**

Artinya:
Tipe data di Go berupa daftar yang bisa berubah ukurannya. Seperti array yang fleksibel.

Contoh:
```go
angka := []int{1, 2, 3, 4, 5}
kata  := []string{"halo", "dunia"}
```

---

**struct**

Artinya:
Tipe data yang mengelompokkan beberapa nilai dengan nama. Seperti formulir dengan kolom-kolom.

Contoh:
```go
type User struct {
    Nama  string
    Umur  int
    Email string
}
```

---

**Sub Benchmark**

Artinya:
Benchmark di dalam benchmark, dibuat dengan `b.Run(...)`. Mirip seperti sub test tapi untuk benchmark.

Contoh:
```go
func BenchmarkGabung(b *testing.B) {
    b.Run("10_kata", func(b *testing.B) { ... })
    b.Run("100_kata", func(b *testing.B) { ... })
}
```

---

**Sub Test**

Artinya:
Test di dalam test, dibuat dengan `t.Run(...)`. Berguna untuk mengelompokkan banyak kasus dalam satu fungsi test.

Contoh:
```go
func TestTambah(t *testing.T) {
    t.Run("positif", func(t *testing.T) { ... })
    t.Run("negatif", func(t *testing.T) { ... })
}
```

---

## T

---

**t.Error("pesan")**

Artinya:
Menandai test sebagai GAGAL dengan pesan, tapi tetap lanjutkan eksekusi test.

Analogi:
Menemukan lubang di jalan — catat, tapi lanjut berjalan untuk cek kondisi jalan lainnya.

---

**t.Log("pesan")**

Artinya:
Mencetak pesan log. Hanya muncul kalau test gagal atau menggunakan flag `-v`.

Catatan:
Berbeda dengan `fmt.Println` — `t.Log` terintegrasi dengan sistem test.

---

**t.Run(nama, func)**

Artinya:
Membuat sub test. Test yang berjalan di dalam test utama dengan nama tersendiri.

---

**t.Skip("alasan")**

Artinya:
Melewati test ini. Test tidak dianggap gagal, tidak dianggap berhasil — berstatus SKIP.

Contoh:
```go
if os.Getenv("DB_HOST") == "" {
    t.Skip("DB_HOST tidak diset")
}
```

---

**t.Skipf("format", args...)**

Artinya:
Sama seperti `t.Skip` tapi bisa format pesan seperti `fmt.Sprintf`.

---

**Table Test (Table-Driven Test)**

Artinya:
Pola testing di mana semua kasus uji disimpan dalam tabel (slice of struct), lalu diiterasi dan dijalankan satu per satu.

Analogi:
Lembar soal ujian — semua soal dan kunci jawaban di satu tempat, mudah ditambah dan diubah.

---

**Teardown**

Artinya:
Pembersihan yang dilakukan setelah test selesai. Contoh: menutup koneksi database, menghapus file sementara.

Analogi:
Panitia membereskan lapangan setelah pertandingan selesai.

---

**`testing.T`**

Artinya:
Tipe data yang dipakai sebagai alat laporan dalam unit test. Dipakai untuk memberitahu Go bahwa test berhasil atau gagal.

---

**`testing.B`**

Artinya:
Tipe data yang dipakai sebagai alat dalam benchmark. Berisi `b.N`, `b.ResetTimer()`, dll.

---

**`testing.M`**

Artinya:
Tipe data yang dipakai dalam `TestMain`. Berisi `m.Run()` untuk menjalankan semua test.

---

**Testify**

Artinya:
Library populer dari pihak ketiga untuk mempermudah penulisan assertion dan mock di Go.

Cara install:
```bash
go get github.com/stretchr/testify
```

Sub-package:
- `assert` — assertion yang lanjut meski gagal
- `require` — assertion yang berhenti saat gagal
- `mock` — alat untuk membuat mock object

---

**TestMain**

Artinya:
Fungsi khusus di Go yang menjadi "pintu masuk" semua test dalam satu package. Dipakai untuk setup sebelum semua test dan teardown setelahnya.

Contoh:
```go
func TestMain(m *testing.M) {
    setup()
    exitCode := m.Run()
    teardown()
    os.Exit(exitCode)
}
```

---

## U

---

**Unit Test**

Artinya:
Test yang memeriksa satu fungsi atau satu bagian kecil kode secara terisolasi, tanpa bergantung ke sistem eksternal.

Analogi:
Memeriksa setiap komponen mesin satu per satu sebelum merakit mobil.

Karakteristik:
- Cepat (milidetik)
- Terisolasi (tidak butuh database, internet, dll.)
- Deterministik (selalu hasil sama kalau input sama)

---

## V

---

**verbose (`-v`)**

Artinya:
Flag `go test -v` yang menampilkan detail setiap test yang berjalan (nama test, PASS/FAIL/SKIP), bukan hanya hasil akhir.

Analogi:
Laporan nilai per mata pelajaran, bukan hanya "lulus/tidak lulus".

---

## W

---

**wantErr**

Artinya:
Konvensi nama field dalam table test yang menandakan apakah test tersebut diharapkan menghasilkan error atau tidak.

Contoh:
```go
tests := []struct {
    name    string
    input   string
    wantErr bool
}{
    {"valid", "budi@gmail.com", false},
    {"invalid", "bukan-email", true},
}
```

---

## Simbol Umum dalam Kode Test

| Simbol | Artinya |
|---|---|
| `*testing.T` | Pointer ke objek test — alat laporan unit test |
| `*testing.B` | Pointer ke objek benchmark — alat laporan benchmark |
| `*testing.M` | Pointer ke objek test main |
| `%d` | Placeholder angka integer dalam format string |
| `%s` | Placeholder string dalam format string |
| `%v` | Placeholder nilai apapun (value) dalam format string |
| `%w` | Placeholder error yang bisa di-unwrap |
| `:=` | Deklarasi dan inisialisasi variabel sekaligus |
| `_` | Blank identifier — mengabaikan nilai yang tidak diperlukan |

---

## Perintah `go test` yang Sering Dipakai

```bash
go test                          # jalankan semua test
go test -v                       # verbose — tampilkan detail
go test -run TestNama            # jalankan test tertentu
go test -run "TestNama/sub"      # jalankan sub test tertentu
go test -bench=.                 # jalankan semua benchmark
go test -bench=BenchmarkNama     # jalankan benchmark tertentu
go test -benchmem                # tampilkan info memori
go test -benchtime=5s            # jalankan benchmark minimal 5 detik
go test -cover                   # tampilkan code coverage
go test ./...                    # jalankan test di semua sub-folder
```
