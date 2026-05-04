# 11 — Mock

> Video: 01:04:38 — 01:26:25

---

## Masalah yang Diselesaikan

Bayangkan kamu punya fungsi yang mengirim email:

```go
func ProsesOrderan(orderID string) error {
    // 1. Ambil data dari database
    order := database.AmbilOrder(orderID)
    
    // 2. Proses pembayaran
    payment.Proses(order.Total)
    
    // 3. Kirim email konfirmasi
    email.Kirim(order.Email, "Pesanan dikonfirmasi")
    
    return nil
}
```

Bagaimana kamu test fungsi ini?

**Masalah:**
- Kamu tidak mau beneran mengakses database saat test
- Kamu tidak mau beneran charge kartu kredit saat test
- Kamu tidak mau beneran kirim email saat test

**Mock adalah solusinya.**

---

## Apa Itu Mock?

**Istilah: Mock**

Artinya:
"Objek palsu" yang meniru perilaku objek asli, tapi tidak melakukan operasi yang sebenarnya.

Analogi:
Bayangkan latihan pemadaman kebakaran. Kamu tidak membakar gedung sungguhan — kamu pakai asap buatan. Asap palsu itu adalah "mock" dari kebakaran nyata. Cukup untuk melatih prosedur, tanpa risiko nyata.

---

## Konsep Dasar: Interface

Sebelum bisa mock, kamu harus paham **interface**.

**Istilah: Interface**

Artinya:
Kontrak yang mendefinisikan "apa yang bisa dilakukan" tanpa mendefinisikan "bagaimana caranya". Siapapun yang memenuhi kontrak ini bisa digunakan secara bergantian.

Analogi:
Interface seperti colokan listrik standar. Standarnya: harus punya 2 lubang, voltase tertentu. Apapun perangkatnya (lampu, TV, kulkas) — kalau memenuhi standar, bisa dicolok. Kamu tidak peduli merek atau modelnya.

```go
// Interface: "siapapun yang bisa mengirim email"
type EmailSender interface {
    Kirim(to string, subject string, body string) error
}

// Implementasi nyata
type GmailSender struct { ... }
func (g *GmailSender) Kirim(to, subject, body string) error {
    // kode nyata kirim email via Gmail
}

// Implementasi mock (untuk test)
type MockEmailSender struct { ... }
func (m *MockEmailSender) Kirim(to, subject, body string) error {
    // tidak kirim email beneran, hanya pura-pura
    return nil
}
```

---

## Langkah Membuat Mock

### Langkah 1: Definisikan Interface

```go
// Di file: email.go

type EmailSender interface {
    Kirim(to string, subject string, body string) error
}
```

### Langkah 2: Ubah Fungsi untuk Menerima Interface

```go
// Di file: orderan.go

// Fungsi menerima interface, bukan implementasi konkret
func ProsesOrderan(orderID string, sender EmailSender) error {
    order := AmbilOrder(orderID)
    
    err := sender.Kirim(order.Email, "Konfirmasi", "Pesanan dikonfirmasi")
    if err != nil {
        return fmt.Errorf("gagal kirim email: %w", err)
    }
    
    return nil
}
```

### Langkah 3: Buat Mock untuk Test

```go
// Di file: orderan_test.go

// Mock struct
type MockEmailSender struct {
    PesanTerkirim []struct {
        To      string
        Subject string
        Body    string
    }
    ShouldError bool  // simulasikan error kalau diperlukan
}

// Implementasikan interface
func (m *MockEmailSender) Kirim(to, subject, body string) error {
    if m.ShouldError {
        return fmt.Errorf("gagal kirim email (simulasi)")
    }
    
    // Catat pesan yang "dikirim" (tidak beneran dikirim)
    m.PesanTerkirim = append(m.PesanTerkirim, struct {
        To      string
        Subject string
        Body    string
    }{to, subject, body})
    
    return nil
}
```

### Langkah 4: Pakai Mock di Test

```go
func TestProsesOrderan(t *testing.T) {
    // Buat mock
    mockEmail := &MockEmailSender{}
    
    // Jalankan fungsi dengan mock
    err := ProsesOrderan("order-123", mockEmail)
    
    // Verifikasi tidak ada error
    assert.NoError(t, err)
    
    // Verifikasi email "terkirim" (tapi tidak beneran kirim)
    assert.Len(t, mockEmail.PesanTerkirim, 1)
    assert.Equal(t, "user@example.com", mockEmail.PesanTerkirim[0].To)
}

func TestProsesOrderanGagalKirimEmail(t *testing.T) {
    // Mock yang mensimulasikan error
    mockEmail := &MockEmailSender{ShouldError: true}
    
    err := ProsesOrderan("order-123", mockEmail)
    
    // Harus ada error
    assert.Error(t, err)
}
```

