---
---
---
---
---
# TRADISIONAL
1. Buat berkas skrip Bash di VPS (misalnya /opt/scripts/deploy.sh) yang berisi perintah pull dan build aplikasi Anda: 
```Bash
#!/bin/bash
cd /var/www/my-app
```
2. Beri izin eksekusi pada berkas tersebut:
```Bash
chmod +x /opt/scripts/deploy.sh
```
3. Install dan Konfigurasi Webhook Engine di VPS:

Menggunakan tool open-source lightweight 'webhook'.

Install utilitas webhook di VPS Ubuntu/Debian:
```Bash
sudo apt update && sudo apt install webhook -y
```
4. Buat berkas konfigurasi /opt/scripts/hooks.json:
```JSON
[
  {
    "id": "deploy-app",
    "execute-command": "/opt/scripts/deploy.sh",
    "command-working-directory": "/opt/scripts",
    "trigger-rule": {
      "match": {
        "type": "value",
        "value": "KUNCI_TOKEN_RAHASIA_ANDA",
        "parameter": {
          "source": "header",
          "name": "X-Deploy-Token"
        }
      }
    }
  }
]
```
Pengaturan trigger-rule memastikan hanya permintaan yang menyertakan token rahasia yang benar yang akan dieksekusi.

5. Jalankan service webhook di latar belakang (secara default berjalan di port 9000):
```Bash
webhook -hooks /opt/scripts/hooks.json -verbose &
```
```Bash
# cek port yang berjalan
ss -tunlp
```
6. Hubungkan ke Cloudflare Tunnel:

Buka dashboard Cloudflare Zero Trust > Networks > Tunnels > Edit Tunnel Anda.

Tambahkan Public Hostname baru:
- Subdomain: deploy (misal: deploy.domainanda.com)
- Type: HTTP
- URL: localhost:9000
Sekarang, endpoint webhook Anda aman diakses via HTTPS di URL: [https://deploy.domainanda.com/hooks/deploy-app](https://deploy.domainanda.com/hooks/deploy-app).
4. Konfigurasi Workflow di GitHub Actions:

  Simpan token rahasia di GitHub Secrets repositori dengan nama DEPLOY_TOKEN ( nilainya harus sama dengan KUNCI_TOKEN_RAHASIA_ANDA).

  Buat file .github/workflows/deploy.yml:YAMLname: Trigger Webhook Deploy
```yml
on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Call VPS Webhook
        run: |
          curl -X POST https://deploy.domainanda.com/hooks/deploy-app \
               -H "X-Deploy-Token: ${{ secrets.DEPLOY_TOKEN }}" \
               --fail
```
Kelebihan & Kelemahan Metode Webhook
- Kelebihan:Sangat ringan dan tidak membutuhkan koneksi SSH.
- Server tidak perlu membuka port ke internet (tetap terlindungi di balik Cloudflare Tunnel).
- Setup simpel dan pemicuan berlangsung instan.
Kelemahan:
- Log hasil eksekusi deploy berada di VPS, bukan di halaman log GitHub Actions (GitHub hanya mengetahui apakah panggilan HTTP berhasil atau gagal).
---
