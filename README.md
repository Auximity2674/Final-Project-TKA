# Laporan Final Project Teknologi Komputasi Awan 

### Kelompok A3
| NRP | Nama Anggota |
|-----|--------------|
| 5027221049 | Arsyad Rizantha Maulana Salim |
| 5027221051 | Ditya Wahyu Ramadan |
| 5027221072 | Zidny Ilman Nafian |
| 5027221073 | Almendo Jekson Darwin Naftali Kambu |

## I. Permasalahan
Anda adalah seorang lulusan Teknologi Informasi, sebagai ahli IT, salah satu kemampuan yang harus dimiliki adalah Keampuan merancang, membangun, mengelola aplikasi berbasis komputer menggunakan layanan awan untuk memenuhi kebutuhan organisasi.

Pada suatu saat anda mendapatkan project untuk mendeploy sebuah aplikasi Sentiment Analysis dengan komponen Backend menggunakan python: <a href="attachments/backend/sentiment-analysis.py">sentiment-analysis.py</a>. Kemudian juga disediakan sebuah Frontend sederhana menggunakan <a href="attachments/frontend/index.html">index.html</a> dan <a href="attachments/frontend/styles.css">styles.css</a> yang telah ditetapkan.

Kemudian anda diminta untuk mendesain arsitektur cloud yang sesuai dengan kebutuhan aplikasi tersebut. Apabila dana maksimal yang diberikan adalah 1 juta rupiah per bulan (65 US$) konfigurasi cloud terbaik seperti apa yang bisa dibuat?

## II. Rancangan Infrastruktur Cloud
Berikut rancangan infrastruktur cloud untuk final project kami.

![infrastruktur](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/1f7b28bd-7552-4729-a1b3-3c25ec45b113 "Rancangan infrastruktur cloud")

Untuk membangun infrastruktur tersebut, kami menggunakan layanan awan dari `Google Cloud Platform` dengan memanfaatkan free trial sebesar `$300`. Rincian spesifikasi VM beserta harganya adalah sebagai berikut:

| No | Nama                           | Spesifikasi          | Harga   |
|--- |--------------------------------|----------------------|---------|
| 1  | Worker 1                       | 2 vCPUs, 2 GB RAM    | $18     |
| 2  | Worker 2                       | 2 vCPUs, 2 GB RAM    | $18     |
| 3  | Load Balancer                  | Bawaan Google Cloud  | -       |
|    |                                | **Total**            | **$36** |


## III. Implementasi dan Konfigurasi

### Google Cloud Platform
#### Pembuatan VM Instance dengan Google Compute Engine
1. Masuk ke [Google Cloud Console](https://console.cloud.google.com/) lalu masuk ke Compute Engine
2. Pada VM instances, buat instance baru dengan klik tombol "Create Instance"
3. Pilih region Jakarta, zone terserah, lalu pilih machine configuration E2
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/af4a951e-fb69-4e1f-80f3-5dd458639de3)
4. Pada machine type, pilih custom dan sesuaikan dengan spesifikasi. Yakni 2 vCPUs dan 2 GB RAM
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/835c55da-0f5b-4440-bf6b-7b5250c7165c)
5. Pada firewall, centang semua agar VM bisa diakses secara umum
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/e99d7250-d594-461d-83bb-61f228d460be)
6. Jika sudah, klik tombol create di bagian bawah layar
7. Ulangi sekali lagi untuk VM instance worker 2

#### Konfigurasi VM Worker 1
1. Masuk via SSH dengan klik tombol SSH pada kolom Connect di halaman VM instances
2. Akan diminta authorization, klik saja Authorize
3. Saat sudah masuk, harusnya akan terlihat seperti ini
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/53d106e2-7a88-43ad-9b7b-af7ff62da93d)
4. Install MongoDB dengan mengikuti [guide resmi](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-debian/), pertama kami install package yang wajib terlebih dahulu
```
sudo apt-get install gnupg curl
```
5. Lalu import public GPG key
```
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor
```
6. Buat keyring file (ini untuk Debian 12, sesuai default VM Google Compute Engine)
```
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] http://repo.mongodb.org/apt/debian bookworm/mongodb-org/7.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```
7. Update database package
```
sudo apt-get update
```
8. Install package MongoDB
```
sudo apt-get install -y mongodb-org
```
9. Sebelum start service, kami edit dulu config file MongoDB agar bisa diakses oleh VM worker 2
```
nano /etc/mongod.conf
```
10. Pada `bindIp` tambahkan IP internal VM worker 1, nanti ini akan mengizinkan VM worker 2 untuk mengakses MongoDB melalui IP internal worker 1. Karena menggunakan IP internal, sehingga perangkat di luar jaringan internal VM kami tidak dapat mengakses MongoDB ini.

