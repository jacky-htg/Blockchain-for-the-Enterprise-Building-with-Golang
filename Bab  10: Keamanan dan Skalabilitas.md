# Bab 10: Keamanan dan Skalabilitas

Bab 10 ini akan membahas dua aspek penting dalam pengembangan jaringan blockchain: keamanan dan skalabilitas. Seiring dengan meningkatnya adopsi blockchain, menjaga keamanan transaksi dan data dalam jaringan terbuka menjadi krusial untuk melindungi integritas jaringan dari berbagai serangan. Di sisi lain, meningkatnya jumlah pengguna juga menuntut jaringan blockchain untuk mampu menangani lebih banyak transaksi tanpa mengorbankan kecepatan atau efisiensi. Tantangan ini mendorong pengembang untuk merancang berbagai mekanisme keamanan, seperti kriptografi dan protokol konsensus, serta mencari solusi skalabilitas yang inovatif, termasuk penggunaan lapisan kedua (Layer-2 solutions) dan arsitektur yang mendukung skala besar. Bab ini akan menguraikan prinsip-prinsip dasar dan teknik-teknik utama yang memastikan blockchain tetap aman dan mampu beradaptasi dengan kebutuhan yang terus berkembang.

## 10.1 Keamanan di Jaringan Blockchain
Keamanan dalam jaringan blockchain merupakan aspek krusial yang memastikan integritas, kerahasiaan, dan ketersediaan data. Blockchain, dengan sifat desentralisasinya, menghadirkan tantangan unik dalam melindungi jaringan dari berbagai ancaman, termasuk serangan 51%, serangan double spending, dan potensi eksploitasi pada smart contracts. Oleh karena itu, penerapan prinsip-prinsip keamanan yang kuat sangat diperlukan untuk menjaga kepercayaan dalam sistem ini.

Salah satu fondasi utama keamanan blockchain terletak pada penggunaan algoritma kriptografi yang kuat, seperti hashing dan enkripsi. Algoritma hashing, misalnya, tidak hanya digunakan untuk menyimpan informasi dengan cara yang tidak dapat diubah, tetapi juga untuk memastikan bahwa setiap perubahan pada data akan mudah terdeteksi. Dengan demikian, setiap blok yang ditambahkan ke dalam rantai tidak hanya terhubung secara kronologis, tetapi juga aman dari modifikasi yang tidak sah. Sementara itu, mekanisme konsensus, seperti Proof of Work, berfungsi untuk mencegah penyerang dari mengambil alih jaringan dengan cara memastikan bahwa setiap transaksi diverifikasi oleh mayoritas peserta sebelum dicatat.

### 10.1.1 Ancaman dan Tantangan dalam Keamanan Blockchain
Walaupun teknologi blockchain menawarkan tingkat keamanan yang tinggi, masih ada beberapa ancaman dan tantangan yang perlu diwaspadai:

**1. Serangan DDoS (Distributed Denial of Service)**

Jaringan blockchain dapat menjadi target serangan DDoS, di mana penyerang berusaha membuat jaringan tidak dapat diakses dengan membanjiri node dengan lalu lintas yang berlebihan. Hal ini dapat mengganggu operasi normal jaringan dan menghambat transaksi.

**2. Keamanan Peer Discovery**

Proses discovery peer yang tidak aman dapat menyebabkan penyerang memperkenalkan node palsu ke dalam jaringan. Node palsu ini dapat berusaha untuk mengumpulkan informasi tentang jaringan atau melakukan serangan lebih lanjut.

**3. Node Malicious**

Jika keamanan peer discovery dapat ditembus, langkah selanjutnya adalah serangan node malicious. Node dalam jaringan blockchain tidak selalu dapat dipercaya. Node yang berperilaku jahat dapat menyebarkan informasi yang salah atau berusaha memanipulasi konsensus. Ini dapat memengaruhi proses voting dan hasil pemilihan dalam aplikasi voting berbasis blockchain.

**4. Serangan Sybil**

Jika serangan node malicious berhasil, bisa dilanjutkan dengan serangan sybil, ini terjadi ketika satu entitas (penyerang) menciptakan banyak identitas atau node palsu di jaringan untuk mendapatkan pengaruh yang tidak seimbang. Tujuannya adalah untuk membanjiri jaringan dengan identitas palsu dan memanipulasi interaksi atau pengambilan keputusan. Dalam jaringan blockchain atau peer-to-peer (P2P) lainnya, serangan ini bisa mempengaruhi:

- Proses validasi transaksi (misalnya, menolak transaksi tertentu).
- Distribusi informasi (mengontrol penyebaran atau pembaruan data di jaringan).
- Pengambilan keputusan (menggunakan jumlah node palsu untuk mempengaruhi voting atau konsensus dalam jaringan).

