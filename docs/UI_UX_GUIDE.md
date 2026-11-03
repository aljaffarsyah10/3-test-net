# test-net — UI/UX Planning & Design System Guide

Dokumen ini adalah pedoman komprehensif desain visual, wireframe, design tokens, interaksi antarmuka (UI states), komponen yang dapat digunakan kembali (*reusable components*), dan standar aksesibilitas untuk implementasi frontend **test-net**.

---

## 1. Design Direction & Visual Identity

### 1.1. Filosofi Desain
* **Scientific & Modern Minimalist**: Menghadirkan antarmuka pengujian jaringan yang bersih, elegan, presisi, dan bebas distraksi layaknya instrumen laboratorium digital modern.
* **Fokus Hierarki Visual pada Angka**: Pusat perhatian pengguna adalah metrik angka aktual (Download, Upload, Ping, Jitter).
* **Anti-Clutter & Single Primary Action**: Tidak ada banner iklan, popup, atau elemen dekoratif berlebihan. Hanya ada satu CTA utama yang dominan pada setiap fase (GO / Cancel / Test Again).
* **High Contrast Dark/Light Theme**: Memanfaatkan palet dark mode yang nyaman di mata dengan aksen neon fungsional untuk menunjukkan alur data aktif.

### 1.2. Identity & Brand Palette
* **Nama Produk**: `test-net`
* **Tagline**: *Precise & Real-Time Connection Benchmark*
* **Aksen Warna Fungsional**:
  * **Brand / Ready**: Emerald (`#10b981` / `text-emerald-500`)
  * **Download Stream**: Cyan / Sky (`#06b6d4` / `text-cyan-400`)
  * **Upload Stream**: Violet / Indigo (`#8b5cf6` / `text-violet-400`)
  * **Latency / Ping / Jitter**: Amber / Emerald (`#f59e0b` / `#10b981`)
  * **Error / Alert**: Rose / Red (`#f43f5e` / `text-rose-500`)

---

## 2. Desktop Wireframe

```
+-----------------------------------------------------------------------------------+
|  [⚡ test-net]                   Primary Server (SG) 🟢        [About] [FAQ] [Docs]|
+-----------------------------------------------------------------------------------+
|                                                                                   |
|                                TEST YOUR INTERNET                                 |
|                       Real-time actual network speed benchmark                     |
|                                                                                   |
|                   +---------------------------------------------+                 |
|                   |                                             |                 |
|                   |                     /\                      |                 |
|                   |                  /      \                   |                 |
|                   |                 |   GO   |                  |                 |
|                   |                  \      /                   |                 |
|                   |                     \/                      |                 |
|                   |                                             |                 |
|                   |           [   START SPEED TEST   ]          |                 |
|                   |                                             |                 |
|                   |     Server: Primary Test Server (1 Gbps)    |                 |
|                   |     Estimated data usage: ~20MB - 120MB     |                 |
|                   +---------------------------------------------+                 |
|                                                                                   |
|   +-----------------+ +-----------------+ +---------------+ +---------------+     |
|   |   DOWNLOAD      | |     UPLOAD      | |     PING      | |    JITTER     |     |
|   |   --.-- Mbps    | |   --.-- Mbps    | |     -- ms     | |     -- ms     |     |
|   +-----------------+ +-----------------+ +---------------+ +---------------+     |
|                                                                                   |
|          ℹ️ Measurement powered by dedicated LibreSpeed measurement engine.        |
|                                                                                   |
+-----------------------------------------------------------------------------------+
|  © 2026 test-net · Privacy Policy · Methodology & FAQ · Open Source LGPL-3.0      |
+-----------------------------------------------------------------------------------+
```

---

## 3. Mobile Wireframe (375px - 430px)

