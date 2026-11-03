# test-net — Release Notes v1.0.0 (MVP)

**Version:** `v1.0.0`  
**Release Date:** November 2026  
**Status:** General Availability (Production MVP)  

---

## 1. Overview
`test-net` v1.0.0 adalah rilis perdana aplikasi web publik untuk pengujian performa dan stabilitas koneksi internet secara langsung dari browser pengguna tanpa memerlukan pendaftaran akun. Aplikasi ini dibangun dengan standar arsitektur modern yang memisahkan frontend ringan dari server pengukuran data plane berkapasitas gigabit.

---

## 2. Key Features

### 2.1. P0 — Core Speed Test Engine
* **Real-time Download Test**: Mengukur throughput download aktual menggunakan 3 koneksi paralel multi-stream dengan data stream uncompressed acak.
* **Real-time Upload Test**: Mengukur kecepatan upload aktual melalui streaming payload via XMLHttpRequest berurutan.
* **HTTP Ping (Round-Trip Latency)**: Mengukur latency RTT HTTP secara akurat (10 sampel) dengan kalkulasi nilai median.
* **RFC 3550 Jitter Metric**: Menghitung stabilitas latency koneksi berdasarkan deviasi selisih absolut rata-rata.
* **Interactive Circular Gauge**: Tampilan speedometer radial SVG responsif dengan aksen warna dinamis sesuai fase pengujian.
* **Test Lifecycle State Machine**: Pengendalian fase pengujian yang kokoh (*Idle, Preparing, Ping, Download, Upload, Completed, Cancelling, Cancelled, Error*).
* **Instant Cancellation & Retry**: Pembatalan pengujian instan menggunakan `AbortController` yang seketika memutus transfer byte.
* **Data Usage Tracking**: Perhitungan dan tampilan real-time jumlah kuota data yang dikonsumsi per pengujian.

### 2.2. P1 — Supporting Pages & Enhancements
* **About & Methodology Page (`/about`)**: Edukasi teknis mengenai metodologi pengukuran, perbandingan HTTP latency vs ICMP ping, dan atribusi lisensi open-source LibreSpeed (LGPL-3.0).
* **Frequently Asked Questions (`/faq`)**: Panduan pemecahan masalah koneksi, penjelasan unit Mbps vs MB/s, dan tips akurasi pengujian.
* **Privacy Policy (`/privacy`)**: Kebijakan privasi transparan (bebas akun, tanpa iklan, dan tanpa penyimpanan riwayat di database server).
* **One-Click Share Result**: Tombol salin hasil pengujian ke clipboard dalam format teks terstruktur.
* **Local Test History**: Riwayat pengujian tersimpan aman murni di peramban pengguna (`localStorage`) menggunakan hook React 19 `useSyncExternalStore` dengan opsi hapus instan.
* **Full Responsive & Dark Theme**: Desain futuristik dengan kontras tinggi, ramah aksesibilitas WCAG 2.1 AA, dan font tabular numerik (`font-mono tabular-nums`).

---

## 3. Architecture Summary
* **Presentation Layer**: Next.js 16 (App Router) + TypeScript + Tailwind CSS v4 + shadcn/ui. Seluruh rute publik di-prerender secara statis (SSG).
* **Measurement Data Plane**: Dedicated LibreSpeed measurement server berbasis `speedtest-go` / container resmi dengan Nginx reverse proxy (TLS 1.3, rate limiting 10 req/s, dan CORS allowlist).
* **Zero Next.js Route Handlers**: Tidak ada payload byte pengujian yang dialirkan melalui server Next.js serverless.

---

## 4. Known Limitations
1. **HTTP RTT vs ICMP Ping**: Browser web tidak memiliki akses socket mentah untuk ICMP echo request, sehingga latency yang terukur adalah HTTP round-trip latency ke port 443 server pengujian.
2. **Ketergantungan Jaringan Lokal**: Pengujian via WiFi dapat dipengaruhi oleh interferensi frekuensi atau jarak router fisik pengguna.

---

## 5. Operational Notes for Administrators
* **Bandwidth Management**: Pastikan VPS dedicated server memiliki kuota bandwidth bulanan yang memadai (minimal 2 TB - 10 TB/bulan) dengan port minimal 1 Gbps.
* **Auto Renewal SSL**: Pastikan cron job Let's Encrypt Certbot aktif di VPS agar koneksi HTTPS port 443 tidak terputus.
* **CORS Origin**: Jika memindahkan frontend ke custom domain baru, pastikan memperbarui peta origin `$cors_origin` pada `nginx.conf` di server pengujian.

---

*test-net Team — Built with Precision.*
