# Tutorial Super Detail: OpenCode + 9Router di Windows

> Panduan ini dibuat dari setup yang **benar-benar berhasil**: Windows +
> OpenCode + 9Router lokal + plugin `@vheins/opencode-9router`.
>
> Tujuan akhirnya:
>
> ``` text
> OpenCode
>    ↓
> 9Router Plugin
>    ↓
> http://127.0.0.1:20128/v1
>    ↓
> qd/qfmodel
>    ↓
> AI menjawab
> ```
>
> **Penting:** jangan menaruh API key asli di file konfigurasi, README,
> GitHub, atau chat.

------------------------------------------------------------------------

# 0. Hasil Akhir yang Kita Mau

Kalau semuanya sudah benar, command ini:

``` cmd
opencode run --model 9router/qd/qfmodel "halo"
```

akan menghasilkan kira-kira:

``` text
> build · qd/qfmodel

Halo! Apa kabar? Ada yang bisa saya bantu hari ini?
```

Kalau sudah sampai sini, berarti berhasil.

------------------------------------------------------------------------

# 1. Yang Harus Sudah Ada

Sebelum mulai, siapkan:

-   Windows
-   Node.js / npm
-   OpenCode
-   9Router
-   API key 9Router
-   9Router berjalan di port `20128`

Cek OpenCode:

``` cmd
opencode --version
```

Contoh:

``` text
1.18.32
```

------------------------------------------------------------------------

# 2. Cek Apakah 9Router Hidup

9Router kita gunakan melalui:

``` text
http://127.0.0.1:20128/v1
```

Sekarang buka **CMD**.

Jalankan:

``` cmd
curl http://127.0.0.1:20128/v1/models
```

Kalau berhasil, harus keluar JSON yang berisi daftar model.

Contoh sederhana:

``` json
{
  "object": "list",
  "data": [
    {
      "id": "qd/qfmodel"
    }
  ]
}
```

Kalau `/v1/models` tidak bisa diakses:

> **STOP. Jangan lanjut ke OpenCode.**

Berarti masalahnya masih di 9Router.

------------------------------------------------------------------------

# 3. Tes 9Router Sebelum Menyentuh OpenCode

Ini langkah penting.

Kita harus memastikan 9Router memang bisa menjawab sebelum OpenCode
disalahkan.

Endpoint:

``` text
http://127.0.0.1:20128/v1/chat/completions
```

Gunakan API key 9Router milikmu.

Request-nya secara konsep:

``` json
{
  "model": "auto",
  "messages": [
    {
      "role": "user",
      "content": "halo"
    }
  ],
  "stream": true
}
```

Kalau keluar jawaban AI:

``` text
Halo! Ada yang bisa saya bantu?
```

berarti:

``` text
9Router       ✅
API key       ✅
Endpoint      ✅
Model route   ✅
```

Sekarang baru kita masuk OpenCode.

------------------------------------------------------------------------

# 4. Install Plugin 9Router

Plugin yang digunakan:

``` text
@vheins/opencode-9router
```

OpenCode mendukung plugin npm melalui konfigurasi `opencode.json`.

Dokumentasi OpenCode: - https://opencode.ai/docs/plugins/ -
https://dev.opencode.ai/docs/config/

Dokumentasi plugin: - https://github.com/vheins/opencode-9router

Cara paling sederhana adalah menggunakan konfigurasi plugin.

------------------------------------------------------------------------

# 5. Lokasi `opencode.json`

Untuk konfigurasi global Windows:

``` text
C:\Users\NAMA_USER\.config\opencode\opencode.json
```

Contoh:

``` text
C:\Users\fabia\.config\opencode\opencode.json
```

> Kalau username Windows kamu bukan `fabia`, ganti sesuai username
> sendiri.

------------------------------------------------------------------------

# 6. Isi `opencode.json`

Buka:

``` text
C:\Users\fabia\.config\opencode\opencode.json
```

Isi minimal:

``` json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": [
    "@vheins/opencode-9router@0.7.0"
  ]
}
```

## Kenapa `0.7.0`?

Dalam setup kita, versi ini digunakan supaya log discovery plugin tidak
mengganggu tampilan OpenCode.

Versi `latest` dapat berubah seiring update plugin.

