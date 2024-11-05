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

Dalam proses discovery peer, kita harus memastikan bahwa node yang bergabung adalah node yang sah. Salah satu cara untuk melakukannya adalah dengan menggunakan whitelist IP atau menggunakan metode handshake yang memverifikasi identitas peer sebelum menambahkannya ke dalam jaringan. 

Metode whitelist IP dapat dilakukan di server bootstrap peer yang sudah dibuat sebelumnya. Pada endpoint `REGISTER` tambahkan logik untuk memverifikasi berdasarkan white list IP.  Berikut contoh implementasinya:

```go
func (s *Server) registerPeer(peer string, conn net.Conn) {
    if !s.isValidPeer(peer) {
        fmt.Fprintln(conn, "Invalid peer address")
        return
    }

    success, msg := s.pm.RegisterPeer(peer)
    fmt.Fprintln(conn, msg)

    if success {
        s.broadcastPeers(peer, "register")
    } else {
        fmt.Println("Failed to register peer:", peer)
    }
}

func (s *Server) isValidPeer(peer string) bool {
    // Implementasi verifikasi, misalnya dengan memeriksa whitelist IP
    return true 
}
```

Pertanyaannya, darimana list IP didapatkan? Ini bisa dengan banyak cara, misalkan kita membuat sebuah aplikasi frontend yang berisi form pendaftaran, dimana proses registrasi akan memasukkan nama register, ktp, alamat, email, nomor hp, dan alamat IP server mereka. Verifikasi secara otomatis bisa dilakukan untuk meverifikasi alamat email dan no HP adalah valid. Bisa juga verifikasi liveness, verifikasi kesesuaian photo dengan ktp, dan lain-lain. Jika diperlukan, verifikasi bisa dilakukan secara manual oleh tim verifikasi.

Dalam koteks studi kasus sistem pemilu, verifikasi jauh lebih mudah, karena node-node yang ada dibuat oleh otoritas penyelenggara pemilu yang sah, sehingga whitelist IP menjadi proses yang sangat mudah. Untuk pihak publik seperti media massa, LSM dan pemantau pemilu, bisa melakukan proses registrasi terlebih dahulu untuk diverifikasi. 

5. Deteksi dan Penanganan Node Malicious

Mengimplementasikan mekanisme untuk mendeteksi node jahat sangat penting. Salah satu cara adalah dengan menggunakan reputasi peer. Setiap peer dapat memberikan penilaian terhadap peer lainnya berdasarkan keandalannya dalam memverifikasi transaksi dan blok.

Contoh implementasi sederhana deteksi node jahat:

```go
type Peer struct {
    Address string
    Reputation int
}

func (p *P2PNetwork) EvaluatePeer(peerAddress string, successful bool) {
    for i, peer := range p.peers {
        if peer.Address == peerAddress {
            if successful {
                p.peers[i].Reputation++
            } else {
                p.peers[i].Reputation--
            }

            // Jika reputasi terlalu rendah, hapus peer dari daftar
            if p.peers[i].Reputation < 0 {
                p.RemovePeer(peerAddress)
            }
            break
        }
    }
}
```

6. Tidak menyimpan data sensitif dalam blockchain

Data dalam blockchain disimpan secara distribusi di semua node, sehinmgga, siapaun yang menjadi node, dapat mengakses dan membaca data tersebut. Untuk menghindari penyerang menggunakan blockchain untuk mengakses informasi-informasi konfidensial, pastikan kita tidak menyimpan data sensitif apapun dalam jaringan blockchain. 

Jika aplikasi yang kita buat sangat membutuhkan data-data sensitif, pertimbangkan untuk menyimpan data secara hybrid, dimana data-data yang umum, bisa disimpan terdistribusi di blockchain, sementara data-data sensitif disimpan secara terpusat di database tertentu yang penyimpanan-nya tetap harus di-encrypt.

