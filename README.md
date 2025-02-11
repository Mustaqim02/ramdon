# Instalasi Marzban dengan HTTPS

Dokumen ini menjelaskan langkah-langkah lengkap untuk menginstal Marzban dan mengonfigurasi HTTPS menggunakan sertifikat SSL dari Let's Encrypt.

---

## Persyaratan
- VPS dengan Debian 11.
- Akses root atau pengguna dengan hak akses `sudo`.
- Domain yang sudah mengarah ke IP VPS (misalnya, `anu.mustaqimraya.my.id`).

---

## Langkah-langkah Instalasi

### 1. Update dan Upgrade Sistem
Perbarui sistem Anda:
```bash
apt update && apt upgrade -y

2. Instal Dependensi yang Diperlukan

Instal dependensi yang diperlukan untuk Marzban:
bash
Copy

apt install -y curl git python3 python3-pip python3-venv

3. Instal Docker dan Docker Compose

Instal Docker dan Docker Compose:
bash
Copy

apt install -y docker.io docker-compose
systemctl enable docker
systemctl start docker

4. Instal Marzban Menggunakan Script Resmi

Gunakan script instalasi resmi dari Gozargah untuk menginstal Marzban:
bash
Copy

bash -c "$(curl -sL https://github.com/Gozargah/Marzban-scripts/raw/master/marzban.sh)" @ install

5. Verifikasi Instalasi Marzban

Setelah instalasi selesai, verifikasi bahwa Marzban berjalan:
bash
Copy

docker-compose -f /opt/marzban/docker-compose.yml ps

6. Akses Marzban

    Buka browser dan akses http://<IP_VPS>:8000.

    Login dengan kredensial default:

        Username: admin

        Password: admin

Konfigurasi HTTPS
1. Instal acme.sh

Instal acme.sh untuk mendapatkan sertifikat SSL:
bash
Copy

curl https://get.acme.sh | sh -s email=your_email@example.com

2. Buat Direktori untuk Sertifikat

Buat direktori untuk menyimpan sertifikat:
bash
Copy

mkdir -p /var/lib/marzban/certs/

3. Dapatkan Sertifikat SSL

Dapatkan sertifikat SSL dari Let's Encrypt:
bash
Copy

~/.acme.sh/acme.sh --set-default-ca --server letsencrypt --issue --standalone -d anu.mustaqimraya.my.id \
  --key-file /var/lib/marzban/certs/key.pem \
  --fullchain-file /var/lib/marzban/certs/fullchain.pem

4. Konfigurasi Marzban untuk Menggunakan SSL

Edit file konfigurasi Marzban (.env):
bash
Copy

nano /opt/marzban/.env

Tambahkan atau ubah baris berikut:
bash
Copy

UVICORN_SSL_CERTFILE=/var/lib/marzban/certs/fullchain.pem
UVICORN_SSL_KEYFILE=/var/lib/marzban/certs/key.pem

5. Restart Marzban

Restart container Marzban:
bash
Copy

docker-compose -f /opt/marzban/docker-compose.yml restart

6. Verifikasi HTTPS

    Buka browser dan akses https://anu.mustaqimraya.my.id:8000.

    Pastikan koneksi aman (HTTPS) dan sertifikat SSL valid.

Opsional: Gunakan Reverse Proxy (Nginx)
1. Instal Nginx

Instal Nginx:
bash
Copy

apt install nginx -y

2. Konfigurasi Nginx

Buat file konfigurasi baru untuk Marzban:
bash
Copy

nano /etc/nginx/sites-available/marzban

Tambahkan konfigurasi berikut:
nginx
Copy

server {
    listen 80;
    server_name anu.mustaqimraya.my.id;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

Simpan file dan aktifkan konfigurasi:
bash
Copy

ln -s /etc/nginx/sites-available/marzban /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx

3. Dapatkan Sertifikat SSL dengan Certbot

Instal Certbot untuk Nginx:
bash
Copy

apt install certbot python3-certbot-nginx -y

Dapatkan sertifikat SSL:
bash
Copy

certbot --nginx -d anu.mustaqimraya.my.id

4. Restart Nginx

Restart Nginx untuk menerapkan perubahan:
bash
Copy

systemctl restart nginx

Troubleshooting
1. Port 8000 Tidak Terbuka

Pastikan port 8000 terbuka di firewall:
bash
Copy

ufw allow 8000/tcp
ufw reload

2. Sertifikat SSL Tidak Valid

Pastikan sertifikat SSL valid dan dikenali oleh browser. Anda dapat menggunakan alat seperti SSL Labs untuk memverifikasi sertifikat Anda.
3. Log Marzban

Cek log container Marzban untuk melihat pesan kesalahan:
bash
Copy

docker logs marzban-marzban-1