Kalau kamu memang ingin menggunakan versi terbaru, bisa menggunakan:

``` json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": [
    "@vheins/opencode-9router@latest"
  ]
}
```

Tetapi untuk setup yang sama persis dengan tutorial ini, gunakan:

``` text
@vheins/opencode-9router@0.7.0
```

> **Jangan campurkan konfigurasi lama dan baru secara sembarangan.**
>
> Kalau plugin sudah melakukan auto-discovery, tidak perlu membuat
> provider manual kecuali memang dibutuhkan.

------------------------------------------------------------------------

# 7. API Key 9Router

Plugin membutuhkan API key.

Kita **tidak** menaruh key di:

``` text
opencode.json
```

Jangan seperti ini:

``` json
{
  "apiKey": "sk-API-RAHASIA"
}
```

Jangan.

API key disimpan di environment Windows.

------------------------------------------------------------------------

# 8. Simpan API Key di Windows

Buka **CMD**.

Jalankan:

``` cmd
setx OPENCODE_9ROUTER_API_KEY "API_KEY_9ROUTER_KAMU"
```

Contoh bentuk:

``` cmd
setx OPENCODE_9ROUTER_API_KEY "sk-xxxxxxxxxxxxxxxx"
```

**Jangan gunakan contoh tersebut sebagai key.**

Masukkan API key asli milikmu.

Kalau berhasil:

``` text
SUCCESS: Specified value was saved.
```

------------------------------------------------------------------------

# 9. Simpan URL 9Router

Masih di CMD:

``` cmd
setx OPENCODE_9ROUTER_URL "http://127.0.0.1:20128/v1"
```

Harus muncul:

``` text
SUCCESS: Specified value was saved.
```

Sekarang Windows memiliki:

``` text
OPENCODE_9ROUTER_API_KEY
OPENCODE_9ROUTER_URL
```

------------------------------------------------------------------------

# 10. PERINGATAN: `setx` Tidak Mengubah CMD yang Sedang Terbuka

Ini bagian yang sangat penting.

Kalau kamu menjalankan:

``` cmd
setx OPENCODE_9ROUTER_API_KEY "..."
```

kemudian langsung:

``` cmd
echo %OPENCODE_9ROUTER_API_KEY%
```

dan hasilnya:

``` text
%OPENCODE_9ROUTER_API_KEY%
```

**jangan panik.**

Itu karena CMD yang sedang terbuka belum mendapatkan environment baru.

`setx` menyimpan variable untuk proses berikutnya.

Jadi:

``` text
CMD LAMA
   ↓
belum tahu variable baru

CMD BARU
   ↓
bisa membaca variable baru
```

------------------------------------------------------------------------

# 11. Tutup CMD

Setelah menjalankan `setx`:

1.  Tutup CMD.
2.  Buka CMD baru.

Sekarang jalankan:

``` cmd
echo %OPENCODE_9ROUTER_API_KEY%
```

Kalau muncul key:

``` text
sk-xxxxxxxx
```

berarti berhasil.

**Jangan kirim key tersebut ke orang lain.**

Lebih aman, cek panjang key saja.

------------------------------------------------------------------------

# 12. Cek Panjang API Key Tanpa Menampilkan Key

Jalankan:

``` cmd
powershell -Command "$env:OPENCODE_9ROUTER_API_KEY.Length"
```

Misalnya:

``` text
35
```

Artinya environment variable sudah terbaca.

Kalau hasil:

``` text
0
```

berarti environment belum masuk ke proses CMD.

------------------------------------------------------------------------

# 13. Kalau `echo` Tetap Tidak Menampilkan Key

Jangan langsung mengulang semuanya.

Cek apakah Windows benar-benar menyimpan key:

``` cmd
reg query HKCU\Environment /v OPENCODE_9ROUTER_API_KEY
```

Kalau ada:

``` text
OPENCODE_9ROUTER_API_KEY
REG_SZ
sk-xxxxxxxx
```

berarti key sudah tersimpan di Windows Registry.

**Jangan kirim nilai `sk-...` ke orang lain.**

------------------------------------------------------------------------

# 14. Kalau Registry Ada Tetapi Environment Masih 0

Ini kondisi yang sempat terjadi pada setup kita.

Registry:

