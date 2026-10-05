# LKPD 4 - Publikasi Docker Image ke Docker Hub

## Petunjuk Awal
Instance/mesin yang digunakan adalah Ubuntu/Debian.

## Langkah Kerja

### A. Persiapkan Lingkungan Kerja Container Docker

#### 1. Sign In / Sign Up di Docker Hub
Buka browser dan akses: https://hub.docker.com

Buat akun jika belum memiliki, atau login jika sudah memiliki akun.

#### 2. Login ke akun Docker Anda dari host
```bash
docker login
```

Atau bisa langsung login dari terminal saja, misal pakai username `paknux`:
```bash
docker login -u paknux
```

Masukkan password Docker Hub Anda saat diminta.

#### 3. Cek image yang akan di-push ke Docker Hub
```bash
docker images
```

Contoh dalam hal ini kita akan push image `ubuntu-ws:v1` yang sudah dibuat di LKPD 3.

#### 4. Beri "tag" ulang image sesuai format Docker Hub
Docker Hub butuh format nama: `username/nama-image:tag`

```bash
docker tag ubuntu-ws:v1 <username-dockerhub>/ubuntu-ws:v1
```

Contoh jika username Docker Hub Anda adalah `paknux`:
```bash
docker tag ubuntu-ws:v1 paknux/ubuntu-ws:v1
```

Lihat hasilnya:
```bash
docker image ls
```

---

### B. Push ke Docker Hub

#### 5. Push ke Docker Hub
```bash
docker push <username-dockerhub>/ubuntu-ws:v1
```

Contoh:
```bash
docker push paknux/ubuntu-ws:v1
```

Tunggu proses upload selesai. Proses ini mungkin memerlukan waktu tergantung ukuran image dan kecepatan internet.

#### 6. Cek di Docker Hub
Buka browser dan akses: https://hub.docker.com/repositories/<username-dockerhub>

Contoh untuk username `paknux`: https://hub.docker.com/repositories/paknux

Pastikan image `ubuntu-ws:v1` sudah muncul di repository Anda.

---

### C. Test dari Host Teman yang Lain

**Absen 2 mengambil image dari repo Docker Hub Absen 1, dan seterusnya (berurutan).**

#### 7. Persiapkan Docker di Instance Baru (jika belum ada)

**Jika Docker belum terinstall:**
```bash
# Opsi 1: Install docker.io (rekomendasi)
sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker $USER
sudo newgrp docker

# Opsi 2: Install via snap (jika docker.io tidak tersedia)
sudo snap install docker
sudo snap start docker
# Note: Snap docker sering perlu sudo untuk perintah docker
```

**Jika mengalami permission denied:**
```bash
# Cek apakah user sudah di group docker
groups

# Jika belum, tambahkan ke group docker
sudo usermod -aG docker $USER
sudo newgrp docker

# Atau gunakan sudo untuk sementara (khusus snap docker)
sudo docker <perintah>
```

#### 8. Pull dari image teman yang lain
```bash
docker pull <username-teman>/ubuntu-ws:v1
```

Contoh jika teman yang LKPD 1 (Absen 1) username-nya `derinugroho`:
```bash
docker pull derinugroho/ubuntu-ws:v1
```

Jika permission denied (untuk snap docker):
```bash
sudo docker pull derinugroho/ubuntu-ws:v1
```

#### 9. Lihat image-nya
```bash
docker image ls
```

Pastikan image dari teman sudah muncul di list image lokal.

#### 10. Buat network untuk komunikasi container
```bash
docker network create mynet
```

#### 11. Siapkan folder aplikasi

**Catatan:** Jika mengalami error "read-only file system" saat mount ke `/var/mywww`, gunakan lokasi di home directory:

```bash
# Opsi 1: Gunakan home directory (rekomendasi untuk menghindari permission issue)
mkdir -p ~/mywww
git clone https://github.com/Deri-Nugroho/docker-2.git ~/mywww
mkdir -p ~/mywww/uploads
sudo chown -R www-data:www-data ~/mywww/uploads

# Opsi 2: Gunakan /var/mywww (jika tidak ada permission issue)
sudo mkdir -p /var/mywww
sudo git clone https://github.com/Deri-Nugroho/docker-2.git /var/mywww
sudo chown -R $USER:$USER /var/mywww
sudo mkdir -p /var/mywww/uploads
sudo chown -R www-data:www-data /var/mywww/uploads
```

