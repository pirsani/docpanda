
# **3. Panduan Pengembang**

## **3.1 Ringkasan**

- **Sistem Arsitektur**

  ![sistem-arsitektur](images/3/sistem-arsitektur.png)
  gambar 3.1 Sistem arsitektur

## **3.2 Instalasi Prasyarat**

- Node.js, PNPM, PostgreSQL, Redis, git.

> **(panduan berikut merupakan panduan untuk ubuntu)**

- Perbarui paket-paket yang ada di server:

```sh
sudo apt update -y
```

### **3.2.1 Instalasi Node.js**

referensi : <https://nodejs.org/en/learn/getting-started/how-to-install-nodejs>

- Pastikan Node.js telah terinstal. Jika belum, instal Node.js:

```sh
sudo apt install nodejs
node -v
```

### **3.2.2 Instalasi pnpm**

referensi : <https://pnpm.io/installation>

```sh
corepack enable
corepack prepare pnpm@latest --activate
```

### **3.2.3 Instalasi PostgreSQL**

referensi : <https://www.postgresql.org/download/linux/ubuntu/>

```sh
sudo apt install -y postgresql postgresql-contrib
```

- Buat Database dan pengguna untuk proyek:

  lakukan pengaturan pengguna dengan memperhatikan praktek keamanan yang baik

```sh
$ sudo -i -u postgres
$ psql
postgres=# CREATE DATABASE nama_database;
postgres=# CREATE USER nama_pengguna WITH ENCRYPTED PASSWORD 'kata_sandi';
postgres=# GRANT ALL PRIVILEGES ON DATABASE nama_database TO nama_pengguna;
postgres=# \q
$ exit
```

### **3.2.4 Instalasi Redis**

```sh
sudo apt install redis-server
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

check apakah Redis berjalan

```sh
sudo systemctl status redis-server
```

Tes Redis

```sh
redis-cli ping
```

response `PONG`

(**OPSIONAL**) jika di windows disarankan menggunakan docker untuk instalasi Redis

berikut contoh `docker-compose.yml`

```yml
services:
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - ./data:/data
```

## **3.3 Instalasi STRASA**

### **3.3.1 Instalasi**

- Clone repository

```sh
git clone  git@github.com:pirsani/panda.git
cd panda
pnpm install
```

- Pengaturan `environment variables` di file .env.

```sh
copy .env.dist .env
```

ubah isi `.env` sesuai konfigurasi anda

```plaintext
# .env
DATABASE_URL_ADMIN="postgresql://postgres:postgres@localhost:5432/panda?schema=public"
DATABASE_URL_HONORARIUM="postgresql://postgres:postgres@localhost:5432/panda?schema=public"

INIT_ADMIN_PASSWORD="Passadmin#123"

BASE_PATH_UPLOAD="/path/to/your/folder/BASE_PATH_UPLOAD"
BASE_PATH_UPLOAD_CHUNK="/path/to/your/folder/BASE_PATH_UPLOAD_CHUNK"
```

### **3.3.2 Struktur Folder**

  Struktur folder secara umum mengikuti [struktur folder Nextjs 14.x](https://nextjs.org/docs/app/getting-started/project-structure).

  Keterangan lebih lanjut dapat melihat dokumentasi [Project Structure and Organization Next.js 14.x](https://nextjs.org/docs/app/getting-started/project-structure).

```tree
> tree -a -F -L 1
./
├── .env*
├── .env.dist
├── .env.local*
├── .eslintrc.json
├── .git/
├── .github/
├── .gitignore
├── .next/
├── .vscode/
├── BASE_PATH_UPLOAD/
├── README.md
├── components.json
├── docs/
├── fonts/
├── helper/
├── next-env.d.ts
├── next.config.mjs
├── node_modules/
├── package.json
├── pnpm-lock.yaml
├── postcss.config.mjs
├── prisma/
├── public/
├── src/
├── tailwind.config.ts
└── tsconfig.json