``` text
API key ada        ✅
```

Tetapi:

``` powershell
$env:OPENCODE_9ROUTER_API_KEY.Length
```

menghasilkan:

``` text
0
```

Artinya proses terminal belum memuat variable tersebut.

Kita bisa mengambil variable dari Windows Registry ke environment proses
sekarang.

Di CMD jalankan:

``` cmd
for /f "tokens=2,*" %A in ('reg query HKCU\Environment /v OPENCODE_9ROUTER_API_KEY ^| findstr OPENCODE_9ROUTER_API_KEY') do set "OPENCODE_9ROUTER_API_KEY=%B"
```

Kemudian:

``` cmd
powershell -Command "$env:OPENCODE_9ROUTER_API_KEY.Length"
```

Kalau keluar:

``` text
35
```

berarti sudah terbaca.

------------------------------------------------------------------------

# 15. Cek Konfigurasi OpenCode

Jalankan:

``` cmd
opencode debug config
```

Yang perlu dicari:

``` json
"plugin": [
  "@vheins/opencode-9router@0.7.0"
]
```

dan provider 9Router yang muncul dari plugin.

Contoh hasil yang pernah berhasil:

``` json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": [
    "@vheins/opencode-9router@0.7.0"
  ],
  "provider": {
    "9router": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "9Router",
      "options": {
        "baseURL": "http://localhost:20128/v1"
      },
      "models": {
        "qd/qfmodel": {
          "name": "qd/qfmodel",
          "tool_call": true
        }
      },
      "api": "http://localhost:20128/v1"
    }
  }
}
```

API key tidak harus tampil di sini.

------------------------------------------------------------------------

# 16. Cek Model

Model yang berhasil ditemukan oleh 9Router:

``` text
qd/qfmodel
```

ID model yang digunakan OpenCode:

``` text
9router/qd/qfmodel
```

Tes:

``` cmd
opencode run --model 9router/qd/qfmodel "halo"
```

Kalau berhasil:

``` text
> build · qd/qfmodel

Halo! Apa kabar? Ada yang bisa saya bantu?
```

**SELESAI.**

------------------------------------------------------------------------

# 17. Kalau Muncul `Missing API key`

Kalau OpenCode berkata:

``` text
Missing API key
```

Jangan panik.

Pertama cek:

``` powershell
$env:OPENCODE_9ROUTER_API_KEY.Length
```

Kalau:

``` text
0
```

berarti API key tidak terbaca oleh proses OpenCode.

Kalau CMD biasa berhasil tetapi OpenCode yang dibuka dari shortcut
gagal, gunakan:

``` cmd
powershell -NoProfile -Command "$env:OPENCODE_9ROUTER_API_KEY=[Environment]::GetEnvironmentVariable('OPENCODE_9ROUTER_API_KEY','User'); opencode"
```

Perintah ini:

1.  mengambil API key dari Windows User Environment,
2.  memasukkannya ke environment proses,
3.  menjalankan OpenCode.

Tidak perlu mengetik ulang API key.

------------------------------------------------------------------------

# 18. Kenapa CMD Bisa Tetapi OpenCode Tidak?

Misalnya:

``` text
CMD
 ↓
OpenCode
 ↓
9Router
 ↓
BERHASIL
```

tetapi:

``` text
Start Menu / shortcut
 ↓
OpenCode
 ↓
Missing API key
```

Berarti kedua proses tersebut mendapatkan environment yang berbeda.

Bukan berarti:

``` text
9Router rusak
```

Bukan juga:

``` text
API key pasti salah
```

Kalau command dari CMD sudah berhasil, berarti koneksi 9Router sudah
terbukti bekerja.

------------------------------------------------------------------------

# 19. Cara Menjalankan OpenCode yang Aman

Kalau shortcut OpenCode masih tidak mendapatkan environment, gunakan:

``` cmd
powershell -NoProfile -Command "$env:OPENCODE_9ROUTER_API_KEY=[Environment]::GetEnvironmentVariable('OPENCODE_9ROUTER_API_KEY','User'); opencode"
```

Atau langsung:

``` cmd
powershell -NoProfile -Command "$env:OPENCODE_9ROUTER_API_KEY=[Environment]::GetEnvironmentVariable('OPENCODE_9ROUTER_API_KEY','User'); opencode run --model 9router/qd/qfmodel 'halo'"
```

