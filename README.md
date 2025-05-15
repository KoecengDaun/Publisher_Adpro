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