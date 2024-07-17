# Bab 2 : Dasar-dasar Blockchain.md
## 2.1 Blok dan Rantai Blok
### Blok dalam Blockchain

Blok adalah unit dasar dari teknologi blockchain. Setiap blok menyimpan sejumlah transaksi yang terjadi dalam jaringan pada suatu periode waktu tertentu. Struktur blok umumnya terdiri dari beberapa elemen utama:

1. Header Blok: Bagian ini berisi metadata penting seperti hash blok sebelumnya, waktu pembuatan blok, dan informasi tambahan yang diperlukan untuk memvalidasi keabsahan blok.

2. Data Transaksi: Merupakan bagian yang menyimpan informasi tentang transaksi yang dimasukkan ke dalam blok tersebut. Data ini bisa berupa transfer aset, eksekusi smart contract, atau informasi lain yang relevan untuk aplikasi blockchain tertentu.

3. Hash: Sebuah nilai hash kriptografis yang unik yang dihasilkan dari seluruh isi blok, termasuk header dan data transaksi. Hash ini digunakan untuk memastikan integritas data blok dan memvalidasi blok di dalam rantai.

4. Nonce: Nilai yang digunakan dalam proses proof-of-work (PoW) untuk mencari hash blok yang memenuhi kriteria tertentu. Nonce disesuaikan secara iteratif hingga sebuah hash blok yang valid ditemukan.

### Blockchain

Blockchain adalah kumpulan blok yang terhubung secara kronologis dan terdesentralisasi. Setiap blok dalam rantai blockchain memiliki referensi ke blok sebelumnya dengan menggunakan nilai hash blok sebelumnya dalam header bloknya. Ini menciptakan struktur data yang tidak dapat diubah dan transparan, di mana setiap perubahan atau penambahan blok baru akan terlihat dalam seluruh jaringan.

Struktur Blockchain terdiri dari beberapa komponen kunci yang memberikan fungsionalitas dan keamanan pada sistem secara keseluruhan:

1. Header: Setiap blok dalam blockchain memiliki header yang berisi metadata seperti versi protokol, timestamp (waktu pembuatan), nonce (untuk PoW), dan hash blok sebelumnya.

2. Data Transaksi: Bagian ini menyimpan informasi tentang transaksi yang termasuk dalam blok tersebut. Transaksi ini mungkin termasuk transfer aset digital, eksekusi smart contract, atau operasi lain yang diizinkan oleh aturan protokol blockchain yang bersangkutan.

3. Hash: Sebuah nilai hash kriptografis yang unik yang dihasilkan dari seluruh konten blok. Hash ini digunakan untuk memverifikasi integritas data blok. Jika isi blok berubah, hashnya juga akan berubah, sehingga memastikan ketidakbisaan memalsukan atau mengubah blok tanpa terdeteksi.

4. Nonce: Nilai acak yang disesuaikan secara iteratif dalam proses proof-of-work (PoW). Tujuan nonce adalah untuk menciptakan hash blok yang memenuhi kriteria tertentu, seperti awalan dengan sejumlah nol, yang menunjukkan bukti bahwa pekerjaan komputasi telah dilakukan untuk memvalidasi blok.

### Fungsi Penting dalam Blockchain

1. Immutability (Ketidakbisaan untuk Diubah): Karena setiap blok mengandalkan hash blok sebelumnya dalam rantai, mengubah satu blok akan memerlukan perubahan pada semua blok yang mengikutinya. Ini membuat sistem blockchain sangat sulit untuk dimanipulasi, karena perubahan yang dicoba akan terdeteksi oleh jaringan sebagai anomali.

2. Decentralization (Desentralisasi): Blockchain menggunakan jaringan node yang tersebar secara geografis dan terhubung satu sama lain. Setiap node memiliki salinan lengkap dari blockchain, dan setiap transaksi harus disetujui secara konsensus oleh mayoritas node dalam jaringan. Ini menghilangkan kebutuhan akan otoritas sentral dan menjaga keamanan dan keadilan sistem.

3. Transparency (Transparansi): Semua transaksi dalam blockchain dapat dilihat oleh semua peserta jaringan. Setiap perubahan atau penambahan pada blok tercatat secara publik dan permanen. Ini meningkatkan akuntabilitas dan mengurangi potensi kecurangan atau manipulasi data.

4. Security (Keamanan): Keamanan blockchain didasarkan pada kriptografi yang kuat dan algoritma konsensus yang dipilih. PoW, PoS, dan mekanisme konsensus lainnya memastikan bahwa data yang dimasukkan ke dalam blockchain sah dan tidak dapat diubah tanpa seizin jaringan.

