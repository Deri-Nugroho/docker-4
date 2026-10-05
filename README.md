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

#### 7. Pull dari image teman yang lain
```bash
docker pull <username-teman>/ubuntu-ws:v1
```

Contoh jika teman yang LKPD 1 (Absen 1) username-nya `wisnu`:
```bash
docker pull wisnu/ubuntu-ws:v1
```

#### 8. Lihat image-nya
```bash
docker image ls
```

Pastikan image dari teman sudah muncul di list image lokal.

#### 9. Lakukan test untuk menjalankan image tersebut sebagai container
```bash
docker run -d \
 --name webserver2 \
 --network mynet \
 -p 8002:80 \
 -v /var/mywww:/var/www/html \
 --restart unless-stopped \
 <username-teman>/ubuntu-ws:v1
```

Contoh:
```bash
docker run -d \
 --name webserver2 \
 --network mynet \
 -p 8002:80 \
 -v /var/mywww:/var/www/html \
 --restart unless-stopped \
 wisnu/ubuntu-ws:v1
```

#### 10. Verifikasi container berjalan
```bash
docker ps
```

Pastikan container `webserver2` berstatus `Up`.

#### 11. Akses web server dari container teman
```bash
curl http://localhost:8002
```

Atau buka lewat browser: `http://<IP-server>:8002`

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

---

### E. Praktik Baik dalam Docker Image

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
