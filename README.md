# WhatsApp MCP Server

Server Model Context Protocol (MCP) untuk WhatsApp yang memungkinkan AI agent membaca dan mengirim pesan WhatsApp.

## Fitur

- 🔍 Cari dan baca pesan WhatsApp (termasuk media)
- 👥 Cari kontak
- 💬 Kirim pesan ke individu atau grup
- 📎 Kirim file (gambar, video, dokumen, audio)
- 🔒 Mendukung deployment remote dengan keamanan

## Arsitektur

```
┌──────────────┐     ┌──────────────────┐     ┌──────────────┐
│  MCP Client  │────▶│  WhatsApp MCP    │────▶│   WhatsApp   │
│  (Claude/    │     │     Server       │     │   Bridge     │
│  AnythingLLM)│◀────│   (Python)       │◀────│    (Go)      │
└──────────────┘     └──────────────────┘     └──────────────┘
```

## Instalasi

### Metode 1: Docker (Direkomendasikan)

```bash
# Clone repository
git clone https://github.com/lharries/whatsapp-mcp.git
cd whatsapp-mcp

# Autentikasi WhatsApp (scan QR code)
docker compose run --rm -it whatsapp-bridge

# Jalankan semua service
docker compose up -d
```

### Metode 2: Manual

Lihat bagian [Instalasi Manual](#instalasi-manual) di bawah.

## Konfigurasi MCP Client

### Untuk Claude Desktop / Cursor (Lokal)

```json
{
  "mcpServers": {
    "whatsapp": {
      "command": "uv",
      "args": ["--directory", "/path/to/whatsapp-mcp-server", "run", "main.py"]
    }
  }
}
```

### Untuk AnythingLLM Docker (Remote via SSE)

```json
{
  "mcpServers": {
    "whatsapp": {
      "type": "sse",
      "url": "https://whatsapp-mcp.domain-anda.com/sse",
      "headers": {
        "X-API-Key": "your-api-key-here"
      }
    }
  }
}
```

## Deployment Remote dengan Keamanan

Untuk menjalankan MCP server di server terpisah dari MCP client:

### 1. Konfigurasi Docker Network

```bash
# Buat network untuk reverse proxy
docker network create web

# Jalankan service
docker compose up -d
```

### 2. Konfigurasi Reverse Proxy (Caddy)

Lihat contoh di `Caddyfile.example`:

```caddyfile
whatsapp-mcp.domain-anda.com {
    @allowed remote_ip <IP_CLIENT_ANDA>
    handle @allowed {
        reverse_proxy whatsapp-mcp:8000 {
            flush_interval -1
        }
    }
    respond "Forbidden" 403
}
```

### 3. Keamanan (Defense-in-Depth)

```bash
# Generate API Key
openssl rand -hex 32

# Simpan di file .env
echo "MCP_API_KEY=your-generated-key" > .env
```

- ✅ **TLS/HTTPS** - Otomatis dengan Caddy
- ✅ **IP Whitelist** - Hanya izinkan IP client yang dikenal
- ✅ **API Key** - Validasi header X-API-Key
- ✅ **Container Hardening** - no-new-privileges, cap_drop ALL

## MCP Tools

| Tool | Deskripsi |
|------|-----------|
| `search_contacts` | Cari kontak berdasarkan nama/nomor |
| `list_messages` | Ambil pesan dengan filter |
| `list_chats` | Daftar chat |
| `send_message` | Kirim pesan teks |
| `send_file` | Kirim file media |
| `download_media` | Download media dari pesan |

## Instalasi Manual

### Prasyarat

- Go 1.21+
- Python 3.11+
- UV package manager
- FFmpeg (opsional, untuk audio)

### Langkah

1. **Jalankan WhatsApp Bridge**

   ```bash
   cd whatsapp-bridge
   go run main.go
   # Scan QR code dengan WhatsApp mobile
   ```

2. **Jalankan MCP Server**

   ```bash
   cd whatsapp-mcp-server
   uv run main.py
   ```

### Windows

Aktifkan CGO untuk go-sqlite3:

```bash
go env -w CGO_ENABLED=1
```

## Troubleshooting

- **QR Code tidak muncul**: Restart bridge
- **Pesan tidak loading**: Tunggu beberapa menit setelah autentikasi
- **Out of sync**: Hapus file `store/*.db` dan autentikasi ulang

## Lisensi

MIT License