#### 12. Jalankan container database (MariaDB)
Aplikasi membutuhkan database server untuk berjalan:

```bash
docker run -d --name dbserver --network mynet -e MYSQL_ROOT_PASSWORD=pass123 mariadb:11-jammy
```

#### 13. Buat database yang dibutuhkan aplikasi
```bash
docker exec dbserver mariadb -u root -ppass123 -e "CREATE DATABASE toko_db;"
```

Atau jika menggunakan snap docker:
```bash
sudo docker exec dbserver mariadb -u root -ppass123 -e "CREATE DATABASE toko_db;"
```

#### 14. Jalankan container web server dari image yang di-pull
```bash
docker run -d \
 --name webserver2 \
 --network mynet \
 -p 8002:80 \
 -v ~/mywww:/var/www/html \
 --restart unless-stopped \
 <username-teman>/ubuntu-ws:v1
```

Contoh:
```bash
docker run -d \
 --name webserver2 \
 --network mynet \
 -p 8002:80 \
 -v ~/mywww:/var/www/html \
 --restart unless-stopped \
 derinugroho/ubuntu-ws:v1
```

**PENTING:** Jika mount error di `/var/mywww`, gunakan `~/mywww` seperti di contoh di atas.

#### 15. Restart container webserver2 (wajib setelah dbserver berjalan)
Container perlu di-restart agar bisa connect ke database yang baru dibuat:

```bash
docker restart webserver2
```

Tunggu 5-10 detik setelah restart sebelum mengakses web server.

#### 16. Verifikasi semua container berjalan
```bash
docker ps
```

Pastikan `dbserver` dan `webserver2` berstatus `Up`.

#### 17. Verifikasi koneksi webserver ke dbserver
```bash
docker exec webserver2 bash -c "mysql -h dbserver -u root -ppass123 -e 'SHOW DATABASES;'"
```

Pastikan database `toko_db` muncul di list.

#### 18. Verifikasi tabel dan data dummy sudah dibuat otomatis
```bash
# Cek tabel
docker exec webserver2 bash -c "mysql -h dbserver -u root -ppass123 toko_db -e 'SHOW TABLES;'"

# Cek data users
docker exec webserver2 bash -c "mysql -h dbserver -u root -ppass123 toko_db -e 'SELECT username, nama_lengkap, role FROM users;'"

# Cek data barang
docker exec webserver2 bash -c "mysql -h dbserver -u root -ppass123 toko_db -e 'SELECT * FROM barang;'"
```

#### 19. Akses web server dari container teman
```bash
curl http://localhost:8002
```

Atau coba akses login.php langsung:
```bash
curl http://localhost:8002/login.php
```

Atau buka lewat browser: `http://<IP-server>:8002`

Jika mengalami error 500 atau tidak bisa connect:
- Pastikan sudah menjalankan langkah 15 (restart webserver2)
- Cek log container: `docker logs webserver2`
- Cek error log Apache: `docker exec webserver2 cat /var/log/apache2/error.log`

---

### D. Perintah Bantuan (Troubleshooting)

| Kebutuhan | Command |
|-----------|---------|
| Cek status login Docker | `docker info` (lihat bagian Username) |
| Logout dari Docker Hub | `docker logout` |
| Lihat tag image | `docker image ls` |
| Hapus image lokal | `docker rmi <nama-image>` |
| Hapus image dari Docker Hub | Hapus melalui web interface di hub.docker.com |
| Lihat layer image | `docker history <nama-image>` |
| Cek size image | `docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"` |
| Lihat log container | `docker logs -f <nama-container>` |
| Masuk ke shell container | `docker exec -it <nama-container> bash` |
| Restart container | `docker restart <nama-container>` |
| Hapus container | `docker rm -f <nama-container>` |
| Lihat network & container | `docker network inspect mynet` |
| Hapus network | `docker network rm mynet` |

---

### E. Error yang Sering Terjadi dan Solusinya

#### 1. Permission Denied saat menjalankan docker

**Error:**
```
permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
```

**Solusi:**
```bash
# Tambahkan user ke group docker
sudo usermod -aG docker $USER
sudo newgrp docker

# Jika masih error, coba restart docker service
sudo systemctl restart docker

# Atau gunakan sudo (khusus untuk snap docker)
sudo docker <perintah>
```

#### 2. Group docker tidak ada (Snap Docker)

**Error:**
```
usermod: group 'docker' does not exist
```

