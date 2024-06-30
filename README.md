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

Pada suatu saat anda mendapatkan project untuk mendeploy sebuah aplikasi Sentiment Analysis dengan komponen Backend menggunakan python: <a href="https://github.com/fuaddary/fp-tka/blob/main/Resources/BE/sentiment-analysis.py">sentiment-analysis.py</a>. Kemudian juga disediakan sebuah Frontend sederhana menggunakan <a href="https://github.com/fuaddary/fp-tka/blob/main/Resources/FE/index.html">index.html</a> dan <a href="https://github.com/fuaddary/fp-tka/blob/main/Resources/FE/styles.css">styles.css</a> yang telah ditetapkan.

Kemudian anda diminta untuk mendesain arsitektur cloud yang sesuai dengan kebutuhan aplikasi tersebut. Apabila dana maksimal yang diberikan adalah 1 juta rupiah per bulan (65 US$) konfigurasi cloud terbaik seperti apa yang bisa dibuat?

## II. Rancangan Infrastruktur Cloud
Berikut rancangan infrastruktur cloud untuk final project kami.

![infrastruktur](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/1f7b28bd-7552-4729-a1b3-3c25ec45b113 "Rancangan infrastruktur cloud")

Untuk membangun infrastruktur tersebut, kami menggunakan layanan awan dari `Google Cloud Platform` dengan memanfaatkan free trial sebesar `$300`. Rincian spesifikasi VM beserta harganya adalah sebagai berikut:

| No | Nama                           | Spesifikasi          | Harga   |
|--- |--------------------------------|----------------------|---------|
| 1  | Worker 1                       | 2 vCPUs, 2 GB RAM, Debian 12    | $18     |
| 2  | Worker 2                       | 2 vCPUs, 2 GB RAM, Debian 12   | $18     |
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
19. Setelah itu, kami membuat file baru untuk web server
```
nano sentiment-analysis.py
```
20. Setelah masuk `nano`, kami copy paste file yang telah disediakan lalu mengganti IP MongdoDB menjadi `localhost` karena database terdapat pada VM ini. Karena kami tidak mengubah port, sehingga kami memakai port default bawaan MongoDB.
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/e0b8e427-7c3c-420a-8179-9645f2091b60)
21. Lalu kami run web server dengan menjalankan perintah ini
```
python sentiment-analysis.py
```
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/7ed5c06e-1b57-474f-b466-9dd76000ef6e)


#### Konfigurasi VM Worker 2
1. Langkah tidak jauh berbeda dengan VM worker 1, karena di sini hanya ada web server sehingga kami cukup menginstall yang berhubungan dengan web server saja. Yakni Python dan package-package yang dibutuhkan.
```
sudo apt-get update
sudo apt-get install -y python3 python3-pip python3-full
python3 -m venv worker-env
source worker-env/bin/activate
pip install flask flask_cors gunicorn flask_pymongo textblob pymongo gevent
```
2. Lalu kami membuat file baru untuk web server, sama seperti VM worker 1. Namun di sini, karena database terdapat pada VM worker 1, kami memasukkan IP internal VM worker 1 sebagai IP database.
```
nano sentiment-analysis.py
```
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/2c95b821-15c4-445d-b1ca-f0e6207241f3)

3. Setelah itu kami run seperti sebelumnya
```
python sentiment-analysis.py
```
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/323d7dfa-5d4d-44f0-8e08-14cc47570e8e)


#### Konfigurasi Group Instance untuk Google Cloud Load Balancer
1. Untuk mengaktifkan load balancer bawaan Google Cloud, kami perlu mengelompokkan VM worker 1 dan VM worker 2 menjadi satu group instance. Pertama kami masuk pada [halaman instance groups](https://console.cloud.google.com/compute/instanceGroups)
2. Klik tombol Create Instance Group di atas layar
3. Pilih new unmanaged instance group, lalu kami pilih yang sesuai dengan konfigurasi VM kami sebelumnya.
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/d546b8cf-9b8d-4567-bd50-46ef5cd1f821)

