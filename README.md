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

| No | Nama                           | Spesifikasi          | Harga |
|--- |--------------------------------|----------------------|------|
| 1  | Worker 1                        | 2 vCPUs, 2 GB RAM | $18  |
| 2  | Worker 2                        | 2 vCPUs, 2 GB RAM | $18  |
| 3  | Load Balancer                        | Bawaan Google Cloud | -  |
|    |                                | **Total**            | **$36** |


## III. Implementasi dan Konfigurasi

### Google Cloud Platform
#### Pembuatan VM Instance untuk Worker 1 dan Worker 2
1. Masuk ke [Google Cloud Console](https://console.cloud.google.com/) lalu masuk ke Compute Engine
2. Pada VM instances, buat instance baru dengan klik tombol "Create Instance"
3. Pilih region Jakarta, zone terserah, lalu pilih machine configuration E2
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/af4a951e-fb69-4e1f-80f3-5dd458639de3)
4. Pada machine type, pilih custom dan sesuaikan dengan spesifikasi. Yakni 2 vCPUs dan 2 GB RAM
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/835c55da-0f5b-4440-bf6b-7b5250c7165c)
5. Pada firewall, centang semua agar VM bisa diakses secara umum
![image](https://github.com/Auximity2674/Final-Project-TKA/assets/134349363/e99d7250-d594-461d-83bb-61f228d460be)
6. Jika sudah, klik tombol create di bagian bawah layar
#### Konfigurasi VM Worker 1
[TBA]
