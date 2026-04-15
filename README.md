# Prometheus & Grafana Setup

Konfigurasi Docker untuk monitoring backend dengan Prometheus dan Grafana.

## Struktur File

```
sre/
├── docker-compose.yml          # Docker compose untuk Prometheus dan Grafana
├── prometheus.yml              # Konfigurasi Prometheus (scrape config)
├── grafana/
│   └── provisioning/
│       ├── datasources/
│       │   └── datasources.yml # Konfigurasi datasource (Prometheus)
│       └── dashboards/
│           ├── dashboards.yml  # Konfigurasi provisioning dashboard
│           └── backend-dashboard.json  # Dashboard monitoring backend
└── README.md
```

## Services

### Prometheus (port 9090)
- Mengumpulkan metrics dari backend service eksternal
- Menyimpan data timeseries
- Scrape interval: 10 detik

### Grafana (port 3000)
- Default credentials: admin / admin
- Sudah terkonfigurasi Prometheus sebagai datasource
- Dashboard "Backend Monitoring" otomatis tersedia

### Backend (Eksternal)
- Backend Anda berjalan di service lain
- Harus mengexpose metrics di `/metrics`
- Update `prometheus.yml` dengan host:port backend Anda

## Cara Menjalankan

```bash
# Start semua services
docker-compose up -d

# Lihat logs
docker-compose logs -f

# Stop services
docker-compose down
```

## Akses

- **Prometheus**: http://localhost:9090
- **Grafana**: http://localhost:3000

## Metrics dari Backend

Backend harus mengexpose metrics dalam format Prometheus di endpoint `/metrics`.

Contoh format:
```
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{handler="/api/users",method="GET"} 1234

# HELP http_request_duration_seconds HTTP request latency
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{handler="/api/users",le="0.1"} 100
```

## Konfigurasi Backend Eksternal

Edit `prometheus.yml` di bagian scrape config untuk `backend`:

```yaml
  - job_name: 'backend'
    scrape_interval: 10s
    static_configs:
      - targets: ['localhost:8080']  # Ganti dengan host:port backend Anda
    metrics_path: '/metrics'
```

Kemungkinan konfigurasi:
- **Backend di localhost**: `localhost:8080`
- **Backend di mesin lain**: `192.168.1.100:8080`
- **Backend di docker network**: `backend-service:8080` (jika ada container terpisah)

## Notes

- Prometheus dan Grafana terhubung via docker network `monitoring`
- Prometheus menyimpan data di volume `prometheus_data`
- Grafana menyimpan data di volume `grafana_data`
- Dashboard otomatis di-load dari `grafana/provisioning/dashboards/`
- Backend berjalan eksternal, pastikan accessible dari host docker
