# 01 — Pendahuluan

> Video: 00:00:00 — 00:01:57

---

## Selamat Datang

Kamu sedang memulai perjalanan belajar **Software Testing di Golang**.

Tidak ada yang perlu kamu tahu dulu. Mulai dari nol itu bukan masalah — justru itu titik awal yang paling jujur.

---

## Apa yang Akan Kamu Pelajari

Kamu akan belajar cara **membuktikan bahwa kode yang kamu tulis benar-benar bekerja**.

Bukan hanya menebak-nebak. Bukan hanya "kayaknya sih bisa". Tapi benar-benar bisa dibuktikan secara otomatis.

---

## Bayangkan Situasi Ini

Kamu baru selesai membuat program. Kamu jalankan, terlihat oke. Kamu kirim ke teman. Teman bilang:

> "Eh, kenapa error?"

Kamu bingung. Di komputermu berjalan normal. Di komputer teman tidak.

**Kenapa bisa terjadi?**

- Mungkin kamu lupa uji beberapa kondisi
- Mungkin kamu uji manual, tapi ada yang terlewat
- Mungkin setelah kamu ubah satu bagian kode, bagian lain jadi rusak tanpa kamu tahu

**Testing adalah solusinya.**

---

## Analogi Dunia Nyata

Bayangkan kamu membuat jembatan.

Sebelum jembatan dibuka untuk umum, ada **uji coba beban**: apakah jembatan bisa menahan truk? Apakah tidak goyang saat ada angin kencang?

Kalau tidak diuji dulu dan ternyata jembatan ambruk — itu bencana.

**Kode program sama seperti jembatan.**

Sebelum program dipakai orang banyak, kita uji dulu: apakah fungsinya benar? Apakah tidak crash saat data tidak biasa? Apakah masih benar setelah kita ubah bagian lain?

Testing adalah "uji beban" untuk kode program.

---

## Kenapa Belajar Testing Itu Penting

| Tanpa Testing | Dengan Testing |
|---|---|
| Bug ditemukan oleh user | Bug ditemukan sebelum sampai ke user |
| Takut ubah kode karena bisa merusak | Berani ubah kode karena ada jaring pengaman |
| Harus uji manual berulang kali | Test berjalan otomatis dalam hitungan detik |
| Sulit kerja tim | Kode lebih mudah dipahami dan dipercaya tim |

---

## Yang Akan Kamu Buat di Materi Ini

Selama belajar ini, kita akan punya satu contoh kasus yang konsisten:

### Contoh Kasus: Kalkulator Sederhana

Kita akan membuat fungsi `Tambah`, `Kurang`, `Kali`, `Bagi`.

Lalu kita akan **menguji setiap fungsi** itu satu per satu.

Ini sederhana tapi mengajarkan semua konsep testing yang perlu kamu tahu.

---

## Cara Menggunakan Materi Ini

1. **Baca dulu** — pahami konsepnya
2. **Ketik sendiri** — jangan copy-paste, ketik agar otot tanganmu ingat
3. **Jalankan** — lihat hasilnya
4. **Rusak** — coba ubah sesuatu, lihat apa yang berubah
5. **Kerjakan latihan** — sebelum lihat jawaban

---

## Struktur Folder yang Akan Kita Buat

```
golang_unit_test/
├── docs/              <-- materi belajar (folder ini)
├── calculator.go      <-- kode program yang akan diuji
└── calculator_test.go <-- kode test-nya
```

---

## Pesan Penting

> Kode yang tidak ditest adalah kode yang hanya "dikira benar".
>
> Kode yang sudah ditest adalah kode yang "terbukti benar".

Mulai dari sekarang, kita akan membiasakan diri membuat kode yang **terbukti benar**.

---

## Selanjutnya

Lanjut ke: [02 — Pengenalan Software Testing](02-pengenalan-software-testing.md)

Di sana kamu akan benar-benar paham **apa itu testing**, kenapa ada, dan bagaimana cara berpikirnya.
