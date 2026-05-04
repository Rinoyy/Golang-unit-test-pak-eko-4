# 10 — Table Test

> Video: 00:56:44 — 01:04:38

---

## Masalah yang Diselesaikan

Pada Sub Test, kita masih menulis setiap kasus secara terpisah:

```go
t.Run("kasus 1", func(t *testing.T) {
    assert.Equal(t, 5, Tambah(2, 3))
})
t.Run("kasus 2", func(t *testing.T) {
    assert.Equal(t, 0, Tambah(0, 0))
})
t.Run("kasus 3", func(t *testing.T) {
    assert.Equal(t, -5, Tambah(-2, -3))
})
```

Ini masih ada banyak pengulangan kode. Kalau ada 20 kasus, kodenya jadi sangat panjang.

**Table Test** menyelesaikan ini dengan cara meletakkan semua kasus dalam sebuah "tabel" (slice of struct), lalu iterasi.

---

## Apa Itu Table Test?

**Istilah: Table Test (Table-Driven Test)**

Artinya:
Pola testing di mana semua kasus uji diletakkan dalam struktur data (tabel), lalu kode test dijalankan sekali untuk setiap baris di tabel tersebut.

Analogi:
Bayangkan tabel Excel:
- Setiap baris = satu kasus uji
- Kolom = input dan expected output
- Kamu tinggal baca setiap baris dan jalankan test yang sama

---

## Cara Membuat Table Test

```go
func TestTambah(t *testing.T) {

    // 1. Definisikan tabel kasus uji
    kasusUji := []struct {
        nama     string
        a        int
        b        int
        expected int
    }{
        {"dua angka positif", 2, 3, 5},
        {"dengan nol", 0, 5, 5},
        {"dua negatif", -2, -3, -5},
        {"positif dan negatif", 10, -4, 6},
        {"nol dan nol", 0, 0, 0},
    }

    // 2. Loop setiap kasus dan jalankan test
    for _, kasus := range kasusUji {
        t.Run(kasus.nama, func(t *testing.T) {
            hasil := Tambah(kasus.a, kasus.b)
            assert.Equal(t, kasus.expected, hasil)
        })
    }
}
```

### Penjelasan Detail:

```go
kasusUji := []struct {
    nama     string
    a        int
    b        int
    expected int
}{
```

Ini membuat **slice of struct anonim**. Artinya: daftar yang berisi struct (kumpulan field).

**Istilah: struct**

Artinya:
Tipe data yang mengelompokkan beberapa nilai dengan nama. Seperti formulir yang punya kolom-kolom.

**Istilah: slice**

Artinya:
Daftar yang bisa berisi banyak item. Seperti tabel dengan banyak baris.

```go
{"dua angka positif", 2, 3, 5},
```
Satu baris di tabel: nama = "dua angka positif", a = 2, b = 3, expected = 5.

```go
for _, kasus := range kasusUji {
```
Loop setiap baris di tabel. `kasus` adalah satu baris.

```go
    t.Run(kasus.nama, func(t *testing.T) {
```
Buat sub test dengan nama dari kolom `nama`.

---

## Output Table Test

```bash
go test -v
```

```
=== RUN   TestTambah
=== RUN   TestTambah/dua_angka_positif
--- PASS: TestTambah/dua_angka_positif (0.00s)
=== RUN   TestTambah/dengan_nol
--- PASS: TestTambah/dengan_nol (0.00s)
=== RUN   TestTambah/dua_negatif
--- PASS: TestTambah/dua_negatif (0.00s)
=== RUN   TestTambah/positif_dan_negatif
--- PASS: TestTambah/positif_dan_negatif (0.00s)
=== RUN   TestTambah/nol_dan_nol
--- PASS: TestTambah/nol_dan_nol (0.00s)
--- PASS: TestTambah (0.00s)
PASS
```

---

## Menambahkan Kasus Baru = Sangat Mudah

Dengan table test, menambahkan kasus baru semudah menambah satu baris:

```go
kasusUji := []struct {
    nama     string
    a        int
    b        int
    expected int
}{
    {"dua angka positif", 2, 3, 5},
    {"dengan nol", 0, 5, 5},
    {"dua negatif", -2, -3, -5},
    // ← tambahkan kasus baru di sini!
    {"angka besar", 1000000, 999999, 1999999},
}
```

Tidak perlu tambah fungsi baru, tidak perlu duplikasi kode.

---

## Contoh Lebih Kompleks: Validasi Input

```go
func Bagi(a, b int) (int, error) {
    if b == 0 {
        return 0, fmt.Errorf("tidak bisa dibagi dengan nol")
    }
    return a / b, nil
}

func TestBagi(t *testing.T) {
    kasusUji := []struct {
        nama     string
        a        int
        b        int
        expected int
        adaError bool
    }{
        {"pembagian normal", 10, 2, 5, false},
        {"hasil nol", 0, 5, 0, false},
        {"dibagi nol", 10, 0, 0, true},
        {"angka negatif", -10, 2, -5, false},
    }

    for _, kasus := range kasusUji {
        t.Run(kasus.nama, func(t *testing.T) {
            hasil, err := Bagi(kasus.a, kasus.b)

            if kasus.adaError {
                assert.Error(t, err, "seharusnya ada error")
            } else {
                assert.NoError(t, err, "seharusnya tidak ada error")
                assert.Equal(t, kasus.expected, hasil)
            }
        })
    }
}
```

