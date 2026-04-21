# Monitoring Sistem K24Klik Order API

WIP: Desc

## Daftar Isi

- [Fitur](#-fitur)
- [Struktur Project](#-struktur-project)
- [Persyaratan](#-persyaratan)
- [Konfigurasi](#-konfigurasi)
- [Menjalankan Aplikasi](#-menjalankan-aplikasi)

## Fitur

- **Autentikasi & Otorisasi**
  - Login dengan JWT token
  - Refresh token
  - Middleware untuk protected routes
  
- **Manajemen User**
  - CRUD operations untuk user
  - Role-based access control

## Struktur Project

```
monitoring-stack/
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml
│   └── rules.yml
├── grafana/
│   └── provisioning/
│       └── datasources/
│           └── prometheus.yml      
│       └── dashboards/
│           ├── dashboard.yml
│           └── k24klik-overview.json       
├── alertmanager/
│   └── alertmanager.yml
├── .env.example
└── README.md
```

## 📦 Persyaratan
- Docker Engine v20.10+
- Docker Compose v2.0+
- OS: Linux / macOS / Windows (Docker Desktop)

## 🛠️ Instalasi dan Menjalankan

1. **Clone repository**

```bash
git clone https://github.com/alfianyulianto/golang-api-boilerplate.git
cd golang-api-boilerplate
```
2. Buat file `.env` dan isi `GF_SECURITY_ADMIN_PASSWORD=k24monitoring2026`.
3. Jalankan perintah: `docker compose up -d`.
4. Akses Grafana di `http://localhost:3000`.

## Kredensial Default
- **Grafana**: admin / k24monitoring2026
- **Prometheus**: http://localhost:9090

## Keputusan Teknis
### Docker Compose
- Port Mapping cAdvisor: Di dokumen, cadvisor menggunakan port 8080. Namun, API Gateway K24Klik juga berjalan di port 8080. Maka, saya arahkan port host-nya ke 8081 untuk menghindari bentrokan (conflict).
- Healthcheck: Saya menambahkan wget sebagai metode check karena image Prometheus dan Grafana versi ini sangat minimalis.
- Persistence: Menggunakan named volumes (prometheus_data, grafana_data) memastikan data monitoring tidak hilang saat container dihapus atau di-update
### Prometheus
- Scrape Interval (15s): Dipilih agar data cukup real-time tanpa membebani resource server secara berlebihan.
- Absent Query: Gunakan absent() untuk mendeteksi kontainer yang hilang secara total dari metrik.
- Node Exporter: Metrik node_cpu_seconds_total digunakan untuk menghitung penggunaan CPU rata-rata dengan mengurangi waktu idle dari total kapasitas.
### Alert Manager
- Routing Logic: Kita membagi jalur pengiriman berdasarkan tingkat keparahan (severity). Alert Critical dikirim seketika (group_wait: 0s) karena butuh penanganan cepat, sementara Warning ditahan dulu selama 5 menit agar tidak berisik jika banyak masalah kecil muncul bersamaan.
- Inhibition Rules: Ini adalah bagian paling cerdas dari DevOps. Jika sebuah server sudah dalam kondisi Critical (misal CPU 95%), maka notifikasi Warning (CPU 80%) di server yang sama tidak perlu dikirim lagi karena kita sudah tahu ada masalah besar di sana.
- Slack Integration: Sesuai instruksi, kita menggunakan webhook receiver ke URL dummy http://localhost:5001 sebagai representasi integrasi Slack.

## Important Checklist
- [x] docker compose up -d berjalan tanpa error
- [x] docker compose ps — semua service healthy
- [x] Prometheus (localhost:9090/targets) — semua target UP
- [x] Grafana (localhost:3000) — dashboard K24Klik muncul otomatis (admin/k24monitoring2026)
- [x] Alertmanager (localhost:9093) — dapat diakses
- [x] Alert rules terdefinisi di Prometheus (localhost:9090/rules)
- [x] Semua 8 alert ada di rules.yml dengan label dan annotation lengkap
- [x] README.md lengkap: setup guide + keputusan teknis
- [x] Tidak ada file sensitif (password, .env) yang ter-commit
- [x] Repository bersih: .gitignore sudah dikonfigurasi

## Reference
- Prometheus Query Language (PromQL) check [reference](https://prometheus.io/docs/prometheus/latest/querying/functions)