```
+---------------------------------------+
| [⚡ test-net]                   [ = ]  |
+---------------------------------------+
|                                       |
|          TEST YOUR INTERNET           |
|        Real-time connection test       |
|                                       |
|       +-----------------------+       |
|       |                       |       |
|       |         ( GO )        |       |
|       |                       |       |
|       |   [ START TEST ]      |       |
|       |                       |       |
|       |  📍 Primary Server    |       |
|       |  Est. Data: ~20-100MB |       |
|       +-----------------------+       |
|                                       |
|       +-----------+ +-----------+     |
|       | DOWNLOAD  | | UPLOAD    |     |
|       | --.--     | | --.--     |     |
|       | Mbps      | | Mbps      |     |
|       +-----------+ +-----------+     |
|       +-----------+ +-----------+     |
|       | PING      | | JITTER    |     |
|       | -- ms     | | -- ms     |     |
|       +-----------+ +-----------+     |
|                                       |
|       ℹ️ Dedicated test engine        |
|                                       |
|  © 2026 test-net · Privacy · FAQ      |
+---------------------------------------+
```

---

## 4. Design System Tokens

### 4.1. Color Palette (Tailwind CSS Mapping)

| Token Semantic | Tailwind Class | Nilai HEX / HSL | Penggunaan |
| :--- | :--- | :--- | :--- |
| `background` | `bg-zinc-950` | `#09090b` | Background utama halaman web |
| `surface` | `bg-zinc-900/80` | `#18181b` (backdrop blur) | Container speed test, dialog, card |
| `surface-subtle` | `bg-zinc-800/40` | `#27272a` | Metric card background saat idle |
| `border` | `border-zinc-800` | `#27272a` | Garis batas card & panel |
| `border-active` | `border-cyan-500/50` | `#06b6d4` | Highlight kartu metrik yang aktif diuji |
| `text-primary` | `text-zinc-50` | `#fafafa` | Judul, angka metrik utama |
| `text-muted` | `text-zinc-400` | `#a1a1aa` | Label, satuan (Mbps, ms), subtext |
| `accent-download` | `text-cyan-400` | `#22d3ee` | Metrik dan grafik Download |
| `accent-upload` | `text-violet-400` | `#a78bfa` | Metrik dan grafik Upload |
| `accent-ping` | `text-emerald-400` | `#34d399` | Metrik Latency Ping & Jitter |
| `accent-danger` | `text-rose-500` | `#f43f5e` | Pesan error koneksi, tombol cancel |
| `accent-warning` | `text-amber-400` | `#fbbf24` | Peringatan kuota data & notice |

### 4.2. Typography

* **Font Utama**: Geist Sans (`font-sans`) untuk teks antarmuka, heading, dan navigasi.
* **Font Numerik**: Geist Mono (`font-mono`) dengan properti `tabular-nums` wajib digunakan pada seluruh angka metrik kecepatan untuk mencegah *layout shift* atau efek teks berguncang saat angka diperbarui dengan cepat (60 fps).

| Skala Tipografi | Ukuran Font | Weight | Leading | Contoh Penggunaan |
| :--- | :--- | :--- | :--- | :--- |
| `display-gauge` | `text-6xl sm:text-7xl` | `font-bold` | `leading-none` | Angka kecepatan pada gauge tengah |
| `heading-1` | `text-3xl sm:text-4xl` | `font-extrabold` | `tracking-tight` | Header utama "Test Your Internet" |
| `heading-card` | `text-2xl sm:text-3xl` | `font-bold` | `leading-tight` | Angka pada MetricCard |
| `body-base` | `text-base` | `font-normal` | `leading-relaxed` | Teks penjelasan, FAQ, About |
| `label-caps` | `text-xs` | `font-semibold` | `tracking-widest uppercase` | Label metrik (DOWNLOAD, UPLOAD, PING) |
| `caption` | `text-xs sm:text-sm` | `font-normal` | `leading-normal` | Server info, footer, disclaimer |

### 4.3. Spacing & Radius
* **Border Radius**:
  * Main Hero / Panel: `rounded-3xl` (`24px`)
  * Metric Cards: `rounded-2xl` (`16px`)
  * Badges & Buttons: `rounded-xl` (`12px`)
  * Circular Trigger / Gauge: `rounded-full`