**5. Serangan 51%**

Ini adalah serangan tindak lanjut ketika serangan sybil berhasil. Dalam serangan ini, sekelompok penyerang menguasai lebih dari 50% kekuatan komputasi jaringan. Hal ini memungkinkan mereka untuk memanipulasi transaksi, mencegah transaksi baru dari dikonfirmasi, dan bahkan membalikkan transaksi yang telah dilakukan.

**6. Double Spending**

Ini adalah masalah yang timbul ketika seorang pengguna mencoba untuk menghabiskan aset digital yang sama lebih dari sekali. Walaupun mekanisme konsensus bertujuan untuk mencegah ini, serangan yang cerdik dapat mengekploitasi kelemahan dalam sistem untuk melakukan double spending. Apalagi jika serangan 51% sudah berhasil, akan sangat mudah melakukan double spending.

**7. Eksploitasi Smart Contracts**

Smart contracts, meskipun memberikan fungsionalitas yang canggih, dapat memiliki celah keamanan yang dapat dieksploitasi. Bug dalam kode dapat dimanfaatkan oleh penyerang untuk mencuri aset atau merusak fungsi kontrak.

**8. Phishing dan Penipuan**

Pengguna blockchain sering menjadi target phishing yang dirancang untuk mencuri informasi pribadi atau akses ke dompet digital. Edukasi pengguna dan praktik keamanan yang baik sangat penting untuk mengurangi risiko ini.

### 10.1.2 Solusi Keamanan

1. Penerapan Rate Limiting

Untuk mencegah serangan DDoS, kita dapat menerapkan rate limiting pada endpoint yang ada di server bootstrap dan node blockchain. Dengan cara ini, kita dapat membatasi jumlah permintaan yang diterima dalam periode waktu tertentu.

Contoh implementasi rate limiting di server bootstrap:

```go
package bootstrap

import (
    "time"
    "sync"
)

type RateLimiter struct {
    mu          sync.Mutex
    requests    map[string]int
    lastRequest map[string]time.Time
    limit       int
    window      time.Duration
}

func NewRateLimiter(limit int, window time.Duration) *RateLimiter {
    return &RateLimiter{
        requests:    make(map[string]int),
        lastRequest: make(map[string]time.Time),
        limit:       limit,
        window:      window,
    }
}

func (rl *RateLimiter) Allow(ip string) bool {
    rl.mu.Lock()
    defer rl.mu.Unlock()

    now := time.Now()
    if last, ok := rl.lastRequest[ip]; ok && now.Sub(last) < rl.window {
        if rl.requests[ip] >= rl.limit {
            return false
        }
        rl.requests[ip]++
    } else {
        rl.requests[ip] = 1
    }

    rl.lastRequest[ip] = now
    return true
}
```

Fungsi ini akan mengecek jumlah request dari 1 IP dalam time window tertentu apakah masih dibawah limitasi yang kita buat. Jika ya maka request akan diproses, dan jika tidak maka request akan ditolak. Kita bisa melakukan konfigurasi rate limit yang diingin ketika memanggil fungsi `NewRateLimiter()`, kemudian jika ada request yang masuk, kita mengecek dengan memanggil fungsi `Allow()`.

Begitu juga di aplikasi blockchain, fungsi rate limiter ini bsia dipasang dengan logika yang sama.

2. Penerapan Hashing pada Blockchain

Kita sudah menerapkan hashing pada Bab 4. Penerapan hashing pada blockchain membuat blockchain mempunyai tingkat keamanan transaksi yang tinggi. Blok-blok dalam blockchain berisi tiga elemen utama: data transaksi, hash dari blok tersebut, dan hash dari blok sebelumnya. Ini menjamin keamanan dalam transaksi blockchain. Memalsukan transkasi blockchain menjadi sangat sulit, karena penyerang tidak bisa memalsukan satu block transaksi saja, melainkan harus membuat ulang seluruh blockchain.

3. Penerapan Konsensus

Kita juga sudah menerapkan konsensus pada Bab 6. Dengan konsensus, seorang penyerang yang memalsukan blockchain, akan ditolak oleh mayoritas jaringan. Penerapan konsensus seperti POW (proof of work) yang membutuhkan komputasi tinggi, membuat penyerang yang ingin menguasai 51% jaringan, harus mengeluarkan ongkos yang sangat tinggi, karena semua transaksi harus melewati POW yang nyata.

4. Verifikasi Peer
5. Deteksi dan Penanganan Node Malicious
6. Tidak menyimpan data sensitif dalam blockchain
7. Penerapan Kriptografi
8. Penggunaan Tanda Tangan Digital
9. Audit Keamanan
10. Edukasi Pengguna