**Solusi:**
```bash
# Start snap docker service
sudo snap start docker
sudo snap restart docker

# Gunakan sudo untuk semua perintah docker
sudo docker pull <image>
sudo docker run <container>
```

**Rekomendasi:** Uninstall snap docker dan install docker.io:
```bash
sudo snap remove docker
sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker $USER
sudo newgrp docker
```

#### 3. Read-only file system saat mount volume

**Error:**
```
error while creating mount source path '/var/mywww': mkdir /var/mywww: read-only file system
```

**Solusi:**
Gunakan lokasi di home directory:
```bash
# Hapus container yang gagal
docker rm webserver2

# Gunakan ~/mywww bukan /var/mywww
mkdir -p ~/mywww
git clone https://github.com/Deri-Nugroho/docker-2.git ~/mywww
mkdir -p ~/mywww/uploads
sudo chown -R www-data:www-data ~/mywww/uploads

# Jalankan container dengan path yang baru
docker run -d \
 --name webserver2 \
 --network mynet \
 -p 8002:80 \
 -v ~/mywww:/var/www/html \
 --restart unless-stopped \
 <username-teman>/ubuntu-ws:v1
```

#### 4. HTTP 500 Internal Server Error

**Error:**
```
curl: (7) Failed to connect to localhost port 8002
```
Atau response HTTP 500.

**Solusi:**
```bash
# Pastikan database server sudah berjalan
docker ps

# Buat database
docker exec dbserver mariadb -u root -ppass123 -e "CREATE DATABASE toko_db;"

# Restart webserver container agar connect ke database
docker restart webserver2

# Tunggu 5-10 detik
sleep 5

# Cek log container
docker logs webserver2

# Cek error log Apache
docker exec webserver2 cat /var/log/apache2/error.log

# Coba akses lagi
curl http://localhost:8002/login.php
```

#### 5. Database connection failed (Temporary failure in name resolution)

**Error di log:**
```
PHP Warning: mysqli::__construct(): php_network_getaddresses: getaddrinfo for dbserver failed: Temporary failure in name resolution
```

**Solusi:**
```bash
# Pastikan kedua container ada di network yang sama
docker network inspect mynet

# Pastikan dbserver berjalan
docker ps

# Restart webserver2
docker restart webserver2

# Verifikasi koneksi
docker exec webserver2 bash -c "mysql -h dbserver -u root -ppass123 -e 'SHOW DATABASES;'"
```

#### 6. Container sudah ada (Conflict)

**Error:**
```
docker: Error response from daemon: Conflict. The container name "/webserver2" is already in use
```

**Solusi:**
```bash
# Hapus container yang ada
docker stop webserver2
docker rm webserver2

# Atau paksa hapus
docker rm -f webserver2

# Jalankan ulang container
docker run -d --name webserver2 ...
```

---

### F. Praktik Baik dalam Docker Image

1. **Gunakan tag yang bermakna:**
   - `v1.0`, `v1.1`, `latest` untuk versi yang stabil
   - `dev`, `test` untuk versi development

2. **Jangan commit data sensitif:**
   - Jangan include password, API key, atau data rahasia dalam image
   - Gunakan environment variable atau Docker secret untuk data sensitif

3. **Optimalkan ukuran image:**
   - Gunakan alpine image jika memungkinkan
   - Bersihkan cache apt/yum setelah install package
   - Gunakan `.dockerignore` untuk exclude file yang tidak perlu

4. **Dokumentasi:**
   - Tuliskan deskripsi yang jelas di Docker Hub
   - Include `README.md` di repository dengan cara penggunaan

---

### Catatan
- Pastikan Docker Hub account Anda sudah terverifikasi dan aktif
- Proses push pertama kali mungkin lebih lama karena upload semua layer
- Push berikutnya akan lebih cepat karena hanya upload layer yang berubah
- Gunakan username Docker Hub Anda yang sebenarnya, bukan contoh `paknux` atau `wisnu`
- Saat pull image di instance baru, pastikan untuk:
  - Setup database server (MariaDB) terlebih dahulu
  - Buat database `toko_db`
  - Restart container webserver setelah database berjalan
  - Gunakan `~/mywww` jika mengalami permission issue dengan `/var/mywww`
- Untuk snap docker, hampir semua perintah perlu menggunakan `sudo`
- Rekomendasi: uninstall snap docker dan install docker.io untuk stabilitas yang lebih baik