------------------------------------------------------------------------

# 20. Kalau Muncul `Cache HIT`

Kalau muncul:

``` text
[9router-provider] [discovery] Cache HIT for http://localhost:20128 (1 models)
```

artinya plugin menemukan model dari cache.

Ini bukan error.

Contoh:

``` text
[9router-provider] [discovery] Cache HIT...
[9router-provider] [9router] Discovered 1 models...
```

Artinya model ditemukan.

Plugin memang mempunyai fitur cache discovery.

------------------------------------------------------------------------

# 21. Kalau Log Discovery Mengganggu

Kalau kamu menggunakan:

``` text
@vheins/opencode-9router@latest
```

bisa muncul log seperti:

``` text
[9router-provider] [discovery] Cache HIT...
[9router-provider] [9router] Discovered...
```

Pada setup tutorial ini kita menggunakan versi:

``` text
@vheins/opencode-9router@0.7.0
```

supaya perilakunya tetap sesuai setup yang sudah diuji.

------------------------------------------------------------------------

# 22. Kalau Muncul `Insufficient account funds`

Kalau muncul:

``` text
Upstream request failed: Insufficient account funds
```

ini bukan masalah OpenCode.

Bukan juga masalah plugin.

Artinya request sudah berhasil mencapai 9Router/upstream, tetapi
route/provider yang digunakan tidak mempunyai saldo atau kredit yang
diperlukan.

Alurnya:

``` text
OpenCode
   ↓
Plugin
   ↓
9Router
   ↓
Upstream provider
   ↓
Insufficient account funds
```

Jadi jangan menghapus plugin hanya karena pesan ini.

------------------------------------------------------------------------

# 23. Kalau `opencode.json` Rusak

JSON harus berbentuk object.

BENAR:

``` json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": [
    "@vheins/opencode-9router@0.7.0"
  ]
}
```

SALAH:

``` json
"plugin": {
  "9router": {}
}
```

SALAH:

``` json
{
  "plugin": [
    "@vheins/opencode-9router@0.7.0"
  ]
```

Karena kurung `}` terakhir hilang.

Kalau OpenCode berkata:

``` text
Config file is not valid JSON
```

periksa JSON terlebih dahulu.

------------------------------------------------------------------------

# 24. Jangan Mengubah Banyak Hal Sekaligus

Kalau terjadi error:

**Jangan langsung:**

-   reinstall OpenCode,
-   reinstall 9Router,
-   ganti API key,
-   hapus cache,
-   ganti model,
-   mengganti plugin,
-   mengedit semua config sekaligus.

Gunakan urutan:

``` text
1. Cek 9Router
2. Cek /v1/models
3. Cek API key
4. Cek environment
5. Cek plugin
6. Cek model
7. Tes OpenCode
```

Dengan begitu kita tahu bagian mana yang rusak.

------------------------------------------------------------------------

# 25. Checklist Super Simpel

Kalau nanti lupa semuanya, cukup ingat ini.

## A. 9Router hidup?

``` cmd
curl http://127.0.0.1:20128/v1/models
```

Harus ada model.

## B. API key ada?

``` cmd
reg query HKCU\Environment /v OPENCODE_9ROUTER_API_KEY
```

Harus ada.

## C. API key terbaca?

``` powershell
$env:OPENCODE_9ROUTER_API_KEY.Length
```

Harus bukan `0`.

## D. Plugin benar?

``` cmd
opencode debug config
```

Cari:

``` text
@vheins/opencode-9router@0.7.0
```

## E. Model bisa dipanggil?

``` cmd
opencode run --model 9router/qd/qfmodel "halo"
```

Kalau AI menjawab:

``` text
Halo!
```

berarti selesai.

------------------------------------------------------------------------

# 26. Konfigurasi Final

## `opencode.json`

``` json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": [
    "@vheins/opencode-9router@0.7.0"
  ]
}
```

## Environment

``` text
OPENCODE_9ROUTER_API_KEY = API KEY MILIKMU
OPENCODE_9ROUTER_URL = http://127.0.0.1:20128/v1
```

## Provider

``` text
9router
```

## Model