13 directories, 15 files
```

`BASE_PATH_UPLOAD` upload disini menyesuaikan dengan dengan konfigurasi yang ada pada `.env`.
**sangat disarankan** untuk pengaturan path ini di luar root project

### **3.3.3 Migrasi Database**

> ⚠️ **PERINGATAN:**
>
> Periksa konfigurasi sebelum melanjutkan.
>
> push hanya dilakukan di environment development, untuk environment production gunakan `deploy`
>
> `seed` hanya dilakukan sekali di awal, seed akan mereset data
  
- Jalankan migrasi Prisma untuk pertama kali.

```sh
pnpm run prisma:db-push
```
  
- Initial database.

    Proses ini akan menginisiasi data awal dengan data user superadmin, negara, provinsi dan kota, dan satker

```sh
pnpm prism db seed
```

## **3.3 Menjalakan development server**

untuk menjalankan aplikasi

`pnpm dev`

jika semua benar aplikasi dapat mengakses aplikasi di <http://localhost:3000>

## **3.4 Deployment ke server production**

Untuk deploy ke production kita dapat menggunakan `nginx` sebagai proxy server dan `pm2` untuk mengelola proses.

Pastikan langkah seperti yang dijelaskan di langkah [Instalasi Prasyarat](#32-instalasi-prasyarat) telah dilakukan

### **3.4.1 Instalasi PM2** secara global
  
```sh
pnpm add -g pm2
```

### **3.4.2 Instalasi NGINX**

```sh
sudo apt install -y nginx
```

  Konfigurasi NGINX untuk meneruskan permintaan ke Next.js. Buat file konfigurasi baru:

```sh
sudo nano /etc/nginx/sites-available/panda-app
```

  Tambahkan konfigurasi berikut ke file tersebut:

```conf
# /etc/nginx/sites-available/panda-app
proxy_cache_path /var/cache/nginx/d01_pirsani levels=1:2 keys_zone=STATIC_D01_PIRSANI:10m inactive=7d use_temp_path=off;

upstream db01.pirsani_upstream {
  server 127.0.0.1:3030;
}

server {
      server_name d01.pirsani.id; # !!! - change to your domain name
      gzip on;
      gzip_proxied any;
      gzip_types application/javascript application/x-javascript text/css text/javascript;
      gzip_comp_level 5;
      gzip_buffers 16 8k;
      gzip_min_length 256;

  location /_next/static/ {
              proxy_cache STATIC_D01_PIRSANI;
              proxy_pass http://db01.pirsani_upstream;
              expires 60m;
              access_log off;
      }

  location / {
              proxy_pass http://db01.pirsani_upstream; # !!! - change to your app port
              proxy_http_version 1.1;
              proxy_set_header Upgrade $http_upgrade;
              proxy_set_header Connection 'upgrade';
              proxy_set_header Host $host;
              proxy_cache_bypass $http_upgrade;
      }

  listen 443 ssl; # managed by Certbot
  ssl_certificate /etc/letsencrypt/live/d01.pirsani.id/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/d01.pirsani.id/privkey.pem; # managed by Certbot
  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
  if ($host = d01.pirsani.id) {
      return 301 https://$host$request_uri;
  } # managed by Certbot


  if ($host = d01.pirsani.id) {
      return 301 https://$host$request_uri;
  } # managed by Certbot


  listen 80;
  server_name d01.pirsani.id d01.pirsani.id;
  return 404; # managed by Certbot

}

```

(**Opsional**) Pastikan untuk memperbarui konfigurasi DNS dan mengarahkan domain ke IP server. Untuk mengamankan koneksi, dapat menggunakan **Certbot** untuk SSL:

```sh
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com
```

jika sudah mempunyai ssl certificate sendiri, silakan sesuaikan dengan yang dimiliki.

Aktifkan konfigurasi NGINX ini:

```bash
sudo ln -s /etc/nginx/sites-available/panda-app /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

Selamat! aplikasi berhasil dipasang

Baca [Dokumentasi Teknis Database](database.md) untuk mempelajari lebih dalam tentang database

---