---

## Cara Berpikir: Table Test = Dokumentasi Sekaligus Test

Table test punya manfaat tambahan: **tabel itu sendiri adalah dokumentasi**.

Siapapun yang membaca tabel ini langsung paham semua perilaku fungsi yang ditest:

```go
kasusUji := []struct {
    input    string
    expected bool
}{
    {"budi@gmail.com", true},    // email valid
    {"budi@", false},            // tanpa domain → invalid
    {"budigmail.com", false},    // tanpa @ → invalid
    {"", false},                 // kosong → invalid
    {"a@b.c", true},             // format minimal → valid
}
```

Membaca tabel ini sama seperti membaca spesifikasi fungsi.

---

## Pola Table Test yang Paling Umum di Go

```go
func TestNamaFungsi(t *testing.T) {
    tests := []struct {
        name     string    // nama kasus (untuk t.Run)
        input    TipeInput // input ke fungsi
        expected TipeOutput // output yang diharapkan
        wantErr  bool      // apakah diharapkan ada error?
    }{
        {
            name:     "kasus sukses",
            input:    ...,
            expected: ...,
            wantErr:  false,
        },
        {
            name:    "kasus gagal",
            input:   ...,
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := NamaFungsi(tt.input)
            
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            
            assert.NoError(t, err)
            assert.Equal(t, tt.expected, got)
        })
    }
}
```

Perhatikan nama variabel:
- `tt` — konvensi untuk "test table row", nama variabel dalam loop
- `got` — konvensi untuk "yang didapat" (actual)
- `wantErr` — konvensi untuk "apakah ingin ada error"

---

## Kenapa Ini Disebut "Table-Driven Test"?

Karena cara berpikirnya seperti mengisi tabel:

| nama | input a | input b | expected | ada error? |
|---|---|---|---|---|
| normal | 10 | 2 | 5 | tidak |
| bagi nol | 10 | 0 | - | ya |
| negatif | -10 | 2 | -5 | tidak |

Kamu mendefinisikan data di tabel, lalu kode test tinggal membaca dan menjalankan setiap baris.

---

## Analogi: Lembar Soal Ujian

Bayangkan seorang guru membuat soal ujian matematika:

**Tanpa table test** = Guru membuat soal satu per satu, menulis kunci jawaban di tempat berbeda-beda, susah untuk update.

**Dengan table test** = Guru punya lembar yang berisi:
| Soal | Jawaban |
|---|---|
| 2 + 3 | 5 |
| 10 - 3 | 7 |
| 4 × 5 | 20 |

Semua soal dan jawaban di satu tempat. Mudah ditambah, mudah diubah, mudah dipahami.

---

## Kesalahan Umum Pemula

1. **Lupa `t.Run` di dalam loop** — kalau tidak pakai `t.Run`, semua kasus dianggap satu test, kamu tidak bisa tahu kasus mana yang gagal

2. **Nama kasus tidak deskriptif** — `"test1"`, `"test2"` tidak membantu. Tulis nama yang menjelaskan kondisinya.

3. **Terlalu banyak field di struct** — kalau struct punya 10 field, tabel jadi susah dibaca. Pertimbangkan untuk pisah menjadi beberapa fungsi test.

4. **Tidak test edge case** — tabel yang hanya berisi happy path tidak berguna. Selalu sertakan kasus batas (nol, negatif, kosong, dll.)

---

## Latihan

**Tugas:**

Buat fungsi `Pangkat(base, exp int) int` yang menghitung `base` pangkat `exp`.

Asumsi:
- `exp` tidak akan negatif
- Kalau `exp == 0` → return 1 (apapun dipangkat 0 = 1)

Buat table test dengan minimal 5 kasus.

---

### Jawaban

Fungsi:
```go
func Pangkat(base, exp int) int {
    if exp == 0 {
        return 1
    }
    hasil := 1
    for i := 0; i < exp; i++ {
        hasil *= base
    }
    return hasil
}
```

Test:
```go
func TestPangkat(t *testing.T) {
    tests := []struct {
        name     string
        base     int
        exp      int
        expected int
    }{
        {"pangkat satu", 5, 1, 5},
        {"pangkat dua", 3, 2, 9},
        {"pangkat tiga", 2, 3, 8},
        {"pangkat nol", 100, 0, 1},
        {"basis nol", 0, 5, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            hasil := Pangkat(tt.base, tt.exp)
            assert.Equal(t, tt.expected, hasil)
        })
    }
}
```

---

## Selanjutnya

Lanjut ke: [11 — Mock](11-mock.md)

Di sana kita belajar cara "berpura-pura" menjadi sistem lain agar test bisa berjalan terisolasi.