``` text
qd/qfmodel
```

## Model ID OpenCode

``` text
9router/qd/qfmodel
```

## Test

``` cmd
opencode run --model 9router/qd/qfmodel "halo"
```

------------------------------------------------------------------------

# 27. Diagram Akhir

``` text
                    WINDOWS
                       │
                       │
             OPENCODE_9ROUTER_API_KEY
                       │
                       ▼
                ┌─────────────┐
                │   OpenCode  │
                └──────┬──────┘
                       │
                       │ Plugin
                       ▼
          ┌─────────────────────────┐
          │ @vheins/opencode-9router│
          │         v0.7.0          │
          └───────────┬─────────────┘
                      │
                      │ HTTP
                      ▼
          ┌─────────────────────────┐
          │         9Router         │
          │ 127.0.0.1:20128/v1      │
          └───────────┬─────────────┘
                      │
                      ▼
               ┌─────────────┐
               │ qd/qfmodel  │
               └──────┬──────┘
                      │
                      ▼
                 AI RESPONSE
```

------------------------------------------------------------------------

# 28. Command Cheat Sheet

### Cek OpenCode

``` cmd
opencode --version
```

### Cek konfigurasi

``` cmd
opencode debug config
```

### Cek 9Router

``` cmd
curl http://127.0.0.1:20128/v1/models
```

### Simpan API key

``` cmd
setx OPENCODE_9ROUTER_API_KEY "API_KEY_KAMU"
```

### Simpan URL

``` cmd
setx OPENCODE_9ROUTER_URL "http://127.0.0.1:20128/v1"
```

### Cek API key tanpa menampilkannya

``` cmd
powershell -Command "$env:OPENCODE_9ROUTER_API_KEY.Length"
```

### Cek Registry

``` cmd
reg query HKCU\Environment /v OPENCODE_9ROUTER_API_KEY
```

### Jalankan OpenCode

``` cmd
opencode
```

### Jalankan model langsung

``` cmd
opencode run --model 9router/qd/qfmodel "halo"
```

### Jalankan OpenCode dengan environment dari Registry

``` cmd
powershell -NoProfile -Command "$env:OPENCODE_9ROUTER_API_KEY=[Environment]::GetEnvironmentVariable('OPENCODE_9ROUTER_API_KEY','User'); opencode"
```

------------------------------------------------------------------------

# 29. Kalau Besok Rusak Lagi

Jangan panik.

Copy-paste ini satu per satu:

``` cmd
curl http://127.0.0.1:20128/v1/models
```

``` cmd
reg query HKCU\Environment /v OPENCODE_9ROUTER_API_KEY
```

``` cmd
opencode debug config
```

``` cmd
opencode run --model 9router/qd/qfmodel "halo"
```

Kalau command terakhir berhasil:

``` text
SELAMAT.
9Router tidak rusak.
OpenCode tidak rusak.
Model tidak rusak.
```

Berarti masalahnya hanya pada cara OpenCode mendapatkan environment
variable.

------------------------------------------------------------------------

# 30. Keamanan API Key

API key adalah password untuk akses API.

Jangan:

``` text
❌ upload ke GitHub
❌ taruh di README
❌ kirim ke orang
❌ taruh di screenshot
❌ commit ke Git
❌ taruh langsung di opencode.json
```

Gunakan:

``` text
✅ Windows Environment Variable
```

Kalau key pernah terlanjur dipublikasikan atau dikirim ke tempat yang
tidak seharusnya:

1.  Revoke key lama.
2.  Buat key baru.
3.  Simpan key baru sebagai `OPENCODE_9ROUTER_API_KEY`.

------------------------------------------------------------------------

# 31. Referensi

-   OpenCode Plugins: https://opencode.ai/docs/plugins/
-   OpenCode Configuration: https://dev.opencode.ai/docs/config/
-   OpenCode CLI: https://dev.opencode.ai/docs/cli/
-   9Router OpenCode Plugin: https://github.com/vheins/opencode-9router

Dokumentasi OpenCode menjelaskan bahwa plugin npm dapat dimuat melalui
konfigurasi `opencode.json`, sedangkan repository
`@vheins/opencode-9router` menjelaskan auto-discovery model 9Router dan
konfigurasi provider.