Dalam konteks aplikasi sistem pemilu yang menjadi studi kasus kita, data-data pemilih termasuk dalam kategori data sensitif. Misalnya NIK, Nama, Tanggal Lahir, dan sidik jari, bisa menjadi target para penyerang untuk mengumpulkan informasi, yang nantinya akan bisa digunakan dalam serangan phising. Data-data pemilih ini dibutuhkan untuk memverifikasi apakah pemilih yang melakukan vote valid atau tidak, sehingga kita akan menyimpannya dalam database terpisah (tersentralisasi).

7. Penerapan Kriptografi

Penerapan kriptografi diperlukan untuk mencegah penyerang mengakses data di tengah jalan (man in the middle). Pembahasan kriptografi lebih lanjut, akan kita bahas di sub bab selanjutnya, yaitu sub bab 10.1.3.

8. Penggunaan Tanda Tangan Digital

Penggunaan tanda tangan digital sangat bermanfaat dalam prose peer verification. Ini juga bisa dikombinasikan dengan whiotelist IP untuk mendapatkan peer yang jauh lebih terverifikasi. Pembahasan tanda tangan digital lebih lanjut dapat dilihat di sub bab 10.1.3 

9. Audit Keamanan

Audit keamanan harus terus menerus dilakukan untuk memastika jaringan blockchain yang kita buat aman. Terutama untuk memastikan smart contract yang dibuat tidak memiliki celah keamanan yang dapat dieksplotasi oleh penyerang. Melakukan audit rutin pada kode smart contracts dan infrastruktur blockchain penting dilakukan untuk mengidentifikasi dan memperbaiki potensi kerentanan.

10. Edukasi Pengguna

Untuk mencegah phising dan penipuan, tidak cukup hanya dengan sistem dan jaringan blockchain yang aman. Kita juga harus memberikan edukasi ke pengguna aplikasi blockchain. Bagaimana pun, praktek keamanan dimulai dari pengguna yang aware terhadap keamanan data individu. Penting untuk memberikan informasi dan pelatihan kepada pengguna tentang cara mengidentifikasi dan menghindari ancaman, seperti phishing dan penipuan.

### 10.1.3 Mekanisme Kriptografi dalam Keamanan Blockchain
Pentingnya keamanan di jaringan blockchain tidak dapat diremehkan, mengingat banyaknya aplikasi yang dibangun di atas teknologi ini, mulai dari cryptocurrency hingga sistem voting digital. Di bagian selanjutnya, kita akan mendalami mekanisme kriptografi yang lebih mendalam, serta bagaimana mereka mendukung keamanan transaksi dan integritas data dalam jaringan blockchain. Kriptografi adalah pilar utama dalam menjaga keamanan jaringan blockchain. Terdapat beberapa mekanisme kriptografi yang berperan penting, antara lain:

**Hashing:**

Hashing adalah proses yang mengubah data menjadi string karakter tetap dengan panjang tertentu. Fungsi hash yang umum digunakan dalam blockchain adalah SHA-256. Fungsi ini memiliki sifat deterministik, artinya data yang sama akan selalu menghasilkan hash yang sama. Hal ini memudahkan untuk memverifikasi integritas data, karena perubahan sekecil apa pun pada input akan menghasilkan hash yang sangat berbeda. Dengan cara ini, jika ada upaya untuk memodifikasi blok yang telah dicatat, hash dari blok tersebut akan berubah, dan jaringan dapat mendeteksi adanya penipuan.

Hashing sudha kita implmentasikan pada Bab 4. Contoh sederhanya-nya adalh seperti berikut:

```go
import (
	"crypto/sha256"
	"fmt"
)

func (b *Block) calculateHash() []byte {
	record := fmt.Sprintf("%d%d%s%s", b.Index, b.Timestamp, b.Data.Data, b.PrevHash)
	h := sha256.New()
	h.Write([]byte(record))
	hashed := h.Sum(nil)
	return hashed
}
```

**Enkripsi:**