![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/b0a78fd8-3da8-4c96-a168-b24bf25357d4)

11. Exit menggunakan Ctrl + X, lalu Y, dan terakhir Enter. Setelah itu kami menghidupkan MongoDB dengan perintah ini
```
sudo systemctl start mongod
```
12. Pastikan MongoDB telah aktif
```
sudo systemctl status mongod
```
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/4ec63f93-3935-4cd5-855d-a0459b3a2687)
13. Agar MongoDB selalu aktif setiap reboot VM, kami jalan perintah ini
```
sudo systemctl enable mongod
```
14. Selanjutnya, install Python serta `pip` untuk install package yang dibutuhkan untuk web server dan database
```
sudo apt-get install -y python3 python3-pip python3-full
```
15. Buat virtual environment untuk install package web server dan database, kami beri nama `worker-env`
```
python3 -m venv worker-env
```
16. Lalu aktifkan virtual environment dengan perintah ini
```
source worker-env/bin/activate
```
17. Akan terlihat bahwa virtual environment telah aktif jika ada tulisan di sebelah username user

![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/c3846e3f-e8d1-48b2-94ba-5cdfaee5921e)

18. Lalu install semua package Python yang dibutuhkan dengan `pip`
```
pip install flask flask_cors gunicorn flask_pymongo textblob pymongo gevent
```
19. [TBA]

#### Konfigurasi VM Worker 2
1. [TBA]

#### Konfigurasi Group Instance untuk Google Cloud Load Balancer
1. [TBA]

## IV. Pengetesan Endpoint
### POST - /analyze
#### Worker 1
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/f08ce1fa-44c3-4c4b-925f-2e2f73bdd7bd)
Teks berhasil terkirim ke endpoint dengan metode POST, terlihat dari respons yang memberi kembali nilai sentiment.
#### Worker 2
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/99229450-516c-46c3-8056-020ef19ce5aa)
Teks berhasil terkirim ke endpoint dengan metode POST, terlihat dari respons yang memberi kembali nilai sentiment.

### GET - /history
#### Worker 1
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/ecf91b86-51a6-4f57-b20e-1e357d97a201)
Request berhasil terkirim ke endpoint dengan metode GET, terlihat dari respons yang memberi kembali data yang tersimpan pada MongoDB.
#### Worker 2
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/0f157ad8-5f8e-4c68-af74-77812b0c4d81)
Request berhasil terkirim ke endpoint dengan metode GET, terlihat dari respons yang memberi kembali data yang tersimpan pada MongoDB.

## V. Pengetesan Load Balancing
### Peak RPS, 60s
#### Worker 1
![total_requests_per_second_1719673705 774](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/de06dcd8-12d0-4d52-8ff2-cbe73a5fb55c)
RPS maksimal tercatat pada 93.4 requests/sec. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/c4bdb4c8-bf79-4975-9ee2-59f1e62937bd)

#### Worker 2
[TBA]

### Peak Concurrency, Spawn Rate 50, 60s
#### Worker 1
![total_requests_per_second_1719675042 25](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/39297aeb-7662-41cc-96fd-aa7306a59944)
Concurrency maksimal tercatat pada 400 users. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.

#### Worker 2
[TBA]

### Peak Concurrency, Spawn Rate 100, 60s
#### Worker 1
![total_requests_per_second_1719675353 113](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/661cffe8-3f9c-4c50-aba2-820dfeede168)
Concurrency maksimal tercatat pada 400 users. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.
#### Worker 2
[TBA]


### Peak Concurrency, Spawn Rate 200, 60s
#### Worker 1
![total_requests_per_second_1719675625 578](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/300eb604-352c-45ec-b9ad-8b59c6b8c7fc)
Concurrency maksimal tercatat pada 400 users. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.
#### Worker 2
[TBA]


### Peak Concurrency, Spawn Rate 500, 60s
#### Worker 1
![total_requests_per_second_1719676232 613](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/86f105e6-1291-4cde-b2cc-33b99a8ef598)
Concurrency maksimal tercatat pada 500 users. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.
#### Worker 2
[TBA]

## VI. Saran dan Kesimpulan
[TBA]
