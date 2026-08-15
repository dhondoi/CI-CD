
---
---
---
---
---
### TRADISIONAL
`Note : Dirancang untuk private repo`
1. Buka Repo > setting > actions > runners
2. Ikuti langkahnya
3. untuk jalankan secara manual gunakan `run.sh`
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
### 