Enkripsi digunakan untuk menjaga kerahasiaan data. Dalam konteks blockchain, data yang sensitif, seperti informasi identitas pengguna, dapat dienkripsi untuk melindunginya dari akses yang tidak sah. Enkripsi juga melindungi data agar tidak disadap oleh pihak ketiga di tengah perjalanan. Enkripsi kunci publik, seperti RSA, memungkinkan pengguna untuk berbagi informasi dengan aman tanpa perlu bertukar kunci rahasia sebelumnya. Hanya penerima yang memiliki kunci pribadi yang dapat mendekripsi pesan.

Mengimplementasikan enkripsi di aplikasi blockchain dapat meningkatkan keamanan data yang ditransmisikan antar node dan disimpan dalam blok. Misalnya, kita bisa mengenkripsi data sensitif seperti identitas pemilih atau informasi voting agar hanya node yang berwenang yang bisa membacanya. Dalam hal ini, kita bisa menggunakan enkripsi simetris (AES) atau enkripsi asimetris (RSA).

Berikut ini contoh kode implementasi enkripsi dan dekripsi menggunakan AES (Advanced Encryption Standard) di Golang, yang cocok untuk mengenkripsi data antar node atau data yang disimpan di blockchain.

1. Menyiapkan Fungsi Enkripsi dan Dekripsi AES

Kita mulai dengan membuat dua fungsi: satu untuk mengenkripsi dan satu lagi untuk mendekripsi. AES menggunakan kunci simetris, yang artinya satu kunci digunakan untuk kedua operasi.

```go
package encryption

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"encoding/base64"
	"fmt"
	"io"
)

// EncryptAES mengenkripsi teks dengan kunci AES.
func EncryptAES(plainText, key string) (string, error) {
	block, err := aes.NewCipher([]byte(key))
	if err != nil {
		return "", err
	}

	cipherText := make([]byte, aes.BlockSize+len(plainText))
	iv := cipherText[:aes.BlockSize]
	if _, err := io.ReadFull(rand.Reader, iv); err != nil {
		return "", err
	}

	stream := cipher.NewCFBEncrypter(block, iv)
	stream.XORKeyStream(cipherText[aes.BlockSize:], []byte(plainText))

	// Mengembalikan teks terenkripsi dalam format Base64 untuk memudahkan transfer.
	return base64.URLEncoding.EncodeToString(cipherText), nil
}

// DecryptAES mendekripsi teks terenkripsi dengan kunci AES.
func DecryptAES(cryptoText, key string) (string, error) {
	cipherText, err := base64.URLEncoding.DecodeString(cryptoText)
	if err != nil {
		return "", err
	}

	block, err := aes.NewCipher([]byte(key))
	if err != nil {
		return "", err
	}

	if len(cipherText) < aes.BlockSize {
		return "", fmt.Errorf("cipherText terlalu pendek")
	}

	iv := cipherText[:aes.BlockSize]
	cipherText = cipherText[aes.BlockSize:]

	stream := cipher.NewCFBDecrypter(block, iv)
	stream.XORKeyStream(cipherText, cipherText)

	return string(cipherText), nil
}
```

2. Menggunakan Fungsi Enkripsi pada Data Sensitif di Blockchain
   
Misalnya, dalam kasus pemilihan umum berbasis blockchain, kita bisa mengenkripsi data pemilih atau voting saat dikirim antar-node. Misalkan pada Fungsi Vote, di bagian pengiriman vote, kita bisa mengenkripsi data pemilih sebelum mengirimkannya ke node lain.

```go
package peer

import (
	"encryption"
	"fmt"
	"net"
)

// SendEncryptedVote mengenkripsi dan mengirim vote ke peer lain.
func (p2p *P2PNetwork) SendEncryptedVote(voterID, candidateID, encryptionKey string) {
	plainVote := voterID + ":" + candidateID

	encryptedVote, err := encryption.EncryptAES(plainVote, encryptionKey)
	if err != nil {
		fmt.Println("Error encrypting vote:", err)
		return
	}

	// Simpan atau kirim encryptedVote ke node lainnya
	for _, peer := range p2p.peers {
		conn, err := net.Dial("tcp", peer.Address)
		if err != nil {
			fmt.Println("Error connecting to peer:", err)
			continue
		}
		defer conn.Close()

		fmt.Fprintf(conn, "vote:%s\n", encryptedVote)
	}
}
```