---

## Menggunakan Library Mock: testify/mock

Membuat mock secara manual seperti di atas bisa melelahkan. Testify punya sub-package `mock` yang memudahkan:

```bash
# Sudah terinstall bersama testify
go get github.com/stretchr/testify/mock
```

### Cara Pakai testify/mock

```go
package main

import (
    "testing"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
)

// Interface yang ingin di-mock
type UserRepository interface {
    AmbilUser(id string) (*User, error)
    SimpanUser(user *User) error
}

// Mock struct — embed mock.Mock
type MockUserRepository struct {
    mock.Mock  // embed ini untuk dapat semua fitur mock
}

// Implementasikan setiap method interface
func (m *MockUserRepository) AmbilUser(id string) (*User, error) {
    // Panggil m.Called untuk mencatat bahwa method ini dipanggil
    args := m.Called(id)
    return args.Get(0).(*User), args.Error(1)
}

func (m *MockUserRepository) SimpanUser(user *User) error {
    args := m.Called(user)
    return args.Error(0)
}
```

### Cara Set Expectation dan Jalankan Test

```go
func TestAmbilProfil(t *testing.T) {
    // Buat mock
    mockRepo := new(MockUserRepository)
    
    // Set expectation:
    // "Kalau AmbilUser dipanggil dengan 'user-1',
    //  kembalikan user ini dan tidak ada error"
    userDiharapkan := &User{ID: "user-1", Nama: "Budi"}
    mockRepo.On("AmbilUser", "user-1").Return(userDiharapkan, nil)
    
    // Jalankan fungsi yang ditest
    profil := AmbilProfil("user-1", mockRepo)
    
    // Verifikasi hasil
    assert.Equal(t, "Budi", profil.Nama)
    
    // Verifikasi bahwa AmbilUser benar-benar dipanggil
    mockRepo.AssertExpectations(t)
}
```

### Penjelasan:

```go
mockRepo.On("AmbilUser", "user-1").Return(userDiharapkan, nil)
```
- `On("AmbilUser", "user-1")` → "ketika method AmbilUser dipanggil dengan argumen 'user-1'"
- `.Return(userDiharapkan, nil)` → "kembalikan user ini dan nil (tidak ada error)"

```go
mockRepo.AssertExpectations(t)
```
Verifikasi bahwa semua method yang di-set dengan `On` benar-benar dipanggil selama test. Kalau `AmbilUser` tidak pernah dipanggil → test gagal.

---

## Contoh Lengkap: Service dengan Dependency

```go
// File: user.go

type User struct {
    ID    string
    Nama  string
    Email string
}

type UserRepository interface {
    AmbilUser(id string) (*User, error)
    SimpanUser(user *User) error
}

type UserService struct {
    repo UserRepository
}

func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}

func (s *UserService) GantiNama(userID, namaBaru string) error {
    user, err := s.repo.AmbilUser(userID)
    if err != nil {
        return fmt.Errorf("user tidak ditemukan: %w", err)
    }
    
    user.Nama = namaBaru
    
    return s.repo.SimpanUser(user)
}
```

```go
// File: user_test.go

type MockUserRepo struct {
    mock.Mock
}

func (m *MockUserRepo) AmbilUser(id string) (*User, error) {
    args := m.Called(id)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*User), args.Error(1)
}

func (m *MockUserRepo) SimpanUser(user *User) error {
    args := m.Called(user)
    return args.Error(0)
}

func TestGantiNama(t *testing.T) {
    mockRepo := new(MockUserRepo)
    
    // Setup: user yang akan dikembalikan saat AmbilUser dipanggil
    userAwal := &User{ID: "u1", Nama: "Budi Lama"}
    mockRepo.On("AmbilUser", "u1").Return(userAwal, nil)
    mockRepo.On("SimpanUser", mock.Anything).Return(nil)
    
    service := NewUserService(mockRepo)
    err := service.GantiNama("u1", "Budi Baru")
    
    assert.NoError(t, err)
    assert.Equal(t, "Budi Baru", userAwal.Nama)
    mockRepo.AssertExpectations(t)
}

func TestGantiNamaUserTidakDitemukan(t *testing.T) {
    mockRepo := new(MockUserRepo)
    
    // Setup: AmbilUser mengembalikan error
    mockRepo.On("AmbilUser", "tidak-ada").Return(nil, fmt.Errorf("not found"))
    
    service := NewUserService(mockRepo)
    err := service.GantiNama("tidak-ada", "Nama Baru")
    
    assert.Error(t, err)
    mockRepo.AssertExpectations(t)
}
```

