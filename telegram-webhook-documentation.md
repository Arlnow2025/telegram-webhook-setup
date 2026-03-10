# Dokumentasi Lengkap Webhook Telegram di VPS

## Status Sistem
- **VPS Provider**: OpenCloudOS
- **IP Address**: 43.163.118.107
- **Domain**: server-openclaw.my.id
- **SSL Certificate**: LetsEncrypt (valid 86 hari)

## Layanan yang Berjalan

### 1. OpenClaw Gateway
- **Port**: 18789 (bind 127.0.0.1)
- **Auth**: Token-based (di-setup di ~/.openclaw/openclaw.json)
- **Status**: Aktif
- **Health Check**: OK

### 2. Telegram Webhook
- **Bot Token**: `YOUR_BOT_TOKEN_HERE` (contoh: 8517356401:AAHoEUDM75bn4klXxSF2gyeSZEt43f4c4MU)
- **Webhook URL**: https://server-openclaw.my.id/telegram-webhook
- **Listening Port**: 8787 (bind 127.0.0.1)
- **Status**: Aktif
- **Telegram API**: Konfigurasi berhasil

### 3. Nginx (Reverse Proxy)
- **Port**: 80/443
- **SSL**: Enabled (TLSv1.2, TLSv1.3)
- **Log**: `/var/log/nginx/telegram-access.log`, `/var/log/nginx/telegram-error.log`

## Konfigurasi Nginx

### File: `/etc/nginx/conf.d/telegram-openclaw.conf`
```nginx
# OpenClaw Webhook (port 8787)
location /telegram-webhook {
    proxy_pass http://127.0.0.1:8787/telegram-webhook;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_pass_header X-Telegram-Bot-Api-Secret-Token;
    proxy_connect_timeout 60s;
    proxy_send_timeout 120s;
    proxy_read_timeout 120s;
}

# OpenClaw Gateway (semua traffic)
location / {
    proxy_pass http://127.0.0.1:18789;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_read_timeout 86400;
}
```

### File: `/etc/nginx/telegram-webhook.conf`
```nginx
location /telegram/webhook {
    proxy_pass http://127.0.0.1:18789/telegram/webhook;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

## Konfigurasi OpenClaw

### File: `/root/.openclaw/openclaw.json` (bagian channels.telegram)
```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "allowlist",
      "botToken": "YOUR_BOT_TOKEN_HERE",
      "allowFrom": ["6350718807"],
      "webhookUrl": "https://server-openclaw.my.id/telegram-webhook",
      "webhookPath": "/telegram-webhook",
      "webhookPort": 8787
    }
  }
}
```

## SSL Certificate
- **Domain**: server-openclaw.my.id
- **Serial**: 6e91c491a114725c8d9e4144adba124a675
- **Type**: ECDSA
- **Expiry**: 2026-06-04 (86 hari lagi)
- **Paths**: `/etc/letsencrypt/live/server-openclaw.my.id/fullchain.pem`

## GitHub Repository
- **URL**: https://github.com/Arlnow2025/telegram-webhook-setup
- **Status**: Tersedia untuk push
- **Authentication**: GitHub PAT (di-setup di ~/.config/gh)

## Aktivasi Webhook Telegram

### Langkah-langkah:
1. **Verifikasi Webhook**: `curl https://api.telegram.org/botYOUR_BOT_TOKEN_HERE/getWebhookInfo`
2. **Nginx Reload**: `systemctl reload nginx`
3. **OpenClaw Status**: `openclaw status`
4. **Health Check**: `curl http://127.0.0.1:18789/health`

### Testing:
```bash
# Test webhook endpoint (ganti YOUR_BOT_TOKEN_HERE)
curl -X POST https://server-openclaw.my.id/telegram-webhook -d '{
    "update_id": 12345,
    "message": {
        "message_id": 1,
        "from": {"id": 6350718807, "is_bot": false, "first_name": "Drake"},
        "chat": {"id": 6350718807, "first_name": "Drake", "type": "private"},
        "text": "Hello"
    }
}'
```

## Troubleshooting

### Common Issues:
1. **Webhook Not Responding**: Periksa firewall, nginx config, OpenClaw status
2. **SSL Error**: Verifikasi certificate path dan expiry
3. **Port Conflict**: Pastikan 8787 dan 18789 tidak digunakan aplikasi lain

### Logs Location:
- Nginx: `/var/log/nginx/telegram-*.log`
- OpenClaw: `/tmp/openclaw/openclaw-*.log`
- System: `journalctl -u nginx -u openclaw-gateway`

## Security Configuration
- **Gateway**: Loopback binding (127.0.0.1)
- **Auth**: Token-based authentication
- **SSL**: Full encryption (TLS 1.2/1.3)
- **DM Policy**: Allowlist (hanya 6350718807)
- **Nginx**: Custom headers dan timeout settings

---

*Dokumentasi ini dihasilkan otomatis pada 2026-03-10 21:00 UTC. Status sistem aktif dan berfungsi dengan baik.*