# Cara Install Genieacs di Ubuntu 24.04
Untuk menginstal GenieACS di Ubuntu 24.04, berikut adalah langkah-langkahnya:

### Prasyarat
1. **Ubuntu 24.04 LTS** (pastikan sistem ter-update).
2. **Node.js** (minimal versi 14.x).
3. **Redis**.
4. **MongoDB** (minimal versi 4.4).
5. **Nginx** (opsional, untuk reverse proxy).

### Langkah 1: Update dan Install Dependencies
Perbarui paket sistem Anda dan install dependensi yang diperlukan:
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install curl git build-essential redis-server -y
```

### Langkah 2: Install MongoDB
Install MongoDB sesuai dokumentasi resmi:
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
```

### Langkah 3: Install Node.js
GenieACS memerlukan Node.js versi 14.x atau lebih tinggi. Install Node.js dengan perintah berikut:
```bash
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt install -y nodejs
```

### Langkah 4: Clone GenieACS Repository
Clone repository GenieACS dari GitHub:
```bash
git clone https://github.com/genieacs/genieacs.git
cd genieacs
```

### Langkah 5: Install Dependencies
Setelah meng-clone GenieACS, install dependensi yang dibutuhkan:
```bash
npm install
```

### Langkah 6: Konfigurasi GenieACS
Buat direktori konfigurasi untuk GenieACS:
```bash
sudo mkdir -p /opt/genieacs/config
sudo cp config/*.example.yml /opt/genieacs/config/
sudo nano /opt/genieacs/config/genieacs.env
```

Sesuaikan file konfigurasi seperti yang dibutuhkan, terutama jika Anda menggunakan MongoDB, Redis, atau setting lain yang custom.

### Langkah 7: Buat Systemd Service untuk GenieACS
Buat service file untuk memudahkan mengelola GenieACS:
```bash
sudo nano /etc/systemd/system/genieacs-cwmp.service
```

Isi dengan konfigurasi berikut:
```ini
[Unit]
Description=GenieACS CWMP
After=network.target

[Service]
ExecStart=/usr/bin/genieacs-cwmp
Restart=always
User=nobody
Group=nogroup
EnvironmentFile=/opt/genieacs/config/genieacs.env

[Install]
WantedBy=multi-user.target
```

Ulangi langkah di atas untuk `genieacs-nbi` dan `genieacs-fs` dengan mengganti `ExecStart` menjadi `genieacs-nbi` dan `genieacs-fs` sesuai layanannya.

Setelah itu, aktifkan dan jalankan service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable genieacs-cwmp genieacs-nbi genieacs-fs
sudo systemctl start genieacs-cwmp genieacs-nbi genieacs-fs
```

### Langkah 8: Konfigurasi Nginx (Opsional)
Jika Anda ingin mengatur reverse proxy dengan Nginx, buat konfigurasi Nginx untuk GenieACS di `/etc/nginx/sites-available/default`:
```bash
server {
    listen 80;
    server_name your_domain.com;

    location / {
        proxy_pass http://localhost:7557;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Jangan lupa untuk restart Nginx setelah konfigurasi:
```bash
sudo systemctl restart nginx
```

### Langkah 9: Akses GenieACS
Anda bisa mengakses GenieACS melalui browser menggunakan alamat:
```
http://your_domain.com
```

Dengan ini, GenieACS seharusnya sudah siap digunakan di Ubuntu 24.04.