### 10.1.3 Mekanisme Kriptografi dalam Keamanan Blockchain
Pentingnya keamanan di jaringan blockchain tidak dapat diremehkan, mengingat banyaknya aplikasi yang dibangun di atas teknologi ini, mulai dari cryptocurrency hingga sistem voting digital. Di bagian selanjutnya, kita akan mendalami mekanisme kriptografi yang lebih mendalam, serta bagaimana mereka mendukung keamanan transaksi dan integritas data dalam jaringan blockchain. Kriptografi adalah pilar utama dalam menjaga keamanan jaringan blockchain. Terdapat beberapa mekanisme kriptografi yang berperan penting, antara lain:

**Hashing:**

Hashing adalah proses yang mengubah data menjadi string karakter tetap dengan panjang tertentu. Fungsi hash yang umum digunakan dalam blockchain adalah SHA-256. Fungsi ini memiliki sifat deterministik, artinya data yang sama akan selalu menghasilkan hash yang sama. Hal ini memudahkan untuk memverifikasi integritas data, karena perubahan sekecil apa pun pada input akan menghasilkan hash yang sangat berbeda. Dengan cara ini, jika ada upaya untuk memodifikasi blok yang telah dicatat, hash dari blok tersebut akan berubah, dan jaringan dapat mendeteksi adanya penipuan.

**Enkripsi:**

Enkripsi digunakan untuk menjaga kerahasiaan data. Dalam konteks blockchain, data yang sensitif, seperti informasi identitas pengguna, dapat dienkripsi untuk melindunginya dari akses yang tidak sah. Enkripsi kunci publik, seperti RSA, memungkinkan pengguna untuk berbagi informasi dengan aman tanpa perlu bertukar kunci rahasia sebelumnya. Hanya penerima yang memiliki kunci pribadi yang dapat mendekripsi pesan.

**Tanda Tangan Digital:**

Tanda tangan digital merupakan metode yang memastikan otentikasi dan integritas pesan. Dalam blockchain, setiap transaksi dapat ditandatangani secara digital menggunakan kunci pribadi pengirim. Tanda tangan ini kemudian dapat diverifikasi oleh penerima menggunakan kunci publik pengirim. Dengan cara ini, penerima dapat yakin bahwa transaksi tersebut benar-benar dilakukan oleh pengirim yang sah dan tidak telah dimodifikasi.

**Mekanisme Konsensus:**

Selain kriptografi, mekanisme konsensus memainkan peran penting dalam keamanan blockchain. Seperti yang telah disebutkan, Proof of Work (PoW) adalah salah satu metode yang paling umum digunakan untuk mencegah serangan. Dengan mengharuskan peserta jaringan (penambang) untuk memecahkan teka-teki matematis yang kompleks sebelum menambahkan blok baru ke dalam rantai, PoW menciptakan penghalang yang signifikan bagi penyerang. Namun, PoW juga memiliki kelemahan, seperti konsumsi energi yang tinggi, yang memicu pengembangan alternatif seperti Proof of Stake (PoS), di mana peserta harus mengunci sejumlah cryptocurrency sebagai jaminan untuk dapat berpartisipasi dalam proses verifikasi.


### 10.1.4 Praktik Terbaik
Untuk menjaga keamanan jaringan blockchain, beberapa praktik terbaik dapat diadopsi:

**Audit Keamanan:**

Melakukan audit rutin pada kode smart contracts dan infrastruktur blockchain untuk mengidentifikasi dan memperbaiki potensi kerentanan.

**Edukasi Pengguna:**

Memberikan informasi dan pelatihan kepada pengguna tentang cara mengidentifikasi dan menghindari ancaman, seperti phishing dan penipuan.

**Pengembangan Protokol yang Kuat:**

Memastikan bahwa protokol blockchain dirancang dengan mempertimbangkan keamanan dari awal, termasuk mekanisme konsensus yang efisien dan perlindungan terhadap serangan.

Keamanan dalam jaringan blockchain adalah topik yang kompleks dan dinamis, yang terus berkembang seiring dengan evolusi teknologi dan munculnya ancaman baru. Melalui pemahaman mendalam tentang mekanisme kriptografi, ancaman yang ada, dan penerapan praktik terbaik, kita dapat memastikan bahwa sistem blockchain tetap aman, dapat diandalkan, dan siap untuk digunakan dalam berbagai aplikasi di masa depan. Kepercayaan dalam teknologi ini sangat tergantung pada upaya kolektif untuk menjaga keamanan dan integritasnya.
