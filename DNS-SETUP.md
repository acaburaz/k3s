# DNS Configuration for gamifying.dev

Configure these records in GoDaddy for the mail server to work properly.

## DNS Records

| Type | Name | Value | TTL | Notes |
|------|------|-------|-----|-------|
| A | `mail` | `85.194.34.199` | 600 | Mail server hostname |
| A | `webmail` | `85.194.34.199` | 600 | Webmail (Roundcube) |
| MX | `@` (root) | `mail.gamifying.dev` | 3600 | Priority: 10 |
| TXT | `@` (root) | `v=spf1 mx a ip4:85.194.34.199 ~all` | 3600 | SPF policy |
| TXT | `_dmarc` | `v=DMARC1; p=quarantine; rua=mailto:postmaster@gamifying.dev` | 3600 | DMARC policy |
| TXT | `mail._domainkey` | (see DKIM section below) | 3600 | DKIM public key |

## DKIM Setup

After deploying docker-mailserver, generate DKIM keys:

```bash
sudo k3s kubectl exec -n mailcow dms-0 -- setup config dkim domain gamifying.dev
```

Extract the public key:

```bash
sudo k3s kubectl exec -n mailcow dms-0 -- cat /etc/dms/config/opendkim/keys/gamifying.dev/mail.txt
```

The output will look like:
```
mail._domainkey.gamifying.dev. IN TXT "v=DKIM1; h=sha256; k=rsa; p=MIGfMA0GCS..."
```

Copy the part after `p=` and create a TXT record in GoDaddy:
- **Name:** `mail._domainkey`
- **Value:** `v=DKIM1; h=sha256; k=rsa; p=MIGfMA0GCS...` (the full p= value)

## PTR (Reverse DNS) Record

Contact your VPS/hosting provider to set the reverse DNS (PTR) record for IP `85.194.34.199` to point to `mail.gamifying.dev`. This is critical for mail deliverability.

## Postmaster Configuration

| Field | Value |
|-------|-------|
| Email | `postmaster@gamifying.dev` |
| Password | `mom2nbfYnLIt6fdWZtWaIw==` |
| Webmail | `https://webmail.gamifying.dev` |

## Testing

After all DNS records are set and propagated (5-30 minutes):

1. **SMTP test:**
   ```bash
   telnet 85.194.34.199 25
   ```
   Should show ESMTP banner.

2. **Webmail access:**
   Navigate to `https://webmail.gamifying.dev`

3. **Mail deliverability:**
   Use [mail-tester.com](https://www.mail-tester.com) to check your configuration after sending a test email.

4. **Check propagation:**
   ```bash
   nslookup -type=MX gamifying.dev
   nslookup -type=TXT gamifying.dev
   ```
