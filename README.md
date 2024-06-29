# Laporan Final Project Teknologi Komputasi Awan 

### Kelompok A3
| NRP | Nama Anggota |
|-----|--------------|
| 5027221049 | Arsyad Rizantha Maulana Salim |
| 5027221051 | Ditya Wahyu Ramadan |
| 5027221072 | Zidny Ilman Nafian |
| 5027221073 | Almendo Jekson Darwin Naftali Kambu |

# Problem
Anda adalah seorang lulusan Teknologi Informasi, sebagai ahli IT, salah satu kemampuan yang harus dimiliki adalah Keampuan merancang, membangun, mengelola aplikasi berbasis komputer menggunakan layanan awan untuk memenuhi kebutuhan organisasi.

# Endpoints
#### 1. Analyze Text

  - Endpoint: `POST /analyze`
  - Description: This endpoint accepts a text input and returns the sentiment score of the text.
  - Request:
    ```
    {
        "text": "Your text here"
    }
    ```
  - Response:
    ```
    {
        "sentiment": <sentiment_score>
    }
    ```

#### 2. Retrieve History

  - Endpoint: `GET /history`
  - Description: This endpoint retrieves the history of previously analyzed texts along with their sentiment scores.
  - Response:
    ```
    {
        {
            "text": "Your previous text here",
            "sentiment": <sentiment_score>
        },
        ...
    }
    ```

# Rancangan Arsitektur Cloud
Berikut rancangan arsitektur cloud final project kami.

![](Images/Rancangan%20Arsitektur%20Cloud.png "test")

Untuk membangun arsitektur tersebut, kami  menggunakan layanan awan dari `Digital Ocean` dengan memanfaatkan free-trial sebesar `$200`. Rincian spesifikasi VM beserta harganya adalah sebagai berikut:

| No | Nama                           | Spesifikasi          | Harga |
|--- |--------------------------------|----------------------|------|
| 1  | Backend                        | 2 vCPUs, 4 GB memory | $24  |
| 2  | Database                       | 2 vCPUs, 2 GB memory | $12  |
| 3  | Frontend                       | 1 vCPU, 1 GB memory  | $6   |
| 4  | Load Balancer                  | 1 vCPU, 2 GB memory  | $12  |
| 5  | Storage                        | 50 GB                | $5   |
|    |                                | **Total**            | **$59** |


# Langkah-Langkah Implementasi dan Konfigurasi VM