5. Smart Contracts: Platform blockchain seperti Ethereum mendukung smart contracts, yang merupakan kode komputer yang berjalan otomatis berdasarkan kondisi yang diprogram. Ini memungkinkan eksekusi otomatis dari kontrak atau perjanjian, memotong perantara, dan mengurangi biaya administrasi.

Dengan struktur dan fungsi ini, blockchain mengubah cara kita berinteraksi dengan data, transaksi, dan kontrak di berbagai sektor, dari keuangan hingga logistik, dari kesehatan hingga manufaktur. Keunggulannya dalam keamanan, transparansi, dan desentralisasi menjadikannya teknologi yang potensial untuk revolusi digital di masa depan.

## 2.2 Hashing dan Kriptografi

Dalam teknologi blockchain, hashing dan kriptografi memainkan peran penting dalam memastikan keamanan dan integritas data. Hashing adalah proses mengubah data input menjadi nilai hash tetap panjang menggunakan fungsi hash kriptografis. Fungsi hash ini memiliki beberapa karakteristik kunci:

- Uniqueness (Keunikan): Setiap input data akan menghasilkan nilai hash yang unik. Meskipun dua input data memiliki sedikit perbedaan, nilai hash yang dihasilkan akan sangat berbeda.
- Deterministic (Deterministik): Setiap kali input yang sama dimasukkan ke dalam fungsi hash yang sama, nilai hash yang dihasilkan akan selalu identik.
- Fixed Length (Panjang Tetap): Nilai hash yang dihasilkan memiliki panjang tetap, tidak peduli seberapa besar atau kecil input data yang digunakan.

Fungsi hash kriptografis yang digunakan dalam blockchain harus memenuhi standar keamanan yang tinggi, seperti SHA-256 (Secure Hash Algorithm 256-bit). Ini digunakan untuk menghasilkan hash dari blok dalam blockchain, yang mencakup informasi seperti data transaksi, timestamp, dan hash blok sebelumnya.

Kriptografi dalam Blockchain

Selain hashing, kriptografi juga digunakan untuk tujuan lain dalam blockchain, termasuk:

- Public-key Cryptography: Sistem kriptografi yang menggunakan sepasang kunci, yaitu kunci publik dan kunci privat, untuk mengenkripsi dan mendekripsi data. Ini digunakan untuk mengamankan transaksi dan mengidentifikasi pengguna dalam blockchain.
- Digital Signatures: Digunakan untuk memverifikasi keaslian dan integritas transaksi. Setiap transaksi ditandatangani dengan kunci privat pengirim dan dapat diverifikasi menggunakan kunci publiknya.
- Encryption (Enkripsi): Melindungi data sensitif yang disimpan di blockchain dengan mengenkripsinya menggunakan algoritma kriptografi yang kuat.

Penerapan hashing dan kriptografi yang tepat dalam blockchain memastikan bahwa data tetap aman, transaksi tidak dapat dipalsukan, dan keamanan sistem terjaga dengan baik.

## 2.3 Transaksi dan Konsensus

Dalam teknologi blockchain, transaksi dan konsensus adalah dua komponen utama yang memungkinkan jaringan blockchain untuk berfungsi dengan aman dan terdesentralisasi.

### Transaksi dalam Blockchain

Transaksi adalah tindakan pengiriman atau penerimaan aset digital atau informasi di dalam jaringan blockchain. Setiap transaksi terdiri dari beberapa elemen kunci:

- Input dan Output: Input adalah informasi yang menunjukkan asal atau sumber dana atau informasi, sementara output adalah hasil dari transaksi tersebut, yang menunjukkan penerima atau tujuan dana atau informasi.
- Tanda Tangan Digital: Setiap transaksi menggunakan tanda tangan digital untuk mengotentikasi bahwa pengguna yang memulai transaksi adalah pemilik sah dari aset atau informasi yang ditransfer.
- Hashing: Data transaksi di-hash untuk menciptakan identifikasi unik yang dikenal sebagai hash transaksi. Ini membantu dalam memverifikasi integritas dan keaslian transaksi.

### Konsensus dalam Blockchain

Konsensus adalah proses di mana semua node dalam jaringan blockchain mencapai kesepakatan tentang keabsahan dan urutan transaksi yang masuk ke dalam blockchain. Beberapa mekanisme konsensus yang umum digunakan meliputi:

