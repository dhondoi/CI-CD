
---
---
---
---
---
### TRADISIONAL
`Note : Dirancang untuk private repo`
1. Buka Repo > setting > actions > runners.
2. Ikuti langkahnya (jika list tahap `install runner` sudah dilakukan, skip ke tahap list `config dst`) yang diperlukan pada server.
3. untuk jalankan secara manual gunakan `run.sh` <br/>
*Status runner di menu **Settings > Actions > Runners** di GitHub sekarang akan berubah menjadi **Idle** (Hijau).*
4. untuk automatis (instal sebagai service)
```bash
# untuk install, jalankan dan cek status
sudo ./svc.sh install && sudo ./svc.sh start && sudo ./svc.sh status
# untuk uninstall, stop dan cek status
sudo ./svc.sh stop && sudo ./svc.sh uninstall && sudo ./svc.sh status
```
5. buat folder pada direktory default nginx.
```bash
# buat direktori
mkdir /var/www/testdir
# ubah kepemilkan beserta isnya
sudo chown -R ${USER}:${USER} /var/www/<NAMA_PROYEK>
# ubah permission
sudo chmod -R 755 /var/www/<NAMA_PROYEK>
```
6. simpan buat konfigurasi `.github/worflows/deploy.yml`
```yml
name: Deploy Web Statis via Self-Hosted Runner

on:
  push:
    branches:
      - main

jobs:
  deploy:
    # Memberitahu GitHub untuk menjalankan pekerjaan ini di Orange Pi Anda
    runs-on: self-hosted
    
    steps:
      - name: Checkout Kode
        uses: actions/checkout@v4

      - name: Salin file ke direktori Nginx
        run: |
          rsync -av --delete --exclude='.git*' ./ /var/www/html/<NAMA_PROYEK>
```
7. Coba commit repo, jika berhasil maka ada proyek repo di server. untuk runner **`organisasi`** / **`enterprise`** mirip, kita hanya lanjut dari config karena yang membedakan hanyalah di mana Anda mendaftarkannya (token registrasi yang Anda salin saat menjalankan perintah `./config.sh`
---
---
---
---
---
### DOCKER
1. mari coba buat repo baru
2. ikuti langkah 1-3 [TRADISONAL](#TRADISIONAL)
3. buat file `index.html`
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Web Statis Saya</title>
</head>
<body>
    <h1>Selamat Datang di Website Saya!</h1>
    <p>Halaman ini diperbarui secara otomatis menggunakan CI/CD Docker.</p>
</body>
</html>
```
3. buat `.Dockerfile`
```Dockerfile
# Gunakan Nginx versi ringan
FROM nginx:alpine

# Salin berkas index.html ke folder default Nginx
COPY index.html /usr/share/nginx/html/index.html

# Buka port 80
EXPOSE 80
```
4. buat `.github/workflows/deploy.yml` (konfigurasi CI/CD)
```yml
name: Deploy Web Statis

on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    runs-on: self-hosted

    steps:
      - name: Ambil Kode Terbaru
        uses: actions/checkout@v4

      - name: Build Image Web Statis
        run: docker build -t web-statis:v1 .

      - name: Jalankan Kontainer Nginx Baru
        run: |
          docker stop web-statis-container || true
          docker rm web-statis-container || true
          docker run -d \
            --name web-statis-container \
            -p 80:80 \
            --restart always \
            web-statis:v1
```