**Dekripsi Vote di Node Penerima**

Di sisi penerima, kita bisa mendekripsi data saat menerima vote dari peer lain.

```go
func (p2p *P2PNetwork) ReceiveEncryptedVote(encryptedVote, encryptionKey string) {
	plainVote, err := encryption.DecryptAES(encryptedVote, encryptionKey)
	if err != nil {
		fmt.Println("Error decrypting vote:", err)
		return
	}

	// Parsing vote menjadi voterID dan candidateID setelah dekripsi
	voteData := strings.Split(plainVote, ":")
	if len(voteData) != 2 {
		fmt.Println("Invalid vote data format")
		return
	}
	voterID, candidateID := voteData[0], voteData[1]

	fmt.Printf("Received vote from %s for candidate %s\n", voterID, candidateID)
}
```

3. Mengonfigurasi Kunci Enkripsi

AES membutuhkan kunci dengan panjang spesifik, biasanya 16, 24, atau 32 byte. Kunci ini bisa disimpan dalam variabel lingkungan atau file konfigurasi agar tetap aman.

```go
// Contoh Kunci AES dengan panjang 32 byte (256 bit)
const encryptionKey = "thisis32bitlongpassphraseimusing!"
```

Pastikan kunci ini dikelola dengan baik, karena siapapun yang memiliki akses ke kunci bisa mengenkripsi dan mendekripsi pesan.


**Tanda Tangan Digital:**

Tanda tangan digital merupakan metode yang memastikan otentikasi dan integritas pesan. Dalam blockchain, setiap transaksi dapat ditandatangani secara digital menggunakan kunci pribadi pengirim. Tanda tangan ini kemudian dapat diverifikasi oleh penerima menggunakan kunci publik pengirim. Dengan cara ini, penerima dapat yakin bahwa transaksi tersebut benar-benar dilakukan oleh pengirim yang sah dan tidak telah dimodifikasi.

**Mekanisme Konsensus:**

Selain kriptografi, mekanisme konsensus memainkan peran penting dalam keamanan blockchain. Seperti yang telah disebutkan, Proof of Work (PoW) adalah salah satu metode yang paling umum digunakan untuk mencegah serangan. Dengan mengharuskan peserta jaringan (penambang) untuk memecahkan teka-teki matematis yang kompleks sebelum menambahkan blok baru ke dalam rantai, PoW menciptakan penghalang yang signifikan bagi penyerang. Namun, PoW juga memiliki kelemahan, seperti konsumsi energi yang tinggi, yang memicu pengembangan alternatif seperti Proof of Stake (PoS), di mana peserta harus mengunci sejumlah cryptocurrency sebagai jaminan untuk dapat berpartisipasi dalam proses verifikasi. Konsensus POW sudah kita implmentasikan pada Bab 6.


## 10.1.4 Kesimpulan

Penting untuk memastikan bahwa protokol blockchain dirancang dengan mempertimbangkan keamanan dari awal, termasuk mekanisme konsensus yang efisien dan perlindungan terhadap serangan. Keamanan dalam jaringan blockchain adalah topik yang kompleks dan dinamis, yang terus berkembang seiring dengan evolusi teknologi dan munculnya ancaman baru. Melalui pemahaman mendalam tentang mekanisme kriptografi, ancaman yang ada, dan penerapan praktik terbaik, kita dapat memastikan bahwa sistem blockchain tetap aman, dapat diandalkan, dan siap untuk digunakan dalam berbagai aplikasi di masa depan. Kepercayaan dalam teknologi ini sangat tergantung pada upaya kolektif untuk menjaga keamanan dan integritasnya.

## 10.2 Skalabilitas
