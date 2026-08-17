---
---
---
---
---
# TRADISIONAL
1. **Buat berkas skrip Bash di VPS (misalnya /opt/scripts/deploy.sh) yang berisi perintah pull dan build aplikasi Anda:**
    - buat direktori
      ```bash
      mkdir /opt/scripts
      ```
    - nano
      ```bash
      nano /opt/scripts/deploy.sh
      ```
    - isi file `deploy.sh`
      ```sh
      #!/bin/bash
      set -e
      
      cd /var/www/my-app
      ```
2. **Beri izin eksekusi pada berkas tersebut:**
   ```Bash
   chmod +x /opt/scripts/deploy.sh
   ```
3. **Install dan Konfigurasi Webhook Engine di VPS:**
    - Menggunakan tool open-source lightweight 'webhook'.
    - Install utilitas webhook di VPS Ubuntu/Debian:
      ```Bash
      sudo apt update && sudo apt install webhook -y
      ```
4. **Buat berkas konfigurasi /opt/scripts/hooks.json:**
   - buat token
     ```bash
     openssl rand -hex 32
     ```
     simpan token tersebut
   - nano
      ```bash
      nano /opt/scripts/hooks.json
      ```
    - isi dan ubah `KUNCI_TOKEN_RAHASIA_ANDA` dari token yang dbuat
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
   - bejalan di background
     ```Bash
     webhook -hooks /opt/scripts/hooks.json -verbose -logfile /var/log/webhook.log &
     ```
   - berjalan tanpa background
     ```bash
     webhook -hooks /opt/scripts/hooks.json -verbose -logfile /var/log/webhook.log
     ```
   - untuk mematikan
     ```bash
     sudo pkill -f webhook
     ```
   - cek port yang berjalan
     ```Bash
     ss -tunlp
     ```
6. Hubungkan ke Cloudflare Tunnel:
   - Buka dashboard Cloudflare Zero Trust > Networks > Tunnels > Edit Tunnel Anda.
   - Tambahkan Public Hostname baru:
     - Subdomain: deploy (misal: deploy.domainanda.com)
     - Type: HTTP
     - URL: localhost:9000
   - Sekarang, endpoint webhook Anda aman diakses via HTTPS di URL: [https://deploy.domainanda.com/hooks/deploy-app](https://deploy.domainanda.com/hooks/deploy-app).
7. Konfigurasi Workflow di GitHub Actions:
   - Simpan token rahasia di GitHub Secrets repositori dengan nama DEPLOY_TOKEN ( nilainya harus sama dengan KUNCI_TOKEN_RAHASIA_ANDA).
   - Buat file .github/workflows/deploy.yml:
     - YAMLname: Trigger Webhook Deploy
        ```yml
        name: Trigger Webhook Deploy
        
        on:
          push:
            branches: [ main ]
        
        jobs:
          deploy:
            runs-on: ubuntu-latest
            steps:
              - name: Call VPS Webhook
                run: |
                  response=$(curl -s -w "\nHTTP_STATUS:%{http_code}" -X POST https://deploy.domainanda.com/hooks/deploy-app \
                       -H "X-Deploy-Token: ${{ secrets.DEPLOY_TOKEN }}")
              
                  body=$(echo "$response" | sed -e 's/HTTP_STATUS:.*//g')
                  status=$(echo "$response" | tr -d '\n' | sed -e 's/.*HTTP_STATUS://')
              
                  echo "=== LOG EKSEKUSI VPS ==="
                  echo "$body"
              
                  if [ "$status" -ne 200 ]; then
                    echo "Deployment di VPS Gagal!"
                    exit 1
                  fi
        ```
Kelebihan & Kelemahan Metode Webhook
- Kelebihan:Sangat ringan dan tidak membutuhkan koneksi SSH.
- Server tidak perlu membuka port ke internet (tetap terlindungi di balik Cloudflare Tunnel).
- Setup simpel dan pemicuan berlangsung instan.
Kelemahan:
- Log hasil eksekusi deploy berada di VPS, bukan di halaman log GitHub Actions (GitHub hanya mengetahui apakah panggilan HTTP berhasil atau gagal).
---