* **Spacing Scale**:
  * Page Container: `max-w-5xl mx-auto px-4 sm:px-6 lg:px-8`
  * Section Gap: `space-y-8 sm:space-y-12`
  * Card Grid Gap: `gap-4 sm:gap-6`

### 4.4. Shadows & Glowing Effects
* **Button Hover Pulse**: `shadow-lg shadow-emerald-500/20`
* **Download Active Glow**: `shadow-xl shadow-cyan-500/10 border-cyan-500/40`
* **Upload Active Glow**: `shadow-xl shadow-violet-500/10 border-violet-500/40`

---

## 5. Komponen Utama (Structure & Layout)

### 5.1. AppHeader (`components/layout/app-header.tsx`)
* **Tanggung Jawab**: Branding, status konektivitas server, navigasi menu.
* **Elemen**:
  * Logo teks `test-net` dengan ikon sinyal/petir.
  * Server indicator pill (`Primary Test Server · Online 🟢`).
  * Link navigasi desktop: Home (`/`), About (`/about`), FAQ (`/faq`), Privacy (`/privacy`).
  * Mobile sheet / hamburger navigation.

### 5.2. SpeedTestPanel (`components/speed-test/speed-test-panel.tsx`)
* **Tanggung Jawab**: Wadah pusat interaksi pengujian yang membungkus Circular Gauge, tombol start/cancel, progress bar, dan status teks.
* **Elemen**:
  * Visual Circular Gauge (SVG SVG-based smooth circular arc atau radial progress ring).
  * Central Readout: Menampilkan nilai metrik yang sedang aktif (misal `142.8` saat fase download) atau tombol `GO` saat idle.
  * Action Bar: Tombol `START TEST` (Idle), `CANCEL` (Sedang berjalan), atau `TEST AGAIN` (Selesai).

### 5.3. MetricCard (`components/speed-test/metric-card.tsx`)
* **Tanggung Jawab**: Menampilkan 4 metrik hasil pengujian.
* **Props**:
  * `title`: "Download" | "Upload" | "Ping" | "Jitter"
  * `value`: `number | null`
  * `unit`: "Mbps" | "ms"
  * `isActive`: `boolean` (memberikan efek pulse dan border bercahaya ketika fase tersebut sedang aktif)
  * `icon`: Lucide icon (`ArrowDown`, `ArrowUp`, `Activity`, `Wifi`)

### 5.4. TestProgress & TestStatus (`components/speed-test/test-progress.tsx`)
* **Tanggung Jawab**: Menampilkan tahapan step bar:
  `[1. Ping & Jitter] ─── [2. Download] ─── [3. Upload] ─── [4. Complete]`
* Menampilkan estimasi progress bar linier persentase (0% - 100%).

### 5.5. ServerInfo & DataUsageNotice
* **ServerInfo**: Nama server, lokasi, estimasi latency awal.
* **DataUsageNotice**: Peringatan estimasi kuota data: *"Speed test uses real data transfer (~20MB–150MB). Mobile data charges may apply."*

---

## 6. Spesifikasi UI States (State Machine Representation)

| State | Tampilan Central Display | Status Text | Tombol Aksi | Kondisi Metric Cards |
| :--- | :--- | :--- | :--- | :--- |
| **Idle** | Tombol `GO` besar hijau bersinar | *"Ready to benchmark your connection"* | `Start Test` (Emerald) | Semua bernilai `--` |
| **Preparing** | Spinner berputar halus | *"Connecting to test server..."* | `Cancel` (Muted) | Semua bernilai `--` |
| **Ping** | Latency millisecond live | *"Measuring latency & jitter..."* | `Cancel` (Rose Ghost) | Ping & Jitter berkedip aktif |
| **Download** | Angka Mbps download live (Cyan) | *"Testing download throughput..."* | `Cancel` (Rose Ghost) | Ping terisi; Download aktif bersinar |
| **Upload** | Angka Mbps upload live (Violet) | *"Testing upload throughput..."* | `Cancel` (Rose Ghost) | Ping & DL terisi; Upload aktif bersinar |
| **Completed** | Ikon Sukses / Ringkasan Kecepatan | *"Test completed successfully"* | `Test Again` (Primary) | Ke-4 metrik terisi permanen |
| **Cancelling** | Spinner transisi | *"Stopping measurement..."* | Dinonaktifkan (`disabled`) | Menahan nilai terakhir |
| **Cancelled** | Pesan dibatalkan | *"Test was cancelled by user"* | `Restart Test` | Kembali ke nilai idle |
| **Error** | Ikon Warning segitiga merah | *"Connection failed. Please retry."* | `Retry Test` (Destructive) | Indikator border merah pada panel |

