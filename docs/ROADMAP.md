# Roadmap Belajar Software Testing di Golang

> Panduan belajar dari nol. Ikuti urutan ini. Jangan lompat.

---

## Peta Perjalanan Belajar

```
MULAI DI SINI
     |
     v
[01] Pendahuluan
     |
     v
[02] Pengenalan Software Testing      <-- Fondasi paling penting
     |
     v
[03] Pengenalan Testing Package       <-- Kenalan dengan alat
     |
     v
[04] Membuat Unit Test                <-- Praktek pertama
     |
     v
[05] Menggagalkan Test                <-- Cara memberi tahu "ada yang salah"
     |
     v
[06] Assertion                        <-- Cara cek hasil lebih rapi
     |
     v
[07] Skip Test                        <-- Cara lewati test sementara
     |
     v
[08] Before dan After Test            <-- Persiapan sebelum/sesudah test
     |
     v
[09] Sub Test                         <-- Test di dalam test
     |
     v
[10] Table Test                       <-- Test banyak kasus sekaligus
     |
     v
[11] Mock                             <-- Pura-pura jadi sistem lain
     |
     v
[12] Benchmark (Pengenalan)           <-- Ukur kecepatan kode
     |
     v
[13] Membuat Benchmark                <-- Praktek benchmark
     |
     v
[14] Sub Benchmark                    <-- Benchmark di dalam benchmark
     |
     v
[15] Table Benchmark                  <-- Benchmark banyak kasus
     |
     v
SELESAI - Siap dipakai di project nyata!
```

---

## Daftar File

| No  | File                                  | Topik                    | Waktu Video  |
|-----|---------------------------------------|--------------------------|--------------|
| 01  | [01-pendahuluan.md](01-pendahuluan.md) | Pendahuluan              | 00:00:00     |
| 02  | [02-pengenalan-software-testing.md](02-pengenalan-software-testing.md) | Apa itu Software Testing | 00:01:57 |
| 03  | [03-pengenalan-testing-package.md](03-pengenalan-testing-package.md) | Testing Package di Go    | 00:10:15     |
| 04  | [04-membuat-unit-test.md](04-membuat-unit-test.md) | Membuat Unit Test        | 00:13:27     |
| 05  | [05-menggagalkan-test.md](05-menggagalkan-test.md) | Menggagalkan Test        | 00:25:37     |
| 06  | [06-assertion.md](06-assertion.md)    | Assertion                | 00:33:31     |
| 07  | [07-skip-test.md](07-skip-test.md)    | Skip Test                | 00:43:24     |
| 08  | [08-before-after-test.md](08-before-after-test.md) | Before dan After Test    | 00:46:35  |
| 09  | [09-sub-test.md](09-sub-test.md)      | Sub Test                 | 00:51:27     |
| 10  | [10-table-test.md](10-table-test.md)  | Table Test               | 00:56:44     |
| 11  | [11-mock.md](11-mock.md)              | Mock                     | 01:04:38     |
| 12  | [12-benchmark-pengenalan.md](12-benchmark-pengenalan.md) | Pengenalan Benchmark     | 01:26:25  |
| 13  | [13-membuat-benchmark.md](13-membuat-benchmark.md) | Membuat Benchmark        | 01:30:23  |
| 14  | [14-sub-benchmark.md](14-sub-benchmark.md) | Sub Benchmark            | 01:38:19     |
| 15  | [15-table-benchmark.md](15-table-benchmark.md) | Table Benchmark          | 01:40:52  |
| --  | [KAMUS-ISTILAH.md](KAMUS-ISTILAH.md)  | Kamus Istilah Lengkap    | --           |

---

## Tips Belajar

- Baca satu file, lalu langsung coba kodenya
- Jangan lanjut ke file berikutnya kalau belum paham
- Kamus istilah bisa dibuka kapan saja saat menemukan kata yang tidak dimengerti
- Setiap file ada latihan — kerjakan dulu, baru lihat jawaban

---

## Yang Akan Kamu Bisa Setelah Selesai

- Menulis unit test untuk setiap fungsi yang kamu buat
- Mendeteksi bug sebelum sampai ke user
- Mengukur seberapa cepat kode kamu berjalan
- Berbicara soal testing dengan developer lain
