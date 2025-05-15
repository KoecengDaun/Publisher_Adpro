## **a. Berapa banyak data yang dikirim publisher dalam satu kali run?**

* **Jumlah pesan**: kode yang kita tulis mengirim **5 event** (`publish_event` dijalankan 5 kali).
* **Payload per pesan** (Borsh-serialized):

  * Field `user_id` (misal `"1"`) → `4 byte` length prefix + `1 byte` data = **5 byte**
  * Field `user_name` (misal `"129500004y-Amir"`, 15 karakter) → `4 byte` length prefix + `15 byte` data = **19 byte**
  * **Total payload** ≈ 5 + 19 = **24 byte** per message
* **Total payload** (tanpa overhead AMQP):

  ```
  5 pesan × 24 byte ≈ 120 byte
  ```

  Jika ditambah framing/header AMQP, tiap pesan bakal menambah puluhan byte lagi, tapi payload murni aplikasi Anda \~120 byte.

---

## **b. Arti `amqp://guest:guest@localhost:5672` sama di publisher & subscriber**
Format URI:

```
amqp://<username>:<password>@<host>:<port>
```

* **`guest:guest`** → username = `guest`, password = `guest` (default RabbitMQ)
* **`localhost`** → broker berjalan di mesin yang sama (loopback)
* **`5672`** → port standar AMQP

## Gambar

### 1. Membuka Rabbit MQ
![Gambar1](publisher\Gambar\Gambar_1.png)

### 2. Mengirim dan prosesing
![Gambar 2](Gambar/Gambar_2.png)
#### Hal yang terjadi
Gambar tersebut menunjukkan alur publish-subscribe antara dua program Rust (publisher dan subscriber) yang berjalan di WSL:

1. **Publisher Mengirim Event**

   * Begitu `target/debug/publisher` dijalankan, ia mem-publish lima pesan `UserCreatedEventMessage` ke queue `user_created` (tanpa log output khusus di screenshot, karena `publish_event` dipanggil tanpa `println!`).

2. **Subscriber Menerima & Memproses**

   * Terminal kiri: `target/debug/subscriber` menerima setiap event dari RabbitMQ.
   * Setiap kali pesan masuk, subscriber menampilkan:

     ```
     In Ade’s Computer [2306244892], Message received: UserCreatedEventMessage { user_id: "1", user_name: "2306244892-Amir" }
     Processed in 36.387µs
     ```
   * Karena delay 1 detik (`thread::sleep`) dikomentari, waktu proses yang tercatat hanya puluhan mikro-detik.

### 3. Perubahan RabbitMQ
![Gambar 3](Gambar/Gambar_3.png)

Spike ini terjadi karena cargo run menjalankan publisher yang mem-publish 5 pesan berturut-turut. Management UI mencatat laju publish dan deliver dalam pesan per detik, sehingga Anda dapat melihat lonjakan singkat (spikes) pada momen tersebut.