---

## 7. Responsive Behavior Specification

### 7.1. Mobile Layout (< 640px)
* Gauge berukuran proporsional: Diameter 240px (mudah disentuh satu tangan).
* Grid 4 Metric Cards disusun menjadi **2 kolom x 2 baris** agar seluruh informasi dapat dilihat tanpa scroll panjang.
* Header menggunakan compact logo dan server status disederhanakan menjadi badge ringkas.

### 7.2. Tablet Layout (640px - 1024px)
* Gauge diameter 280px.
* Metric Cards dalam 4 kolom sejajar dengan ukuran font medium (`text-2xl`).

### 7.3. Desktop Layout (> 1024px)
* Lebar container utama maksimal `max-w-4xl`.
* Gauge diameter 320px dengan typography display ekstra besar (`text-7xl font-mono`).
* Metric Cards memiliki padding luas (`p-6`) dengan visual glow saat hover.

---

## 8. Accessibility & Ergonomics (WCAG 2.1 AA)

1. **Screen Reader Optimization (`aria-live`)**:
   * Elemen metrik dinamis dibungkus dengan `aria-live="polite"` dan `aria-atomic="true"`.
   * Nilai metrik tidak di-announce setiap perubahan angka pecahan mikro agar tidak membanjiri audio screen reader, melainkan di-announce per pergantian fase (misal: *"Download test completed: 85 Megabits per second"*).
2. **Keyboard Navigation**:
   * Seluruh tombol interaktif (`GO`, `Cancel`, `Restart`, menu link) dapat dijangkau menggunakan tombol `Tab`.
   * Outline fokus tampak jelas (`focus-visible:ring-2 focus-visible:ring-emerald-500 focus-visible:ring-offset-2`).
3. **Reduced Motion**:
   * Mendukung query `@media (prefers-reduced-motion: reduce)`.
   * Ketika aktif, animasi putaran gauge dan efek pulsing glow dimatikan secara otomatis menjadi transisi warna statis.
4. **Color Contrast Compliance**:
   * Seluruh teks memiliki rasio kontras minimal 4.5:1 terhadap background `zinc-950` untuk teks biasa, dan minimal 3:1 untuk angka display besar.

---

## 9. Component Inventory & Props Interface

```typescript
// MetricCard Props
export interface MetricCardProps {
  title: string;
  value: number | null;
  unit: string;
  icon: React.ReactNode;
  isActive?: boolean;
  accentColor?: "cyan" | "violet" | "emerald" | "amber";
}

// SpeedTestPanel Props
export interface SpeedTestPanelProps {
  status: SpeedTestPhase;
  metrics: SpeedTestMetrics;
  progressRatio: number;
  serverName: string;
  onStart: () => void;
  onCancel: () => void;
  onRestart: () => void;
}

// CircularGauge Props
export interface CircularGaugeProps {
  value: number | null;
  unit: string;
  status: SpeedTestPhase;
  progressRatio: number;
}
```

---

### Status Desain (Design System Sign-off)
* **Dokumen UI/UX Guide**: Selesai dan siap menjadi acuan implementasi kode pada Phase 4 & Phase 5.
* **Konsistensi**: Sesuai dengan spesifikasi PRD dan arsitektur teknis [docs/TECHNICAL_PLAN.md](file:///d:/10Project/3-Test-net/test-net/docs/TECHNICAL_PLAN.md).
* **Phase 3 Status**: **APPROVED & READY FOR PHASE 4**.
