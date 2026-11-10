# Menghubungkan 9Router dengan OpenCode di Windows (OpenCode V2)

Panduan ini mendokumentasikan proses pemasangan dan troubleshooting **9Router + OpenCode di Windows menggunakan CMD**, berdasarkan proses setup yang dijalankan pada 10 November 2026.

Dokumen ini mencatat kondisi yang benar-benar sudah diuji, kesalahan yang sempat muncul, perbaikannya, serta langkah terakhir yang masih perlu diuji. Tujuannya bukan sekadar menampilkan konfigurasi yang terlihat rapi, tetapi memberi jejak pemeriksaan agar masalah berikutnya bisa dilokalisasi tanpa menginstal ulang semua aplikasi.

> **Status terakhir:** server 9Router aktif, endpoint API berhasil diuji dengan API key, environment variable Windows tersimpan dan terbaca, plugin 9Router versi `0.9.1` aktif, dan model 9Router sudah muncul di daftar model OpenCode. Tes pemanggilan model melalui `opencode run` setelah service dimulai ulang **belum dikonfirmasi berhasil**. Lihat bagian [Status pengujian terakhir](#status-pengujian-terakhir).

---

## Daftar Isi

1. [Tujuan dan alur koneksi](#tujuan-dan-alur-koneksi)
2. [Lingkungan yang digunakan](#lingkungan-yang-digunakan)
3. [Perbedaan tutorial lama dan konfigurasi V2](#perbedaan-tutorial-lama-dan-konfigurasi-v2)
4. [Prasyarat](#prasyarat)
5. [Memastikan 9Router aktif](#memastikan-9router-aktif)
6. [Memeriksa daftar model](#memeriksa-daftar-model)
7. [Menguji API 9Router secara langsung](#menguji-api-9router-secara-langsung)
8. [Menyimpan API key di Windows CMD](#menyimpan-api-key-di-windows-cmd)
9. [Membuat konfigurasi OpenCode V2](#membuat-konfigurasi-opencode-v2)
10. [Memastikan plugin dan model terdeteksi](#memastikan-plugin-dan-model-terdeteksi)
11. [Mengatur environment untuk OpenCode Service](#mengatur-environment-untuk-opencode-service)
12. [Tes akhir OpenCode](#tes-akhir-opencode)
13. [Riwayat error dan solusinya](#riwayat-error-dan-solusinya)
14. [Checklist troubleshooting](#checklist-troubleshooting)
15. [Keamanan API key](#keamanan-api-key)
16. [Status pengujian terakhir](#status-pengujian-terakhir)
17. [Referensi](#referensi)

---

## 1. Tujuan dan alur koneksi

Target akhirnya adalah menggunakan model yang disediakan 9Router langsung dari OpenCode.

```text
Windows 10
   |
   | ROUTER_API_KEY
   v
OpenCode v2.2.5
   |
   | Plugin: @vheins/opencode-9router@0.9.1
   v
Provider: 9router
   |
   | HTTP API
   v
http://127.0.0.1:20128/v1
   |
   v
9Router lokal
   |
   v
Model dari 9Router
   |
   v
Jawaban AI di OpenCode
```

Dua endpoint utama yang digunakan:

| Endpoint | Fungsi |
|---|---|
| `http://127.0.0.1:20128/v1/models` | Mengambil daftar model yang tersedia dari 9Router. |
| `http://127.0.0.1:20128/v1/chat/completions` | Mengirim permintaan chat ke model melalui API kompatibel OpenAI. |

Perlu dibedakan antara **server aktif**, **API key valid**, **plugin terpasang**, **model terdaftar**, dan **request model berhasil**. Kelima hal tersebut adalah pemeriksaan yang berbeda.

---

## 2. Lingkungan yang digunakan

Berikut lingkungan yang teridentifikasi dari pengujian:

| Komponen | Versi / nilai | Status berdasarkan pengujian |
|---|---|---|
| Sistem operasi | Windows 10, `10.0.19044.7807` | Teridentifikasi dari CMD |
| OpenCode | `v2.2.5` | Terverifikasi dengan `opencode --version` |
| Plugin | `@vheins/opencode-9router@0.9.1` | Terverifikasi dengan `opencode plugin list` |
| Node.js | Terpasang | Versi persis tidak dicatat di log terakhir |
| npm | Terpasang | Versi persis tidak dicatat di log terakhir |
| 9Router CLI | Terpasang | Perintah versi sebelumnya dinyatakan berhasil; versi persis tidak dicatat |
| Alamat server 9Router | `127.0.0.1:20128` | Endpoint berhasil diakses |
| Alamat OpenCode Service | `127.0.0.1:28894` | Terlihat dari `opencode service status` |
| Environment variable | `ROUTER_API_KEY` | Terbaca pada CMD baru dan tersimpan di Registry |

### Periksa versi aplikasi

Jalankan di CMD:

```cmd
opencode --version
node -v
npm -v
9router --version
```

Contoh hasil OpenCode yang sudah diverifikasi pada setup ini:

```text
opencode v2.2.5
```

Jika semua perintah menampilkan versi, aplikasi dan perintah CLI dikenali oleh CMD. Itu belum membuktikan bahwa server 9Router atau autentikasi API sudah berjalan.

---

## 3. Perbedaan tutorial lama dan konfigurasi V2

Tutorial awal menggunakan konfigurasi OpenCode V1. Sementara setup Windows ini menggunakan OpenCode `v2.2.5`, sehingga konfigurasi harus mengikuti format V2.

| Bagian | Format OpenCode V1 | Format OpenCode V2 |
|---|---|---|
| Daftar plugin | `plugin` | `plugins` |
| Daftar provider | `provider` | `providers` |
| Paket provider | `npm` | `package` |
| Pengaturan provider | `options` | `settings` |
| Runtime OpenAI-compatible | `@ai-sdk/openai-compatible` | `@opencode/ai/providers/openai-compatible` |

Contoh V1:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["@vheins/opencode-9router@0.7.0"]
}
```

Contoh V2:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugins": ["@vheins/opencode-9router@0.9.1"]
}
```

**Jangan mencampur struktur V1 dan V2 dalam satu konfigurasi.** Untuk OpenCode `v2.2.5`, gunakan `plugins` dan `providers` sesuai dokumentasi plugin V2.

### Mengapa versi plugin 0.7.0 tidak dipakai?

Tutorial awal mengunci plugin pada `0.7.0`. Versi tersebut mendahului dukungan OpenCode V2. Berdasarkan changelog repository plugin:

- `0.9.0` menambahkan dukungan OpenCode V2.
- `0.9.1` memperbaiki pendaftaran model untuk provider yang didefinisikan di konfigurasi V2.

Pada pengujian Windows ini, plugin `0.9.1` berhasil dimuat dan model 9Router berhasil muncul di daftar model OpenCode.

---

## 4. Prasyarat

Pastikan hal berikut tersedia:

- Windows 10 atau Windows yang kompatibel.
- Node.js dan npm.
- OpenCode V2.
- 9Router terpasang dan dapat dijalankan.
- Server 9Router berjalan di port `20128`.
- API key untuk autentikasi API 9Router.
- Plugin `@vheins/opencode-9router` akan dimuat oleh OpenCode melalui konfigurasi.

Tidak perlu menginstal ulang aplikasi jika perintah versi masing-masing sudah berjalan. Dalam troubleshooting ini, instalasi ulang bukan langkah pertama.

---

## 5. Memastikan 9Router aktif

### 5.1 Pastikan aplikasi 9Router berjalan

Jalankan 9Router seperti biasa dan pastikan server lokalnya aktif pada port `20128`.

Alamat dasar yang digunakan:

```text
http://127.0.0.1:20128
```

### 5.2 Periksa endpoint model

Jalankan di CMD:

```cmd
curl -i http://127.0.0.1:20128/v1/models
```

Pada setup ini, perintah tersebut berhasil mengembalikan JSON dengan struktur seperti berikut:

```json
{
  "object": "list",
  "data": [
    {
      "id": "cbai/glm-5.2",
      "object": "model",
      "owned_by": "cbai"
    },
    {
      "id": "kr/auto",
      "object": "model",
      "owned_by": "kr"
    },
    {
      "id": "oc/glm-5.3-flash-free",
      "object": "model",
      "owned_by": "oc"
    }
  ]
}
```

Contoh tersebut hanya cuplikan. Respons asli berisi lebih banyak model.

Jika daftar model muncul, artinya endpoint `/v1/models` dapat diakses. Namun, ini **belum membuktikan** bahwa API key valid untuk endpoint chat.

### 5.3 Jika endpoint gagal

| Hasil | Kemungkinan arti | Tindakan |
|---|---|---|
| `200 OK` dan JSON model | Endpoint model dapat diakses | Lanjut uji chat completion |
| `Connection refused` | Server belum berjalan atau port salah | Jalankan 9Router dan periksa port |
| `404 Not Found` | Path endpoint salah atau tidak tersedia | Periksa endpoint API 9Router |
| `401 Unauthorized` | Endpoint meminta autentikasi | Uji dengan header API key jika endpoint tersebut mewajibkannya |

---

## 6. Memeriksa daftar model

Model yang dikembalikan 9Router memiliki ID seperti:

```text
oc/glm-5.3-flash-free
kr/claude-sonnet-4.5
kr/deepseek-3.2
cbai/glm-5.2
```

Setelah plugin mendaftarkan provider ke OpenCode, model ditampilkan dengan prefix provider `9router/`.

Contoh:

| ID model dari 9Router | ID model di OpenCode |
|---|---|
| `oc/glm-5.3-flash-free` | `9router/oc/glm-5.3-flash-free` |
| `kr/claude-sonnet-4.5` | `9router/kr/claude-sonnet-4.5` |
| `kr/deepseek-3.2` | `9router/kr/deepseek-3.2` |

ID model harus disalin persis dari daftar OpenCode. Jangan menghilangkan prefix `9router/` ketika menjalankan model lewat OpenCode.

---

## 7. Menguji API 9Router secara langsung

Pengujian ini dilakukan sebelum menghubungkan API ke OpenCode. Tujuannya memisahkan masalah 9Router dari masalah konfigurasi OpenCode.

### 7.1 Pengujian tanpa API key: gagal

Perintah yang dijalankan:

```cmd
curl -i -X POST "http://127.0.0.1:20128/v1/chat/completions" -H "Content-Type: application/json" -d "{\"model\":\"oc/glm-5.3-flash-free\",\"messages\":[{\"role\":\"user\",\"content\":\"Balas hanya: koneksi berhasil\"}],\"stream\":false}"
```

Hasil yang diterima:

```text
HTTP/1.1 401 Unauthorized
```

```json
{
  "error": {
    "message": "Missing API key",
    "type": "authentication_error",
    "code": "invalid_api_key"
  }
}
```

**Kesimpulan:** endpoint chat dapat dijangkau, tetapi request ditolak karena header autentikasi tidak dikirim.

### 7.2 Pengujian dengan API key: berhasil

Tambahkan header:

```text
Authorization: Bearer API_KEY_9ROUTER
```

Contoh perintah CMD yang menggunakan environment variable agar key tidak perlu ditulis langsung:

```cmd
curl -i -X POST "http://127.0.0.1:20128/v1/chat/completions" -H "Content-Type: application/json" -H "Authorization: Bearer %ROUTER_API_KEY%" -d "{\"model\":\"oc/glm-5.3-flash-free\",\"messages\":[{\"role\":\"user\",\"content\":\"Balas hanya: koneksi berhasil\"}],\"stream\":false}"
```

Hasil yang benar-benar diterima pada pengujian:

```text
HTTP/1.1 200 OK
```

Contoh inti respons:

```json
{
  "object": "chat.completion",
  "model": "glm-5.3-flash-free",
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "koneksi berhasil"
      },
      "finish_reason": "stop"
    }
  ]
}
```

Respons aktual juga menampilkan informasi penggunaan token dan sumber billing. Pengujian saat itu menunjukkan `billing_source: free` dan `cost: 0`.

**Kesimpulan yang sudah terbukti:** server 9Router dapat menerima API key dan mengembalikan jawaban model melalui endpoint chat completion.

---

## 8. Menyimpan API key di Windows CMD

Gunakan satu nama environment variable secara konsisten:

```text
ROUTER_API_KEY
```

Jangan mencampur nama ini dengan `OPENCODE_9ROUTER_API_KEY` dari tutorial lama, kecuali konfigurasi yang dipakai memang secara eksplisit merujuk ke nama lama tersebut.

### 8.1 Simpan API key

Di CMD:

```cmd
setx ROUTER_API_KEY "API_KEY_9ROUTER_KAMU"
```

Ganti placeholder dengan API key milik sendiri. Jangan gunakan teks contoh sebagai key.

Jika berhasil, Windows menampilkan:

```text
SUCCESS: Specified value was saved.
```

### 8.2 Tutup dan buka CMD baru

Ini langkah penting. `setx` menyimpan environment variable untuk proses baru. CMD yang sedang terbuka tidak otomatis mendapatkan nilai terbaru.

Tutup CMD:

```cmd
exit
```

Buka CMD baru dari Start Menu.

### 8.3 Periksa environment variable tanpa menampilkan key

```cmd
if defined ROUTER_API_KEY (echo API KEY TERBACA) else (echo API KEY BELUM TERBACA)
```

Hasil yang diharapkan:

```text
API KEY TERBACA
```

### 8.4 Periksa apakah tersimpan di Registry

```cmd
reg query HKCU\Environment /v ROUTER_API_KEY >nul 2>&1 && echo API KEY TERSIMPAN || echo API KEY BELUM TERSIMPAN
```

Hasil yang diharapkan:

```text
API KEY TERSIMPAN
```

Perintah ini tidak menampilkan isi API key.

### 8.5 Hasil pengujian yang tercatat

Pada CMD pertama setelah menjalankan `setx`, pemeriksaan berikut menghasilkan:

```text
API KEY BELUM TERBACA
```

Itu terjadi karena CMD yang sama belum memuat environment variable baru.

Setelah CMD ditutup dan dibuka kembali, hasilnya menjadi:

```text
API KEY TERBACA
API KEY TERSIMPAN
```

Jadi penyimpanan environment variable Windows sudah berhasil.

### 8.6 Perintah PowerShell yang pernah digunakan

Pada proses troubleshooting sempat digunakan:

```cmd
powershell -NoProfile -Command "if ($env:ROUTER_API_KEY) {'API KEY TERBACA'} else {'API KEY BELUM TERBACA'}"
```

Namun, panduan ini menggunakan CMD. Pemeriksaan CMD dengan `if defined` sudah cukup dan tidak perlu berpindah shell.

---

## 9. Membuat konfigurasi OpenCode V2

### 9.1 Lokasi file konfigurasi

Lokasi konfigurasi global Windows:

```text
%USERPROFILE%\.config\opencode\opencode.json
```

Untuk akun Windows `fabian`, lokasi lengkapnya:

```text
C:\Users\fabian\.config\opencode\opencode.json
```

### 9.2 Jangan menjalankan file JSON sebagai perintah CMD

Kesalahan yang pernah terjadi:

```cmd
C:\Users\fabian>C:\Users\fabian\.config\opencode\opencode.json
```

CMD membalas bahwa path tersebut bukan perintah yang dikenali. Itu wajar karena file JSON bukan executable.

Buat folder jika belum ada:

```cmd
if not exist "%USERPROFILE%\.config\opencode" mkdir "%USERPROFILE%\.config\opencode"
```

Buka file menggunakan Notepad:

```cmd
notepad "%USERPROFILE%\.config\opencode\opencode.json"
```

Jika file belum ada, pilih **Yes** ketika Notepad menawarkan membuat file baru.

### 9.3 Konfigurasi OpenCode V2 yang sesuai dokumentasi plugin

Gunakan konfigurasi berikut untuk OpenCode V2 dengan 9Router lokal dan environment variable `ROUTER_API_KEY`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugins": [
    {
      "package": "@vheins/opencode-9router@0.9.1",
      "options": {
        "baseURL": "http://127.0.0.1:20128",
        "apiKey": "{env:ROUTER_API_KEY}",
        "cache": false,
        "discoveryTimeout": 60000
      }
    }
  ],
  "providers": {
    "9router": {
      "name": "9Router",
      "package": "@opencode/ai/providers/openai-compatible",
      "settings": {
        "baseURL": "http://127.0.0.1:20128/v1",
        "apiKey": "{env:ROUTER_API_KEY}"
      }
    }
  }
}
```

Simpan dengan `Ctrl + S`.

#### Fungsi tiap bagian

| Bagian | Fungsi |
|---|---|
| `plugins` | Memuat plugin eksternal pada OpenCode V2. |
| `package` | Menentukan paket plugin `@vheins/opencode-9router`. |
| `options.baseURL` | Alamat dasar server 9Router untuk proses discovery plugin. |
| `options.apiKey` | API key yang dipakai plugin ketika mengakses backend untuk discovery jika autentikasi dibutuhkan. |
| `cache: false` | Menonaktifkan cache discovery sementara selama troubleshooting. Setelah stabil, cache dapat diaktifkan kembali. |
| `discoveryTimeout` | Batas waktu proses discovery dalam milidetik. Nilai `60000` berarti 60 detik. |
| `providers` | Mendefinisikan provider dalam format OpenCode V2. |
| `providers.9router.package` | Menggunakan runtime OpenAI-compatible milik OpenCode V2. |
| `providers.9router.settings.baseURL` | Endpoint API kompatibel OpenAI, termasuk `/v1`. |
| `providers.9router.settings.apiKey` | API key yang digunakan saat OpenCode mengirim request chat ke provider. |

**Penting:** `baseURL` pada opsi plugin menggunakan alamat dasar `http://127.0.0.1:20128`, sedangkan `baseURL` provider menggunakan `http://127.0.0.1:20128/v1`.

### 9.4 Jangan menambahkan properti `env` yang tidak diperlukan

Dalam salah satu langkah troubleshooting sebelumnya, sempat disarankan menambahkan properti berikut:

```text
"env": ["ROUTER_API_KEY"]
```

Properti tersebut **tidak tercantum dalam contoh konfigurasi provider V2 pada README resmi plugin**. Jangan jadikan properti itu bagian dari konfigurasi final. Gunakan referensi environment langsung pada `apiKey`:

```text
"apiKey": "{env:ROUTER_API_KEY}"
```

### 9.5 Mengapa ada konfigurasi plugin dan provider sekaligus?

Plugin menyediakan proses penemuan model dan mendaftarkan model untuk provider dengan ID yang diawali `9router`.

Provider V2 mendefinisikan runtime OpenAI-compatible, endpoint `/v1`, serta autentikasi request model.

README resmi plugin menjelaskan bahwa plugin dapat mendaftarkan provider default jika belum ada provider keluarga `9router`. Pada konfigurasi di atas, provider `9router` ditulis secara eksplisit agar alamat endpoint dan API key request dapat ditentukan dengan jelas.

---

## 10. Memastikan plugin dan model terdeteksi

### 10.1 Periksa konfigurasi OpenCode

```cmd
opencode debug config
```

Gunakan perintah ini untuk memeriksa konfigurasi yang berhasil dibaca OpenCode. Jika muncul pesan konfigurasi tidak valid, periksa tanda kurung, koma, tanda kutip, dan struktur JSON.

### 10.2 Periksa plugin

```cmd
opencode plugin list
```

Hasil yang sudah diperoleh pada setup ini:

```text
ID       VERSION  SOURCE
9router  0.9.1    @vheins/opencode-9router@0.9.1
```

Ini membuktikan plugin terpasang dan dikenali OpenCode.

### 10.3 Periksa daftar model

```cmd
opencode models
```

Untuk menyaring model dari provider 9Router:

```cmd
opencode models | findstr /i "9router"
```

Hasil yang sudah diperoleh mencakup:

```text
9router/all
9router/cbai/deepseek-v4.1-flash
9router/cbai/glm-5.1
9router/cbai/glm-5.2
9router/cbai/glm-5v-turbo
9router/cbai/kimi-k2.5
9router/cbai/kimi-k2.6
9router/cbai/kimi-k2.7
9router/cbai/minimax-m3
9router/kr/auto
9router/kr/auto-thinking
9router/kr/claude-haiku-4.5
9router/kr/claude-sonnet-4.5
9router/kr/deepseek-3.2
9router/kr/glm-5
9router/kr/qwen3-coder-next
9router/mmf/mimo-auto
9router/oc/beam-free
9router/oc/flint-40b-free
9router/oc/glm-5.3-flash-free
9router/oc/jev-1.13-free
9router/oc/mimo-v2.6-flash-free
9router/oc/nemotron-3.5-lightning-free
9router/oc/nemotron-3.5-super-vl-free
```

Log asli berisi daftar yang lebih lengkap, termasuk varian `thinking` dan `agentic`.

**Kesimpulan:** plugin berhasil dimuat dan discovery model berhasil. Ini adalah salah satu hasil terpenting dari troubleshooting.

---

## 11. Mengatur environment untuk OpenCode Service

OpenCode menggunakan client-server architecture. Background service dapat berjalan sebagai proses tersendiri. Karena itu, environment variable yang tersedia pada CMD belum tentu tersedia pada proses service yang sudah berjalan.

### 11.1 Periksa service

```cmd
opencode service status
```

Alamat service yang terlihat pada setup ini:

```text
http://127.0.0.1:28894
```

Alamat `28894` adalah alamat OpenCode Service, bukan port 9Router. Port 9Router tetap `20128`.

### 11.2 Teruskan environment variable ke service

Pastikan CMD sudah membaca `ROUTER_API_KEY`, lalu jalankan:

```cmd
opencode service set env ROUTER_API_KEY "%ROUTER_API_KEY%"
```

Perintah ini digunakan untuk meneruskan environment variable ke proses OpenCode Service.

Jangan menambahkan tanda kutip yang hilang atau mengubah nama menjadi `OPENCODE_9ROUTER_API_KEY` ketika konfigurasi JSON menggunakan `ROUTER_API_KEY`.

### 11.3 Restart service

Setelah mengubah environment service, restart service:

```cmd
opencode service restart
```

Jika perlu, service dapat dihentikan dan dinyalakan kembali secara terpisah:

```cmd
opencode service stop
opencode service start
```

Kemudian periksa:

```cmd
opencode service status
```

Perintah service yang tersedia dapat dilihat dengan:

```cmd
opencode service --help
```

### 11.4 Periksa kesehatan service

```cmd
opencode api get /api/info
```

Dokumentasi troubleshooting OpenCode juga menyarankan `service status`, pemeriksaan `/api/info`, dan restart service saat mendiagnosis masalah client-server.

---

## 12. Tes akhir OpenCode

Pastikan:

1. 9Router berjalan.
2. CMD baru dapat membaca `ROUTER_API_KEY`.
3. File `opencode.json` menggunakan struktur V2 yang benar.
4. Plugin `9router` tampil pada `opencode plugin list`.
5. Model target tampil pada `opencode models`.
6. OpenCode Service sudah menerima environment variable dan direstart.

Kemudian jalankan:

```cmd
opencode run --model 9router/oc/glm-5.3-flash-free "Balas hanya: OpenCode berhasil terhubung ke 9Router"
```

Hasil yang diharapkan:

```text
> build · oc/glm-5.3-flash-free

OpenCode berhasil terhubung ke 9Router
```

Alternatif model yang juga muncul dalam daftar hasil discovery:

```cmd
opencode run --model 9router/kr/deepseek-3.2 "Balas hanya: koneksi berhasil"
```

Gunakan model yang memang masih tersedia dalam daftar `opencode models` saat pengujian. Daftar model dapat berubah mengikuti 9Router.

---

## 13. Riwayat error dan solusinya

Bagian ini mencatat kesalahan yang muncul selama proses setup, termasuk penyebab dan penanganannya.

### 13.1 Path `opencode.json` dianggap sebagai perintah CMD

**Perintah yang keliru:**

```cmd
C:\Users\fabian\.config\opencode\opencode.json
```

**Error:**

```text
is not recognized as an internal or external command
```

**Penyebab:** file JSON dicoba dijalankan seperti program.

**Solusi:** buka file menggunakan Notepad:

```cmd
notepad "%USERPROFILE%\.config\opencode\opencode.json"
```

### 13.2 `401 Unauthorized` dan `Missing API key` dari curl

**Penyebab:** request `/v1/chat/completions` tidak menyertakan header `Authorization`.

**Solusi:** kirim header:

```text
Authorization: Bearer %ROUTER_API_KEY%
```

Contoh perintah lengkap tersedia di bagian [Menguji API 9Router secara langsung](#menguji-api-9router-secara-langsung).

**Hasil akhir:** berhasil mendapatkan `HTTP/1.1 200 OK` dan jawaban `koneksi berhasil`.

### 13.3 `setx` berhasil, tetapi CMD yang sama belum membaca key

**Gejala:**

```text
SUCCESS: Specified value was saved.
API KEY BELUM TERBACA
```

**Penyebab:** `setx` menyimpan variabel untuk proses baru. CMD lama belum memperbarui environment-nya.

**Solusi:** tutup CMD, buka CMD baru, kemudian periksa:

```cmd
if defined ROUTER_API_KEY (echo API KEY TERBACA) else (echo API KEY BELUM TERBACA)
```

**Hasil akhir:** `API KEY TERBACA` dan `API KEY TERSIMPAN`.

### 13.4 `Missing API key` saat menjalankan OpenCode

**Perintah yang gagal:**

```cmd
opencode run --model 9router/oc/glm-5.3-flash-free "Halo, apakah koneksi berhasil?"
```

**Error:**

```text
Error: Missing API key
```

**Kemungkinan penyebab yang diperiksa:**

- CMD memiliki environment variable, tetapi proses OpenCode Service belum tentu memilikinya.
- Provider harus memiliki referensi API key pada pengaturan V2.
- Plugin options dan provider settings memiliki fungsi berbeda.

**Langkah perbaikan yang disiapkan:**

1. Pastikan `ROUTER_API_KEY` terbaca pada CMD baru.
2. Pastikan `options.apiKey` pada plugin menggunakan `{env:ROUTER_API_KEY}`.
3. Pastikan `providers.9router.settings.apiKey` menggunakan `{env:ROUTER_API_KEY}`.
4. Teruskan variabel ke service:

   ```cmd
   opencode service set env ROUTER_API_KEY "%ROUTER_API_KEY%"
   ```

5. Restart service:

   ```cmd
   opencode service restart
   ```

6. Periksa ulang model dan jalankan tes akhir.

**Catatan status:** perintah `opencode service set env` sudah pernah dijalankan. Setelah itu error sempat berubah menjadi `Model unavailable`. Namun, tes akhir setelah model kembali terdaftar belum tercatat. Karena itu, jangan menyebut autentikasi OpenCode sudah berhasil sebelum perintah tes akhir benar-benar menghasilkan jawaban.

### 13.5 `Model unavailable: 9router/oc/glm-5.3-flash-free`

**Error yang pernah muncul:**

```text
Error: Model unavailable: 9router/oc/glm-5.3-flash-free
```

**Kemungkinan penyebab:** model belum tersedia pada katalog/provider yang sedang digunakan oleh proses OpenCode saat request dijalankan, atau service belum memuat ulang konfigurasi dan daftar model setelah perubahan.

**Pemeriksaan:**

```cmd
opencode service status
opencode models | findstr /i "9router"
opencode plugin list
```

Pada pemeriksaan berikutnya, model `9router/oc/glm-5.3-flash-free` sudah muncul kembali dan plugin `9router 0.9.1` terdeteksi.

**Solusi yang perlu diuji setelah status model terdaftar:** restart service, kemudian jalankan ulang perintah `opencode run`.

### 13.6 `Config file is not valid JSON`

Jika muncul, periksa:

- Semua `{` memiliki pasangan `}`.
- Semua `[` memiliki pasangan `]`.
- String menggunakan tanda kutip ganda `"`.
- Tidak ada koma berlebih setelah elemen terakhir.
- Properti JSON tidak ditulis sebagai teks tanpa tanda kutip.
- Struktur memakai format V2, bukan campuran V1 dan V2.

Periksa konfigurasi dengan:

```cmd
opencode debug config
```

### 13.7 Log `Cache HIT`

Contoh:

```text
[9router-provider] [discovery] Cache HIT ...
```

Ini bukan error. Plugin menggunakan hasil discovery yang tersimpan di cache.

Selama troubleshooting, konfigurasi menggunakan:

```text
"cache": false
```

Setelah koneksi stabil, cache dapat diaktifkan kembali. Dokumentasi plugin menyebut cache default berlangsung selama tiga jam.

### 13.8 `Insufficient account funds`

Jika request sudah mencapai upstream tetapi muncul:

```text
Upstream request failed: Insufficient account funds
```

Masalahnya bukan otomatis berasal dari OpenCode atau plugin. Periksa saldo, kredit, kuota, atau kebijakan billing pada route/provider upstream yang dipilih.

### 13.9 CMD dan PowerShell menggunakan sintaks berbeda

Untuk CMD:

```cmd
if defined ROUTER_API_KEY (echo API KEY TERBACA) else (echo API KEY BELUM TERBACA)
```

Untuk PowerShell:

```powershell
if ($env:ROUTER_API_KEY) { 'API KEY TERBACA' } else { 'API KEY BELUM TERBACA' }
```

Jangan menggunakan sintaks `$env:...` langsung di CMD. Panduan ini memprioritaskan CMD agar konsisten dengan lingkungan pengguna.

---

## 14. Checklist troubleshooting

Jalankan pemeriksaan secara berurutan. Jangan mengganti API key, menghapus cache, mengganti versi plugin, dan menginstal ulang semua aplikasi sekaligus. Itu hanya membuat sumber masalah semakin sulit ditemukan.

### A. Aplikasi dan versi

```cmd
opencode --version
node -v
npm -v
9router --version
```

### B. Server 9Router aktif

```cmd
curl -i http://127.0.0.1:20128/v1/models
```

### C. Environment variable terbaca

```cmd
if defined ROUTER_API_KEY (echo API KEY TERBACA) else (echo API KEY BELUM TERBACA)
```

### D. Environment variable tersimpan

```cmd
reg query HKCU\Environment /v ROUTER_API_KEY >nul 2>&1 && echo API KEY TERSIMPAN || echo API KEY BELUM TERSIMPAN
```

### E. API chat completion berhasil

```cmd
curl -i -X POST "http://127.0.0.1:20128/v1/chat/completions" -H "Content-Type: application/json" -H "Authorization: Bearer %ROUTER_API_KEY%" -d "{\"model\":\"oc/glm-5.3-flash-free\",\"messages\":[{\"role\":\"user\",\"content\":\"Balas hanya: koneksi berhasil\"}],\"stream\":false}"
```

Target: `HTTP/1.1 200 OK`.

### F. Konfigurasi valid

```cmd
opencode debug config
```

### G. Plugin terpasang

```cmd
opencode plugin list
```

Target pada setup ini:

```text
9router  0.9.1  @vheins/opencode-9router@0.9.1
```

### H. Model 9Router terdaftar

```cmd
opencode models | findstr /i "9router"
```

### I. Environment service dan status service

```cmd
opencode service set env ROUTER_API_KEY "%ROUTER_API_KEY%"
opencode service restart
opencode service status
```

### J. Tes OpenCode

```cmd
opencode run --model 9router/oc/glm-5.3-flash-free "Balas hanya: koneksi berhasil"
```

---

## 15. Keamanan API key

API key adalah kredensial akses. Perlakukan seperti password.

**Jangan:**

- Menaruh API key asli di `README.md`.
- Menaruh API key asli di `opencode.json`.
- Mengunggah API key ke GitHub.
- Mengirim API key melalui chat, screenshot, atau issue publik.
- Melakukan commit file konfigurasi yang berisi key asli.

**Gunakan:**

- Windows User Environment Variable `ROUTER_API_KEY`.
- Referensi environment pada konfigurasi: `{env:ROUTER_API_KEY}`.
- Placeholder pada dokumentasi: `API_KEY_9ROUTER_KAMU`.

Jika key asli pernah terpublikasi atau terkirim ke tempat yang tidak aman, revoke key tersebut dari 9Router, buat key baru, lalu simpan ulang dengan `setx`.

Perintah pemeriksaan di panduan ini sengaja tidak menampilkan isi API key.

---

## 16. Status pengujian terakhir

Bagian ini memisahkan hasil yang telah terbukti dari langkah yang belum dikonfirmasi.

### Sudah berhasil

- [x] OpenCode versi `2.2.5` dikenali CMD.
- [x] 9Router berjalan pada `127.0.0.1:20128`.
- [x] `GET /v1/models` mengembalikan daftar model.
- [x] Request chat tanpa API key menghasilkan `401 Missing API key`, sesuai perilaku autentikasi.
- [x] Request chat dengan header `Authorization: Bearer ...` menghasilkan `200 OK`.
- [x] Model `oc/glm-5.3-flash-free` berhasil menjawab `koneksi berhasil` melalui API 9Router.
- [x] `ROUTER_API_KEY` berhasil disimpan menggunakan `setx`.
- [x] CMD baru berhasil membaca `ROUTER_API_KEY`.
- [x] Registry Windows menyimpan `ROUTER_API_KEY`.
- [x] Plugin `@vheins/opencode-9router@0.9.1` tampil pada `opencode plugin list`.
- [x] Model 9Router, termasuk `9router/oc/glm-5.3-flash-free`, tampil pada `opencode models`.
- [x] OpenCode Service berjalan pada `http://127.0.0.1:28894`.

### Belum dikonfirmasi

- [ ] Tes final `opencode run` setelah service menerima environment variable dan model kembali terdaftar menghasilkan jawaban AI.

### Perintah tes final

```cmd
opencode run --model 9router/oc/glm-5.3-flash-free "Balas hanya: OpenCode berhasil terhubung ke 9Router"
```

Jika perintah tersebut menghasilkan jawaban yang diminta, ubah checklist di atas menjadi selesai dan catat tanggal pengujian.

---

## 17. Referensi

Dokumentasi yang digunakan untuk menyusun konfigurasi dan troubleshooting:

1. **Plugin OpenCode 9Router**  
   https://github.com/vheins/opencode-9router
2. **README versi bahasa Indonesia plugin 9Router**  
   https://github.com/vheins/opencode-9router/blob/main/README.id.md
3. **Release plugin 9Router**  
   https://github.com/vheins/opencode-9router/releases
4. **OpenCode Configuration**  
   https://opencode.ai/docs/config/
5. **OpenCode Plugins**  
   https://opencode.ai/docs/plugins/
6. **OpenCode CLI Commands**  
   https://opencode.ai/docs/cli/commands/
7. **OpenCode Troubleshooting**  
   https://opencode.ai/docs/troubleshooting/

Catatan versi: contoh konfigurasi plugin V2 pada panduan ini mengikuti dokumentasi repository `@vheins/opencode-9router` yang diperiksa pada 10 November 2026. Versi plugin dipatok ke `0.9.1` agar konfigurasi yang digunakan tidak berubah diam-diam ketika versi terbaru dirilis.

---

## Ringkasan konfigurasi final

| Pengaturan | Nilai |
|---|---|
| OS | Windows 10 |
| OpenCode | `v2.2.5` |
| Plugin | `@vheins/opencode-9router@0.9.1` |
| Environment variable | `ROUTER_API_KEY` |
| 9Router base URL untuk plugin | `http://127.0.0.1:20128` |
| 9Router API base URL untuk provider | `http://127.0.0.1:20128/v1` |
| Model uji | `oc/glm-5.3-flash-free` |
| Model ID OpenCode | `9router/oc/glm-5.3-flash-free` |
| OpenCode Service | `http://127.0.0.1:28894` |

**Kesimpulan:** server dan API 9Router sudah terbukti berfungsi. Plugin 9Router `0.9.1` juga sudah berhasil dimuat dan mendaftarkan model ke OpenCode `v2.2.5`. Tahap yang masih harus dibuktikan adalah keberhasilan request model dari `opencode run` setelah environment variable diteruskan ke OpenCode Service dan service direstart.
