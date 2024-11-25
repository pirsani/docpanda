
# **3. Panduan Pengembang**

## **3.1 Ringkasan**

- **Sistem Arsitektur**

  ![sistem-arsitektur](images/3/sistem-arsitektur.png)
  gambar 3.1 Sistem arsitektur


## **3.2 Instalasi Prasyarat**


- Node.js, PNPM, PostgreSQL, Redis, PM2, git.

**(panduan berikut merupakan panduan untuk ubuntu)**

- Perbarui paket-paket yang ada di server:

```sh
sudo apt update && sudo apt upgrade -y
```

### **3.2.1 Instalasi Node.js**
- Pastikan Node.js telah terinstal. Jika belum, instal Node.js:

```sh
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

### **3.2.2 Instalasi pnpm**:

```sh
corepack enable
corepack prepare pnpm@latest --activate
```

### **3.2.3 Instalasi PM2** secara global:
  
```sh
pnpm add -g pm2
```

### **3.2.4 Instalasi PostgreSQL**:

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

### **3.2.5 Instalasi NGINX**:

   ```sh
   sudo apt install -y nginx
   ```

  Konfigurasi NGINX untuk meneruskan permintaan ke Next.js Anda. Buat file konfigurasi baru:

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

(Opsional) Jika Anda menggunakan domain, pastikan untuk memperbarui konfigurasi DNS Anda dan mengarahkan domain ke IP server Anda. Untuk mengamankan koneksi, Anda dapat menggunakan **Certbot** untuk SSL:

```sh
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com
```

jika anda mempunyai ssl certificate sendiri, silakan sesuaikan dengan yang anda miliki.


Aktifkan konfigurasi NGINX ini:

```bash
sudo ln -s /etc/nginx/sites-available/panda-app /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

## **3.3 Instalasi Panda**

### **3.3.1 Instalasi**:

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


### **3.3.3 Migrasi Database**:

> ⚠️ **PERINGATAN:** 

> Pastikan Anda memeriksa konfigurasi sebelum melanjutkan.
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

## **3.3 Development Workflow**
- How to run the development server (`next dev`).
- Debugging tips for key technologies (e.g., Prisma, Zustand).
- Guidelines for adding new features or fixing bugs.

## **3.4 Testing**
- Tools and strategies used for testing:
  - Unit tests for components (React Testing Library).
  - Integration testing for API routes.
- Instructions for running tests.

## **3.5 Deployment**
- Deployment process for staging and production environments.
- Managing environment variables securely in production.
- Monitoring and troubleshooting deployments.

---