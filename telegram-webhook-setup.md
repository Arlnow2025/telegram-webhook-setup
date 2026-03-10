# Telegram Webhook Setup Guide

## Overview

This guide explains how to set up and configure Telegram webhooks for OpenClaw agents, including webhook mode activation, security configuration, and troubleshooting.

## Quick Setup Steps

### 1. Create Telegram Bot

1. Open Telegram and chat with **@BotFather**
2. Run `/newbot` and follow prompts
3. Save the bot token (you'll need it for configuration)

### 2. Configure Webhook in OpenClaw

Edit your `~/.openclaw/openclaw.json` configuration:

```json5
{
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "allowlist",
      "botToken": "your-bot-token-here",
      "webhookUrl": "https://your-domain.com/telegram-webhook",
      "webhookSecret": "your-webhook-secret",
      "webhookPath": "/telegram-webhook",
      "webhookPort": 8787
    }
  }
}
```

### 3. Start Gateway

```bash
openclaw gateway
```

### 4. Verify Webhook

OpenClaw will automatically attempt to set the webhook when the gateway starts. Check logs for:

```
Telegram webhook set successfully to: https://your-domain.com/telegram-webhook
```

## Security Configuration

### Webhook Security

1. **Webhook Secret**: Always set a strong webhook secret
2. **Domain**: Use HTTPS with valid SSL certificate
3. **IP Whitelisting**: Consider restricting to Telegram's IP ranges

### Access Control

```json5
{
  "channels": {
    "telegram": {
      "dmPolicy": "allowlist",
      "allowFrom": ["your-telegram-user-id"],
      "groupPolicy": "allowlist"
    }
  }
}
```

## Finding Your Telegram User ID

### Method 1: Bot Logs (Recommended)

1. DM your bot
2. Run: `openclaw logs --follow`
3. Look for: `from.id: 123456789`

### Method 2: Bot API

```bash
curl "https://api.telegram.org/bot<your-bot-token>/getUpdates"
```

### Method 3: Third-party Bots

Use @userinfobot or @getidsbot (less private)

## Webhook Troubleshooting

### Common Issues

#### 1. Webhook Not Setting

```bash
# Check gateway logs
openclaw logs --follow

# Test webhook endpoint manually
curl -X POST https://your-domain.com/telegram-webhook \
  -H "Content-Type: application/json" \
  -d '{"test": "payload"}'
```

#### 2. SSL Certificate Issues

Ensure your domain has a valid SSL certificate. Telegram requires HTTPS.

#### 3. Firewall/DNS Issues

```bash
# Test Telegram API connectivity
curl https://api.telegram.org/bot<your-bot-token>/getMe

# Check DNS resolution
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

### Gateway Configuration

```json5
{
  "gateway": {
    "port": 18789,
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "your-gateway-token"
    }
  }
}
```

## Multi-Agent Webhook Setup

### Per-Agent Configuration

```json5
{
  "channels": {
    "telegram": {
      "accounts": {
        "main": {
          "webhookUrl": "https://main-agent.your-domain.com/telegram-webhook"
        },
        "support": {
          "webhookUrl": "https://support-agent.your-domain.com/telegram-webhook"
        }
      }
    }
  }
}
```

### Agent-Specific Access

```json5
{
  "agents": {
    "list": [
      {
        "id": "main",
        "channels": {
          "telegram": {
            "allowFrom": ["your-user-id"],
            "groupPolicy": "allowlist"
          }
        }
      },
      {
        "id": "support",
        "channels": {
          "telegram": {
            "allowFrom": ["support-user-id"],
            "groupPolicy": "allowlist"
          }
        }
      }
    ]
  }
}
```

## Webhook vs Long Polling

### Webhook Mode (Recommended for Production)

**Pros:**
- Real-time message delivery
- Lower latency
- Better resource utilization

**Cons:**
- Requires public HTTPS endpoint
- SSL certificate needed
- More complex setup

### Long Polling Mode

**Pros:**
- Simpler setup
- Works behind firewalls
- No public endpoint needed

**Cons:**
- Higher latency
- More resource usage
- Polling intervals

## Advanced Configuration

### Custom Webhook Path

```json5
{
  "channels": {
    "telegram": {
      "webhookPath": "/custom-telegram-webhook",
      "webhookPort": 8080
    }
  }
}
```

### Proxy Configuration

```json5
{
  "channels": {
    "telegram": {
      "proxy": "socks5://localhost:9050",
      "webhookUrl": "https://your-domain.com/telegram-webhook"
    }
  }
}
```

### Network Configuration

```json5
{
  "channels": {
    "telegram": {
      "network": {
        "autoSelectFamily": true,
        "dnsResultOrder": "ipv4first"
      }
    }
  }
}
```

## Testing Webhook

### Test Script

```bash
#!/bin/bash
# test-telegram-webhook.sh

WEBHOOK_URL="https://your-domain.com/telegram-webhook"
WEBHOOK_SECRET="your-webhook-secret"

# Test webhook endpoint
curl -X POST "$WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $WEBHOOK_SECRET" \
  -d '{
    "update_id": 123456789,
    "message": {
      "message_id": 1,
      "from": {
        "id": 123456789,
        "is_bot": false,
        "first_name": "Test",
        "last_name": "User",
        "username": "testuser",
        "language_code": "en"
      },
      "chat": {
        "id": 123456789,
        "first_name": "Test",
        "last_name": "User",
        "username": "testuser",
        "type": "private"
      },
      "date": 1234567890,
      "text": "Hello, webhook!"
    }
  }'
```

## GitHub Upload

### Create Repository

```bash
gh repo create telegram-webhook-setup --public
```

### Upload Documentation

```bash
# Create README.md with this content
gh repo create telegram-webhook-setup --public

# Initialize git
git init
git add README.md
git commit -m "Initial commit: Telegram webhook setup guide"
git branch -M main
git remote add origin https://github.com/Arlnow2025/telegram-webhook-setup.git
git push -u origin main
```

### Alternative: Single File Upload

```bash
# Create file
gh gist create README.md --public --description "Telegram webhook setup guide"
```

## Monitoring and Maintenance

### Log Monitoring

```bash
# Monitor Telegram webhook logs
openclaw logs --follow | grep -i telegram

# Check webhook status
openclaw channels status --probe
```

### Health Checks

```bash
# Test webhook endpoint
curl -I https://your-domain.com/telegram-webhook

# Check Telegram bot status
curl https://api.telegram.org/bot<your-bot-token>/getMe
```

## Best Practices

1. **Always use HTTPS** for webhook endpoints
2. **Set strong webhook secrets**
3. **Monitor logs regularly**
4. **Test webhook functionality after changes**
5. **Keep bot tokens secure**
6. **Use proper access control**
7. **Regular backups of configuration**
8. **Monitor webhook delivery success rates**

## Resources

- [OpenClaw Telegram Documentation](https://docs.openclaw.ai/channels/telegram)
- [Telegram Bot API Documentation](https://core.telegram.org/bots/api)
- [OpenClaw Webhook Documentation](https://docs.openclaw.ai/automation/webhook)
- [GitHub Repository](https://github.com/Arlnow2025/telegram-webhook-setup)

---

*Last updated: March 2026*