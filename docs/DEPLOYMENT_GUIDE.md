# test-net — Production Deployment Guide

Dokumen ini adalah panduan operasional resmi untuk melakukan deployment sistem **test-net** (Frontend Next.js App Router dan Dedicated LibreSpeed Measurement Server) ke lingkungan production.

---

## 1. Production Architecture Overview

`test-net` menerapkan **Decoupled Deployment Model**:

```
                       [ User Browser (HTTPS) ]
                                   │
                 ┌─────────────────┴─────────────────┐
                 │                                   │
      1. Web UI & Assets                    2. Measurement Traffic
   (HTML, JS, CSS, Metadata)                 (Ping, Download, Upload)
                 │                                   │
                 ▼                                   ▼
        [ Next.js Frontend ]                [ Dedicated Server ]
         Vercel / Cloudflare                     Linux VPS
      (https://test-net.com)             (https://speed.test-net.com)
                                                     │
                                            ┌────────┴────────┐
                                            ▼                 ▼
                                      Nginx Proxy       speedtest-go
                                    (TLS, CORS, Rate)  (Port 8080 Core)
```

---

## 2. Prerequisites & Resource Allocation

### 2.1. Frontend Web Hosting
* **Platform**: Vercel, Cloudflare Pages, atau Node.js VPS.
* **Node.js**: >= 20 LTS.
* **Domain**: `test-net.yourdomain.com` atau apex domain `yourdomain.com`.

### 2.2. Dedicated Speed Test Server (Data Plane)
* **Hosting**: Linux VPS (Ubuntu 22.04 LTS atau Debian 12).
* **Provider Rekomendasi**: Hetzner, DigitalOcean, Linode, AWS EC2, atau VPS lokal Indonesia (Biznet Gio/IDCloudHost) dengan latensi terdekat ke target audiens.
* **Spesifikasi**: Minimal 1 vCPU, 1 GB RAM, 25 GB SSD.
* **Port Jaringan**: Minimal **1 Gbps Unmetered** (atau bandwidth allocation minimal 2–10 TB/bulan).
* **Firewall Ports Open**:
  * `80/tcp` (HTTP untuk sertifikat SSL Let's Encrypt / ACME challenge)
  * `443/tcp` (HTTPS untuk trafik speed test aman)
  * `22/tcp` (SSH manajemen server dengan key-based auth)

---

## 3. Domain & DNS Configuration

| Tipe DNS | Host / Name | Target / Value | Keterangan |
| :--- | :--- | :--- | :--- |
| **CNAME** | `@` atau `www` | `cname.vercel-dns.com` (atau provider frontend) | Frontend Next.js Web App |
| **A** | `speed` | `[ALAMAT_IP_VPS]` | Dedicated Measurement Server |

---

## 4. Production Environment Variables

### 4.1. Frontend Environment (`.env.production` / Vercel Dashboard)
```env
# URL Dedicated Server Pengujian Produksi (Wajib HTTPS)
NEXT_PUBLIC_SPEEDTEST_SERVER_URL=https://speed.yourdomain.com

# Nama Tampilan Server
NEXT_PUBLIC_SPEEDTEST_SERVER_NAME="Singapore Gigabit Server"

# Durasi Uji (Detik)
NEXT_PUBLIC_TEST_DURATION_DL=15
NEXT_PUBLIC_TEST_DURATION_UL=15
NEXT_PUBLIC_TEST_PING_SAMPLES=10
```

---

## 5. Deployment Frontend (Next.js)

### Opsi A: Deployment via Vercel (Rekomendasi)
1. Hubungkan repositori GitHub `test-net` ke dashboard Vercel.
2. Atur **Root Directory** ke `test-net` jika berada di dalam subdirektori.
3. Masukkan Environment Variables di atas.
4. Klik **Deploy**. Vercel akan otomatis mengompilasi dan mengaktifkan SSL di CDN global.

### Opsi B: Self-Hosted Node.js via PM2 / Systemd
```bash
cd test-net
npm ci --omit=dev
npm run build
pm2 start npm --name "test-net-frontend" -- start
```

---

## 6. Deployment Dedicated Measurement Server

### 6.1. Persiapan VPS
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl ufw certbot

# Konfigurasi Firewall
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

### 6.2. Penerbitan Sertifikat SSL Let's Encrypt
```bash
sudo certbot certonly --standalone -d speed.yourdomain.com --agree-tos -m admin@yourdomain.com --no-eff-email
```

### 6.3. Menjalankan Docker Compose
Salin direktori `infra/speedtest-server/` ke VPS di `/opt/speedtest-server`:
```bash
mkdir -p /opt/speedtest-server/certs
sudo cp /etc/letsencrypt/live/speed.yourdomain.com/fullchain.pem /opt/speedtest-server/certs/
sudo cp /etc/letsencrypt/live/speed.yourdomain.com/privkey.pem /opt/speedtest-server/certs/

cd /opt/speedtest-server
# Sesuaikan domain di nginx.conf jika diperlukan
docker compose pull
docker compose up -d
docker compose ps
```

---

## 7. Production Smoke Testing

Setelah deployment selesai, jalankan perintah pengujian konektivitas berikut dari terminal eksternal:

```bash
# 1. Frontend Smoke Test
curl -I https://test-net.yourdomain.com
# Response wajib: HTTP/2 200

# 2. Dedicated Server Health Check
curl -i https://speed.yourdomain.com/
# Response wajib: {"status":"online","service":"test-net dedicated measurement server"}

# 3. Ping Endpoint Verification
curl -i https://speed.yourdomain.com/empty
# Response wajib: HTTP/2 200, Content-Type text/plain, Cache-Control no-store

# 4. CORS Preflight Check
curl -X OPTIONS -i https://speed.yourdomain.com/empty \
  -H "Origin: https://test-net.yourdomain.com" \
  -H "Access-Control-Request-Method: POST"
# Response wajib: HTTP/2 204 No Content, Access-Control-Allow-Origin matching frontend

# 5. Download Stream Check (5MB chunk)
curl -o /dev/null -w "Size: %{size_download} bytes, Speed: %{speed_download} B/s\n" \
  https://speed.yourdomain.com/garbage?ckSize=5
```

---

## 8. Monitoring & Maintenance

* **Health Check Endpoint**: `https://speed.yourdomain.com/` dapat dipantau menggunakan Uptime Kuma, BetterStack, atau StatusCake.
* **Auto Renewal SSL**:
  Tambahkan cronjob untuk pembaharuan sertifikat otomatis:
  ```bash
  0 3 1 * * certbot renew --quiet && docker compose -f /opt/speedtest-server/docker-compose.yml restart proxy
  ```
* **Bandwidth Monitoring**: Pasang `vnstat` pada VPS untuk memantau akumulasi konsumsi kuota bandwidth bulanan.

---

## 9. Rollback Procedure

Jika ditemukan isu performa kritis setelah rilis:
1. **Frontend Rollback**:
   * Pada dashboard Vercel, buka menu *Deployments*, pilih commit rilis sebelumnya yang stabil (`phase 9`), lalu klik **Instant Rollback**.
2. **Dedicated Server Rollback**:
   * Jalankan `docker compose down` di VPS, lalu revert file konfigurasi Nginx ke versi stabil sebelumnya:
     ```bash
     cd /opt/speedtest-server
     docker compose restart
     ```
