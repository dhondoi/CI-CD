Mendeploy web statis dari GitHub ke server mandiri seperti Orange Pi adalah proyek yang sangat bagus! Cara paling efektif dan umum untuk melakukan CI/CD dalam skenario ini adalah menggunakan **GitHub Actions** yang akan mengirim file web Anda ke Orange Pi menggunakan protokol SSH (SCP/Rsync) setiap kali ada perubahan kode.

Berikut adalah langkah-langkah lengkapnya:

### Prasyarat Penting

Agar GitHub Actions bisa mengirim file ke Orange Pi Anda, Orange Pi harus terhubung ke internet dan **bisa diakses dari luar**. Anda bisa menggunakan:

* IP Publik Statis + Port Forwarding di router Anda.
* Layanan Tunneling seperti **Cloudflare Tunnel** (sangat disarankan, gratis dan aman) atau Ngrok.

---

### Langkah 1: Persiapan Web Server di Orange Pi

Pertama, kita perlu menyiapkan tempat bernaung untuk web statis Anda di Orange Pi menggunakan web server ringan seperti Nginx.

1. Buka terminal/SSH ke Orange Pi Anda.
2. Update sistem dan install Nginx:
```bash
sudo apt update
sudo apt install nginx -y

```


3. Buat folder untuk menyimpan file web statis Anda (misalnya `mywebsite`):
```bash
sudo mkdir -p /var/www/mywebsite

```


4. Ubah kepemilikan folder agar user Anda bisa menulis file ke dalamnya tanpa akses root:
```bash
sudo chown -R $USER:$USER /var/www/mywebsite

```



---

### Langkah 2: Membuat SSH Key untuk GitHub Actions

GitHub memerlukan kunci akses untuk bisa masuk dan menaruh file di Orange Pi Anda secara otomatis tanpa password.

1. Di terminal Orange Pi, jalankan perintah ini untuk membuat pasangan kunci SSH baru (tekan *Enter* terus sampai selesai, jangan beri password):
```bash
ssh-keygen -t ed25519 -f ~/.ssh/github_actions_key -C "github-actions"

```


2. Masukkan kunci publik (Public Key) ke dalam daftar kunci yang diizinkan di Orange Pi:
```bash
cat ~/.ssh/github_actions_key.pub >> ~/.ssh/authorized_keys

```


3. Tampilkan isi **Private Key**. Anda akan menyalin isi teks ini untuk dimasukkan ke GitHub pada langkah berikutnya:
```bash
cat ~/.ssh/github_actions_key

```


*(Blok dan copy semua teks dari `-----BEGIN OPENSSH PRIVATE KEY-----` sampai `-----END OPENSSH PRIVATE KEY-----`)*

---

### Langkah 3: Menambahkan GitHub Secrets

Sekarang kita beritahu GitHub cara menghubungi Orange Pi Anda.

1. Buka repositori web statis Anda di GitHub.
2. Pergi ke tab **Settings** > **Secrets and variables** > **Actions**.
3. Klik tombol **New repository secret** dan tambahkan 4 secret berikut satu per satu:
* `SERVER_HOST` : Isi dengan IP Publik atau Domain Orange Pi Anda.
* `SERVER_PORT` : Isi dengan port SSH (biasanya `22`, kecuali Anda mengubahnya).
* `SERVER_USERNAME`: Isi dengan username login Orange Pi Anda (misal: `orangepi` atau `ubuntu`).
* `SERVER_SSH_KEY` : Paste seluruh teks Private Key yang Anda copy di Langkah 2.



---

### Langkah 4: Membuat Workflow CI/CD

Langkah terakhir adalah membuat skrip otomatisasi di dalam repositori Anda.

1. Di repositori GitHub Anda, buat file baru dengan path/struktur folder berikut:
`.github/workflows/deploy.yml`
2. Paste kode konfigurasi berikut ke dalam file tersebut:

```yaml
name: Deploy Web Statis ke Orange Pi

on:
  push:
    branches:
      - main  # Akan berjalan setiap kali ada push ke branch main

jobs:
  deploy:
    name: Deploy ke Server
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout Kode
        uses: actions/checkout@v4

      - name: Copy file ke Orange Pi via SSH
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USERNAME }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          port: ${{ secrets.SERVER_PORT }}
          source: "." # Mengcopy semua file di repositori
          target: "/var/www/mywebsite" # Folder tujuan di Orange Pi
          rm: true # (Opsional) Menghapus file lama di target sebelum mengcopy yang baru

```

3. Simpan dan **Commit** file tersebut.

### Selesai! Uji Coba Deployment

Setiap kali Anda mengubah kode HTML/CSS di komputer Anda dan melakukan `git push` ke branch `main`, GitHub Actions akan otomatis berjalan (bisa dicek di tab **Actions** di GitHub). File terbaru akan langsung dikirim dan diupdate di dalam folder `/var/www/mywebsite` di Orange Pi Anda.

