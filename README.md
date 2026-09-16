# TREM-KU Server AI Infrastructure

Repositori ini mengatur infrastruktur docker (*containers*) untuk ekosistem AI TREM-KU.

## CHANGELOG
### [2026-09-16]
- **docs:** Konfigurasi broker EMQX tervalidasi pada port 1883 (TCP lokal) dan 8083 (WebSockets).
- **docs:** Menambahkan catatan arsitektur bahwa akses *telemetry* dari Edge (Trem) menuju cloud harus dirutekan menggunakan **MQTT over WebSockets (WSS)** melalui layanan Cloudflare Zero Trust (Tunnels).
- **infra:** Layanan Qdrant, Ollama, dan Redis telah disiapkan dan diekspos secara lokal untuk dikonsumsi oleh `tremagent` API.