- Proof of Work (PoW): Digunakan oleh Bitcoin dan beberapa blockchain lainnya, PoW melibatkan kompetisi di antara node untuk memecahkan masalah matematika yang kompleks. Node yang pertama kali menyelesaikan masalah ini diberi wewenang untuk menambahkan blok baru ke blockchain dan menerima imbalan dalam bentuk kripto.
- Proof of Stake (PoS): Berbeda dengan PoW, PoS menentukan validitas blok berdasarkan jumlah koin yang di-stake oleh node. Semakin banyak koin yang di-stake, semakin besar kemungkinan node untuk memilih blok berikutnya dalam blockchain.
- Delegated Proof of Stake (DPoS): Varian dari PoS di mana pemegang koin memilih sekelompok kecil saksi atau delegasi yang bertanggung jawab untuk memvalidasi blok dan menjaga keamanan jaringan.

### Peran Transaksi dan Konsensus

Transaksi dan konsensus adalah pilar utama dalam menjaga keamanan, integritas, dan keandalan jaringan blockchain. Transaksi memfasilitasi transfer nilai dan informasi di antara pengguna, sementara mekanisme konsensus memastikan bahwa setiap transaksi yang dimasukkan ke dalam blockchain diverifikasi dan diotorisasi oleh mayoritas node dalam jaringan.

Dengan menggunakan kombinasi transaksi yang terenkripsi dan mekanisme konsensus yang andal, blockchain menghadirkan solusi inovatif untuk keamanan dan validitas data dalam berbagai aplikasi, dari keuangan digital hingga manajemen rantai pasokan dan banyak lagi.

## 2.4 Smart Contracts

Smart contracts, atau kontrak pintar, adalah program komputer yang dirancang untuk mengeksekusi, menegosiasikan, atau menegaskan kesepakatan atau kontrak secara otomatis ketika kondisi tertentu terpenuhi. Konsep ini diperkenalkan oleh platform blockchain seperti Ethereum, yang memungkinkan pengguna untuk membuat aplikasi terdesentralisasi (dApps) yang menjalankan smart contracts sebagai bagian dari fungsionalitasnya.

### Fitur Utama Smart Contracts:

1. Automated Execution: Smart contracts beroperasi berdasarkan kode yang ditulis sebelumnya dan dieksekusi secara otomatis ketika kondisi yang ditetapkan terpenuhi. Ini menghilangkan kebutuhan akan pihak ketiga atau intermediasi dalam proses kontrak.

2. Decentralized: Smart contracts berjalan di atas jaringan blockchain yang terdesentralisasi. Setiap transaksi atau perubahan pada smart contract direkam dan diverifikasi di seluruh jaringan, meningkatkan transparansi dan keamanan.

3. Immutable: Setelah diterapkan, smart contract tidak dapat diubah atau dimanipulasi oleh pihak lain tanpa izin, karena sifat blockchain yang memungkinkan data yang sudah ada untuk tidak dapat diubah.

4. Conditional Logic: Smart contracts dapat mencakup logika kondisional yang rumit, yang memungkinkan untuk menetapkan syarat-syarat tertentu yang harus dipenuhi sebelum kontrak dieksekusi. Contoh termasuk pembayaran otomatis setelah pengiriman barang atau pengembalian dana jika kondisi tertentu tidak terpenuhi.

### Penerapan Smart Contracts:

- Keuangan: Penggunaan utama smart contracts adalah dalam sektor keuangan, termasuk pinjaman peer-to-peer, derivatif keuangan, dan penyelesaian otomatis transaksi.
- Manajemen Rantai Pasokan: Smart contracts dapat digunakan untuk memantau dan mengelola rantai pasokan, termasuk otomatisasi pembayaran dan pelacakan inventaris.
- Asuransi: Klaim asuransi bisa diproses otomatis sesuai dengan kondisi yang tercatat dalam smart contract, mengurangi biaya administrasi dan meningkatkan kecepatan penyelesaian.
- Properti Intelektual: Pengelolaan hak cipta dan lisensi bisa diotomatisasi dengan smart contracts, memastikan pemegang hak mendapat royalti secara tepat waktu.

Smart contracts menghadirkan potensi untuk mengubah cara berinteraksi dalam berbagai bidang, mengurangi biaya administrasi dan meningkatkan kecepatan serta keamanan proses bisnis. Namun, perlu diperhatikan juga bahwa pengembangan dan penerapan smart contracts memerlukan pemahaman mendalam tentang bahaya keamanan dan risiko kontrak yang mungkin terjadi.
