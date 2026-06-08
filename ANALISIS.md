# Analisis Perbaikan

## Permasalahan 1

### Gejala

"docker-compose up" gagal parsing / layanan tidak berjalan; error parse atau layanan tidak saling terhubung. (docker-compose.yml)

### Penyebab

- Baris `services` tidak memiliki titik dua (colon) -> file compose tidak valid.
- Nama volume yang dipakai di service (`db` menggunakan `db-data`) berbeda dari deklarasi di bawah (`database-data`).
- `web3` build context salah ditulis `./web33` (typo).
- Host DB di `web1` diset `mysql` sementara service bernama `db`.
- `web2` punya DB_PASS `wrongpassword` (tidak sesuai env DB container).
- `web3` hanya terhubung ke network `backend` (tidak ke `frontend`) sehingga tidak dapat diakses oleh `nginx`.

### Solusi

- Tambahkan colon: `services:` dan perbaiki indentasi sesuai YAML.
- Samakan nama volume (mis. gunakan `db-data` di deklarasi volumes).
- Perbaiki path build `./web3`.
- Set `DB_HOST: db` di semua web service atau gunakan nama service yang konsisten.
- Perbaiki `DB_PASS` agar sesuai `MYSQL_PASSWORD` atau gunakan secrets.
- Masukkan `frontend` pada `web3` networks (sama seperti web1/web2) agar nginx dapat mengaksesnya.

## Permasalahan 2

### Gejala

Container nginx gagal start (config test failed) atau reverse proxy menayangkan 502/host not found; salah arah ke service yang tak ada. (nginx/nginx.conf)

### Penyebab

- File mengandung blok markdown `nginx` sehingga nginx menolak konfigurasi.
- Upstream menggunakan nama `web11` (typo) dan `web3:8080` sementara container PHP/Apache umumnya listen di port 80.

### Solusi

- Hapus fence markdown (```nginx) dan simpan hanya konfigurasi nginx murni.
- Perbaiki upstream ke `server web1:80; server web2:80; server web3:80;`.
- Pastikan semua web service berada pada network yang sama dengan nginx (frontend).

## Permasalahan 3

### Gejala

Skrip inisialisasi MySQL gagal dijalankan atau memasukkan baris dengan placeholder `REPLACE_NIM` & `REPLACE_NAMA`. (db/init.sql)

### Penyebab

- File diawali/ditutup dengan ```sql yang membuat MySQL menganggapnya bukan SQL murni.
- Isi menggunakan placeholder yang tidak diganti.

### Solusi

- Hapus fence ```sql sehingga hanya berisi SQL murni.
- Ganti placeholder dengan data nama dan nim.

## Permasalahan 4

### Gejala

`docker build` gagal dengan pesan "manifest not found" atau image tidak tersedia; services web tidak berjalan. (web\*/Dockerfile)

### Penyebab

- Base image salah ketik: `php:8.2-apach` / `php:8.2-apche` harusnya `php:8.2-apache`.
- `EXPOSE 8080` sementara Apache di image mendengarkan di port 80 (potensi kebingungan).

### Solusi

- Perbaiki menjadi `FROM php:8.2-apache` di semua `web` Dockerfile.
- Set `EXPOSE 80` atau hapus EXPOSE; pastikan nginx proxy_pass menggunakan port yang sesuai (80).

## Permasalahan 5

### Gejala

Halaman web menampilkan teks "ganti ke namamu" & "ganti ke nimmu". (web\*/index.php)

### Penyebab

- File dibuat sebagai template dengan placeholder yang belum diganti.
- Typo pada string identifikasi container.

### Solusi

- Ganti `$nama` dan `$nim` dengan nilai yang benar atau baca dari env jika perlu.
- Perbaiki label Container pada masing-masing file.