---

## `mock.Anything` — Argumen Apapun Cocok

```go
// Cocok dengan SimpanUser yang dipanggil dengan argumen APAPUN
mockRepo.On("SimpanUser", mock.Anything).Return(nil)

// vs. hanya cocok dengan user spesifik
mockRepo.On("SimpanUser", &User{Nama: "Budi"}).Return(nil)
```

---

## Cara Berpikir: Isolasi Test dengan Mock

Ingat dari materi sebelumnya: **test harus terisolasi**.

Mock adalah alat utama untuk mencapai isolasi:

```
Tanpa Mock:
Test → Service → Database nyata (tidak stabil, lambat, perlu setup)

Dengan Mock:
Test → Service → Mock Database (stabil, cepat, tidak perlu setup)
```

Kalau test gagal dengan mock, kamu tahu pasti itu karena **kode Service yang salah**, bukan karena masalah database.

---

## Kapan Pakai Mock?

Pakai mock untuk:
- Database
- API eksternal (payment gateway, email service)
- File system
- Jam/waktu (biar bisa test "saat jam 12 malam")
- Apapun yang tidak bisa kamu kontrol sepenuhnya

Jangan mock:
- Fungsi sederhana yang tidak punya side effect
- Logika bisnis itu sendiri

---

## Analogi: Simulator Penerbangan

Pilot baru belajar terbang menggunakan **flight simulator** (simulator), bukan pesawat sungguhan.

Simulator adalah "mock" dari pesawat nyata:
- Tampak seperti pesawat nyata
- Respons seperti pesawat nyata
- Tapi tidak ada konsekuensi nyata kalau "crash"

Pilot bisa belajar dan berlatih dengan aman. Baru setelah lulus ujian simulator, naik pesawat nyata.

Mock dalam testing persis seperti itu.

---

## Kesalahan Umum Pemula

1. **Mock semua hal** — tidak perlu mock fungsi sederhana. Hanya mock yang punya side effect atau dependensi eksternal.

2. **Tidak verifikasi expectation** — lupa `AssertExpectations(t)`, jadi tidak tahu apakah method benar-benar dipanggil.

3. **Mock terlalu detail** — mock yang terlalu spesifik membuat test rapuh. Pakai `mock.Anything` kalau argumen persisnya tidak penting.

4. **Tidak test kasus error** — selalu test bagaimana kode bereaksi saat dependensi mengembalikan error.

---

## Latihan

**Tugas:**

Definisikan interface `NotificationSender`:
```go
type NotificationSender interface {
    KirimSMS(nomor, pesan string) error
}
```

Buat `UserService` dengan method `DaftarUser(nama, nomor string) error` yang:
1. Simpan user (asumsikan berhasil)
2. Kirim SMS sambutan via `NotificationSender`

Buat test menggunakan mock:
1. Test berhasil (SMS terkirim)
2. Test gagal (SMS error)

---

### Jawaban

```go
// Implementasi
type UserService struct {
    notif NotificationSender
}

func (s *UserService) DaftarUser(nama, nomor string) error {
    // simpan user (disederhanakan)
    fmt.Printf("User %s didaftarkan\n", nama)
    
    return s.notif.KirimSMS(nomor, "Selamat datang, "+nama+"!")
}

// Mock
type MockNotif struct {
    mock.Mock
}

func (m *MockNotif) KirimSMS(nomor, pesan string) error {
    args := m.Called(nomor, pesan)
    return args.Error(0)
}

// Test
func TestDaftarUserBerhasil(t *testing.T) {
    mockNotif := new(MockNotif)
    mockNotif.On("KirimSMS", "08123", mock.Anything).Return(nil)
    
    svc := &UserService{notif: mockNotif}
    err := svc.DaftarUser("Budi", "08123")
    
    assert.NoError(t, err)
    mockNotif.AssertExpectations(t)
}

func TestDaftarUserSMSGagal(t *testing.T) {
    mockNotif := new(MockNotif)
    mockNotif.On("KirimSMS", "08123", mock.Anything).
        Return(fmt.Errorf("SMS gagal terkirim"))
    
    svc := &UserService{notif: mockNotif}
    err := svc.DaftarUser("Budi", "08123")
    
    assert.Error(t, err)
    mockNotif.AssertExpectations(t)
}
```

---

## Selanjutnya

Lanjut ke: [12 — Pengenalan Benchmark](12-benchmark-pengenalan.md)

Di sana kita mulai belajar tentang performa — mengukur seberapa cepat kode kamu berjalan.
