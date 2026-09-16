# TREM-KU Server AI Infrastructure

Deployment TREM-KU untuk Docker Desktop pada Windows 11. Stack terdiri dari Ollama, Qdrant, EMQX, Redis, Agent API, dan Voice Service.

## Prasyarat

- Windows 11 dengan Docker Desktop dan backend WSL2.
- Driver NVIDIA serta dukungan GPU Docker aktif.
- Repositori `serverAI` dan `tremagent` berada sebagai folder sejajar di drive yang sama.
- Port lokal `3100`, `8000`, `11434`, `6333`, `6379`, `1883`, `8083`, dan `18083` belum digunakan aplikasi lain.

Struktur yang digunakan:

```text
D:\serverAI
D:\tremagent
D:\ollama_cache
D:\tremku_models\huggingface
```

## Menjalankan untuk Pertama Kali

Jalankan PowerShell:

```powershell
Set-Location D:\serverAI
Copy-Item .env.example .env
Copy-Item emqx_bootstrap_users.example.csv emqx_bootstrap_users.csv
```

Ganti seluruh nilai `change-me` di `.env`. Password pada `emqx_bootstrap_users.csv` harus sama dengan `MQTT_PASSWORD` di `.env`. Setelah itu:

```powershell
docker compose config --quiet
docker compose up -d --build
docker compose exec -T ollama ollama pull qwen3.8:27b
docker compose exec -T ollama ollama pull nomic-embed-text
docker compose exec -T agent-api npm run seed:heritage
docker compose ps
```

Opsional, siapkan cache Whisper sebelum percakapan suara pertama:

```powershell
docker compose exec -T voice python -c "import asyncio, service; asyncio.run(service.get_stt_model())"
```

## Operasi Harian

```powershell
# Menjalankan service
docker compose up -d

# Melihat status
docker compose ps

# Melihat log
docker compose logs -f agent-api voice emqx

# Membangun ulang setelah perubahan kode
docker compose up -d --build agent-api voice

# Menghentikan service tanpa menghapus data
docker compose stop

# Menghidupkan kembali
docker compose start

# Menghapus container dan network tanpa menghapus volume bind
docker compose down
```

## Endpoint Lokal

- Agent health: `http://127.0.0.1:3100/health`
- Agent chat: `http://127.0.0.1:3100/api/chat`
- Dashboard KOMANDO: `http://127.0.0.1:3100/dashboard`
- Voice DINUS: `http://127.0.0.1:8000`
- EMQX dashboard: `http://127.0.0.1:18083`
- Ollama internal: `http://127.0.0.1:11434`

Seluruh port hanya terikat ke loopback Windows. Jangan membuka Ollama, Redis, Qdrant, atau MQTT langsung ke internet.

## Cloudflare Tunnel dan Access

Hostname publik Agent API:

```text
https://aiapi.elektrodinus.id/api/chat
```

Tunnel pada Windows diarahkan ke `http://localhost:3100`. Jika `cloudflared` dijalankan sebagai service dalam Compose yang sama, gunakan `http://agent-api:3100`.

Klien antarmesin memerlukan empat konfigurasi berikut:

```env
TREMKU_API_URL=https://aiapi.elektrodinus.id/api/chat
TREMKU_API_KEY=nilai_AGENT_API_KEY_server
CF_ACCESS_CLIENT_ID=client_id_service_token
CF_ACCESS_CLIENT_SECRET=client_secret_service_token
```

Header request:

```http
Content-Type: application/json
x-api-key: TREMKU_API_KEY
CF-Access-Client-Id: CF_ACCESS_CLIENT_ID
CF-Access-Client-Secret: CF_ACCESS_CLIENT_SECRET
```

Jangan memberikan file `.env` server atau Cloudflare Tunnel Token kepada klien. Buat Service Token terpisah untuk setiap perangkat agar akses dapat dicabut secara mandiri.

## Pemeriksaan Kesehatan

```powershell
Invoke-RestMethod http://127.0.0.1:3100/health
Invoke-RestMethod http://127.0.0.1:8000/health
docker compose ps
```

Respons Agent API berstatus sehat jika MQTT, Redis, Qdrant, dan Ollama seluruhnya tersedia.

## Data Persisten

- Ollama: `D:/ollama_cache`
- Faster Whisper: `D:/tremku_models/huggingface`
- Qdrant: `./qdrant_data`
- Redis: `./redis_data`
- EMQX: `./emqx_data`
- Log agent dan briefing: `../tremagent/tremku-agents/data`

## Keamanan

- Secret lokal hanya disimpan pada `.env` dan file bootstrap yang diabaikan Git.
- MQTT menolak koneksi anonim dan menggunakan built-in database EMQX.
- Agent API memerlukan `x-api-key`; endpoint publik ditambah Cloudflare Access Service Token.
- Port host dibatasi ke `127.0.0.1`.
- Rotasi API key dan Service Token secara berkala atau segera setelah dicurigai bocor.

## Changelog

### 2026-09-16

- Mengubah deployment menjadi stack Docker Desktop Windows 11 yang lengkap.
- Menambahkan service Agent API dan Voice Service ke Compose.
- Mengaktifkan reservasi seluruh GPU NVIDIA untuk Ollama.
- Menambahkan health check dan urutan startup dependency.
- Mengaktifkan autentikasi MQTT, bootstrap user EMQX, authorization fail-closed, dan node cookie khusus.
- Mengaktifkan password Redis dan API key Agent API.
- Membatasi seluruh port host ke loopback Windows.
- Menambahkan volume persisten untuk Ollama, Whisper, Qdrant, Redis, EMQX, dan data agent.
- Memvalidasi Cloudflare Tunnel pada `aiapi.elektrodinus.id`, termasuk health check, penolakan request tanpa API key, dan request chat terautentikasi.
- Mendokumentasikan Cloudflare Access Service Token untuk akses API dari PC lain.