4. Klik tombol Create di bawah layar
5. Setelah itu masuk ke [halaman load balancing](https://console.cloud.google.com/net-services/loadbalancing/list/loadBalancers). Klik Create Load Balancer di atas layar.
6. Biarkan konfigurasi seperti default.
7. Klik tombol Create di bawah layar.
8. Semua sudah sesuai, saat kami klik load balancer yang telah dibuat. Maka akan muncul IP load balancer.
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/5f5e9b8a-e74f-4631-918a-974cc904efc5)


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
![total_requests_per_second_1719679002 57](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/7d7b47b2-589b-4d7e-a29c-4c462e07b325)
Menggunakan spawn rate 450, RPS maksimal tercatat pada 135.1 requests/sec. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/0dd093c6-1100-4720-be5b-618ba5f057cf)

#### Worker 2
![total_requests_per_second_1719678592 279](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/bde4d351-2319-4353-9562-c730b7f9d89d)
Menggunakan spawn rate 450, RPS maksimal tercatat pada 145.83 requests/sec. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/592a3610-0271-4baf-ae2b-6dc8e4e8eada)


### Peak Concurrency, Spawn Rate 50, 60s
#### Worker 1
![total_requests_per_second_1719675042 25](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/39297aeb-7662-41cc-96fd-aa7306a59944)
Concurrency maksimal tercatat pada 400 users. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.

#### Worker 2
![total_requests_per_second_1719677148 251](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/1898b02b-415b-4e39-98a8-9e677e5fbc66)
Concurrency maksimal tercatat pada 400 users. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.

### Peak Concurrency, Spawn Rate 100, 60s
#### Worker 1
![total_requests_per_second_1719675353 113](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/661cffe8-3f9c-4c50-aba2-820dfeede168)
Concurrency maksimal tercatat pada 400 users. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.
#### Worker 2
![total_requests_per_second_1719677322 958](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/f6a5b06f-f214-4130-8651-26d17f250ea2)
Concurrency maksimal tercatat pada 400 users. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.


### Peak Concurrency, Spawn Rate 200, 60s
#### Worker 1
![total_requests_per_second_1719675625 578](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/300eb604-352c-45ec-b9ad-8b59c6b8c7fc)
Concurrency maksimal tercatat pada 400 users. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.
#### Worker 2
![total_requests_per_second_1719677591 688](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/0a96e926-513b-417b-81a0-4f3ee705a8ab)
Concurrency maksimal tercatat pada 380 users. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.


### Peak Concurrency, Spawn Rate 500, 60s
#### Worker 1
![total_requests_per_second_1719679002 57](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/7d7b47b2-589b-4d7e-a29c-4c462e07b325)
Concurrency maksimal tercatat pada 450 users. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.
#### Worker 2
![total_requests_per_second_1719678592 279](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/bde4d351-2319-4353-9562-c730b7f9d89d)
Concurrency maksimal tercatat pada 450 users. Tanpa failure, sehingga dapat dipastikan tes ini berhasil.

## VI. Saran dan Kesimpulan
Dengan budget bulanan sebesar $36, kami dapat mengoptimalkan Google Cloud Platform (GCP) untuk menjalankan dua VM (virtual machine) dengan Google Compute Engine. VM 1 akan menampung worker 1 yang terdiri dari web server dan database, sedangkan VM 2 akan menampung worker 2 yang hanya memiliki web server. Juga didukung oleh load balancer bawaan dari Google Cloud.

Berdasarkan laporan dari Locust, terbukti bahwa peningkatan user spawn rate secara signifikan meningkatkan RPS (Request Per Second) dan concurrency. Namun, perlu diperhatikan bahwa peak concurrency yang berlebihan dapat mengakibatkan kegagalan koneksi server (failure).

Saran ke depan ialah mencoba untuk mengoptimalkan kembali secara spesifikasi VM serta teknik-teknik cloud lainnya untuk meningkatkan RPS dan concurrency tanpa mengkompromikan kestabilan koneksi. Seperti auto-scaling, auto-recovery/auto-revive, caching, dan lainnya.

## Vidio Revisi
https://youtu.be/wDFfvx0fDGU
