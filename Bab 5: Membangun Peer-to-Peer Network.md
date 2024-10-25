# Bab 5: Membangun Peer-to-Peer Network

Dalam bab ini, kita akan membangun jaringan peer-to-peer (P2P) untuk blockchain kita. Jaringan P2P memungkinkan setiap node dalam jaringan untuk berkomunikasi langsung satu sama lain tanpa perlu otoritas pusat. Kita akan mengimplementasikan dasar-dasar jaringan P2P, termasuk menambahkan peer, berkomunikasi dengan peer, dan menangani koneksi masuk.

Namun sebelum masuk ke pembahasan P2P Network, kita akan menyisipkan pembahasan tentang TCP dan socket programming. Hal ini penting, mengingat implmentasi P2P Network yang dibangun dalam buku ini, menggunakan protocol TCP dengan konsep socket programming.

## 5.1 TCP dan Socket Programming

TCP (Transmission Control Protocol) dan socket programming adalah dua konsep penting dalam komunikasi jaringan. Keduanya bekerja sama untuk memungkinkan aplikasi berinteraksi dan bertukar data melalui jaringan. Berikut adalah penjelasan lebih lanjut mengenai keduanya serta hubungan di antara mereka.

### 5.1.1 TCP (Transmission Control Protocol)
1. Pengertian TCP
TCP adalah salah satu protokol transport layer dalam suite protokol Internet (TCP/IP). Protokol ini berfungsi untuk menyediakan komunikasi yang andal, berorientasi koneksi, antara aplikasi yang berjalan pada host yang berbeda dalam jaringan.

2. Karakteristik Utama TCP
- Connection-Oriented: TCP membangun koneksi antara pengirim dan penerima sebelum data dikirim, memastikan komunikasi yang terjamin.
- Pengiriman Data yang Andal: TCP menjamin bahwa semua data yang dikirim akan sampai ke tujuan dengan cara mengirim ulang paket yang hilang atau rusak.
- Urutan Pengiriman: Data yang dikirim akan tiba di penerima dalam urutan yang sama dengan yang dikirim, berkat mekanisme pengurutan.
- Kontrol Aliran: TCP mencegah pengirim mengirim terlalu banyak data dalam satu waktu, menggunakan mekanisme pengendalian yang disebut "windowing".
- Kontrol Kemacetan: TCP dapat mendeteksi kemacetan dalam jaringan dan menyesuaikan laju pengiriman data untuk mengurangi beban pada jaringan.
  
3. Proses Koneksi TCP
Koneksi TCP dibangun melalui proses yang dikenal sebagai Three-Way Handshake:

- SYN: Klien mengirimkan paket SYN untuk memulai koneksi.
- SYN-ACK: Server merespons dengan paket SYN-ACK, menandakan bahwa koneksi dapat diterima.
- ACK: Klien mengirimkan paket ACK untuk mengonfirmasi bahwa koneksi telah dibangun.
  
4. Penutupan Koneksi TCP
Koneksi TCP ditutup dengan proses Four-Way Handshake yang melibatkan pengiriman paket FIN dan ACK untuk menandakan bahwa koneksi dapat dihentikan.

### 5.1.2 Socket Programming
1. Pengertian Socket
Socket adalah endpoint untuk mengirim dan menerima data melalui jaringan. Dalam konteks socket programming, socket adalah alat yang digunakan oleh aplikasi untuk berkomunikasi dengan aplikasi lain melalui TCP/IP.

2. Tipe Socket
- Stream Sockets: Menggunakan TCP, memberikan komunikasi yang berorientasi koneksi dan andal.
- Datagram Sockets: Menggunakan UDP, yang tidak berorientasi koneksi dan tidak menjamin keandalan pengiriman data.
  
3. Komponen Socket Programming
- Server: Program yang menyediakan layanan kepada client. Server mendengarkan pada port tertentu dan menerima koneksi dari client.
- Client: Program yang menghubungkan diri ke server untuk meminta layanan. Client mengirimkan data ke server dan menerima data dari server.
  
4. Proses Komunikasi Socket
Proses komunikasi antara server dan client menggunakan socket melibatkan langkah-langkah berikut:

- Membuat Socket: Membuat socket di server dan client menggunakan fungsi atau metode yang sesuai.
- Mengikat Socket (Binding): Server mengikat socket ke alamat IP dan port tertentu.
- Mendengarkan (Listening): Server mendengarkan koneksi yang masuk.
- Menerima Koneksi (Accepting): Server menerima koneksi dari client.
- Mengirim dan Menerima Data: Setelah koneksi terjalin, data dapat dikirim dan diterima.
- Menutup Koneksi: Setelah komunikasi selesai, socket ditutup untuk membebaskan sumber daya.
  
5. Bahasa Pemrograman
Socket programming dapat dilakukan dalam berbagai bahasa pemrograman seperti Python, Java, C, C++, dan Go. Setiap bahasa memiliki library atau modul yang berbeda untuk bekerja dengan socket.

### 5.1.3 Hubungan Antara TCP dan Socket Programming
- Protokol yang Digunakan: Socket programming memungkinkan aplikasi untuk menggunakan protokol komunikasi, seperti TCP. Dengan menggunakan TCP sebagai protokol transport, socket programming menyediakan komunikasi yang andal dan teratur.
- Endpoint untuk Komunikasi: Socket berfungsi sebagai endpoint komunikasi yang dibangun di atas protokol TCP, memungkinkan aplikasi untuk saling bertukar data dengan cara yang efisien.
- Implementasi TCP: Dalam socket programming, pengembang menggunakan fungsi atau metode yang berkaitan dengan TCP untuk membangun koneksi, mengirim, dan menerima data. Misalnya, saat membuat stream socket di Go, TCP secara otomatis digunakan sebagai protokol transport.

### 5.1.4 Contoh Socket Programming Menggunakan TCP
Berikut adalah contoh sederhana socket programming menggunakan TCP dalam bahasa Go:

**Server**
```go
package main

import (
	"fmt"
	"net"
)

func main() {
	// Membuat listener pada port 8080
	listener, err := net.Listen("tcp", ":8080")
	if err != nil {
		fmt.Println("Error listening:", err.Error())
		return
	}
	defer listener.Close()

	fmt.Println("Menunggu koneksi...")

	// Menerima koneksi
	for {
		conn, err := listener.Accept()
		if err != nil {
			fmt.Println("Error accepting:", err.Error())
			continue
		}

		fmt.Println("Koneksi dari:", conn.RemoteAddr())

		// Mengirim pesan ke client
		message := []byte("Halo, Client!\n")
		_, err = conn.Write(message)
		if err != nil {
			fmt.Println("Error sending:", err.Error())
		}

		// Menutup koneksi
		conn.Close()
	}
}
```

**Client**
```go
package main

import (
	"fmt"
	"net"
	"os"
)

func main() {
	// Menghubungkan ke server
	conn, err := net.Dial("tcp", "localhost:8080")
	if err != nil {
		fmt.Println("Error connecting:", err.Error())
		os.Exit(1)
	}
	defer conn.Close()

	// Menerima data dari server
	buffer := make([]byte, 1024)
	n, err := conn.Read(buffer)
	if err != nil {
		fmt.Println("Error reading:", err.Error())
		return
	}

	// Menampilkan pesan dari server
	fmt.Print(string(buffer[:n]))
}
```

TCP dan socket programming adalah dua aspek kunci dalam komunikasi jaringan. TCP menyediakan protokol yang andal untuk pengiriman data, sementara socket programming memungkinkan pengembang untuk memanfaatkan TCP dalam aplikasi mereka, menciptakan interaksi yang efisien dan terstruktur di antara berbagai aplikasi dalam jaringan.

## 5.2 Dasar-dasar Jaringan Peer-to-Peer

Jaringan Peer-to-Peer (P2P) adalah jaringan di mana setiap node berfungsi sebagai klien dan server sekaligus, memungkinkan pertukaran data langsung antara node tanpa perantara. Dalam konteks blockchain, jaringan P2P memungkinkan distribusi dan sinkronisasi blok di antara semua node dalam jaringan.

## 5.3 Implementasi Jaringan P2P di Golang

Berikut adalah implementasi dasar untuk jaringan P2P menggunakan Golang. Struktur folder kita akan seperti ini:

```console
app
-- blockchain
---- data.go
---- block.go
---- blockchain.go
-- peer
---- peer.go
main.go
go.mod
```

File app/peer/peer.go berisi:

```go
package peer

import (
	"encoding/json"
	"fmt"
	"myapp/app/blockchain"
	"net"
)

type Peer struct {
	address string
}

type P2PNetwork struct {
	peers      []Peer
	Blockchain *blockchain.Blockchain
}

func NewP2PNetwork() *P2PNetwork {
	return &P2PNetwork{
		peers:      make([]Peer, 0),
		Blockchain: &blockchain.Blockchain{},
	}
}

func (p2p *P2PNetwork) AddPeer(address string) {
	peer := Peer{address: address}
	p2p.peers = append(p2p.peers, peer)
}

func (p2p *P2PNetwork) BroadcastBlock(block blockchain.Block) {
	for _, peer := range p2p.peers {
		conn, err := net.Dial("tcp", peer.address)
		if err != nil {
			fmt.Println("Error connecting to peer:", err)
			continue
		}
		defer conn.Close()

		encoder := json.NewEncoder(conn)
		if err := encoder.Encode(block); err != nil {
			fmt.Println("Error encoding block:", err)
		}
	}
}

func (p2p *P2PNetwork) ListenForBlocks(port string) {
	ln, err := net.Listen("tcp", ":"+port)
	if err != nil {
		fmt.Println("Error setting up listener:", err)
		return
	}
	defer ln.Close()

	for {
		conn, err := ln.Accept()
		if err != nil {
			fmt.Println("Error accepting connection:", err)
			continue
		}
		go p2p.handleConnection(conn)
	}
}

func (p2p *P2PNetwork) handleConnection(conn net.Conn) {
	defer conn.Close()

	decoder := json.NewDecoder(conn)
	var block blockchain.Block
	if err := decoder.Decode(&block); err != nil {
		fmt.Println("Error decoding block:", err)
		return
	}

	p2p.Blockchain.AddBlock(block.Data.Data)
	fmt.Println("Received new block:", block)
}
```

Implementasi di atas menggunakan Golang untuk:

- AddPeer: Menambahkan peer baru ke dalam jaringan P2P.
- BroadcastBlock: Mengirimkan blok transaksi ke semua peer dalam jaringan.
- ListenForBlocks: Mendengarkan koneksi masuk dari peer lain untuk menerima blok transaksi.
- handleConnection: Menangani koneksi dari peer lain, mendecode blok yang diterima, dan menambahkannya ke blockchain lokal.

## 5.4 Menghubungkan dan Berkomunikasi dengan Peers

Kita akan memperbarui fungsi main untuk menambahkan kemampuan membuat genesis block, menambahkan transaksi baru, dan mengetes validasi blockchain.

File main.go:

```go
package main

import (
	"bufio"
	"flag"
	"fmt"
	"os"
	"myapp/app/blockchain"
	"myapp/app/peer"
)

func readInput(p2p *peer.P2PNetwork) {
	scanner := bufio.NewScanner(os.Stdin)
	for scanner.Scan() {
		data := scanner.Text()
		block := p2p.Blockchain.AddBlock(data)
		p2p.BroadcastBlock(block)
		fmt.Println("Added new block:", block)
	}
}

func main() {
	port := flag.String("port", "3000", "Port to listen on")
	flag.Parse()

	// Membuat jaringan P2P baru
	p2p := peer.NewP2PNetwork()

	// Menambahkan peer secara manual
	if *port == "3000" {
		p2p.AddPeer("localhost:3001")
		p2p.AddPeer("localhost:3002")
	} else if *port == "3001" {
		p2p.AddPeer("localhost:3000")
		p2p.AddPeer("localhost:3002")
	} else if *port == "3002" {
		p2p.AddPeer("localhost:3000")
		p2p.AddPeer("localhost:3001")
	}
	
	// Mendengarkan koneksi untuk menerima blok
	go p2p.ListenForBlocks(*port)

	if *port == "3000" {
		// Membroadcast blok genesis ke semua peer
		genesisBlock := blockchain.Block{Index: 0, Timestamp: 0, Data: blockchain.Data{Data: "Genesis Block"}}
		p2p.BroadcastBlock(genesisBlock)
	}

	// Membaca input dari pengguna untuk menambahkan blok baru dan membroadcast-nya
	go readInput(p2p)

	// Menunggu agar program tetap berjalan
	select {}
}
```

Dalam contoh di atas, kita membuat jaringan P2P baru, menambahkan beberapa peer, mendengarkan koneksi untuk menerima blok, dan mengirim blok genesis ke semua peer dalam jaringan. Fungsi readInput digunakan untuk membaca input dari pengguna (stdin). Setiap kali ada input baru, sebuah blok baru dibuat dengan data tersebut, kemudian blok tersebut ditambahkan ke blockchain lokal dan dibroadcast ke semua peer dalam jaringan. Fungsi readInput dijalankan dalam goroutine terpisah untuk terus-menerus membaca input dari pengguna.

### Cara Menjalankan Peer
Jika diperhatikan, kode di atas menambahkan peer secara manual, yaitu port 3000, 3001 dan 3002. Untuk itu, kita akan menjalankan peer dengan skenario ada 3 peer masing-masing dengan address localhost:3000, localhost:3001, dan localhost:3002. Khusus jika kita menjalankan port 3000 maka akan tercipta genesis block.

Untuk menjalankan aplikasi ini, Anda perlu menjalankan beberapa instance dari program pada port yang berbeda. Berikut adalah langkah-langkahnya:

Buka terminal dan jalankan perintah untuk memulai node pertama pada port 3001:
```console
go run main.go -port=3001
```

Buka terminal kedua dan jalankan perintah untuk memulai node kedua pada port 3002:

```console
go run main.go -port=3002
```

Buka terminal ketiga dan jalankan perintah untuk memulai node ketiga pada port 3000:

```console
go run main.go -port=3000
```

Setiap node akan terhubung satu sama lain berdasarkan konfigurasi port yang telah ditentukan. Anda bisa mengetikkan data baru di setiap terminal, yang akan ditambahkan sebagai blok baru dalam blockchain dan dibroadcast ke semua node dalam jaringan.

## 5.5 Menangani Koneksi dan Pemutusan Koneksi

Kode yang digunakan untuk menangani koneksi dan pemutusan koneksi dapat ditemukan dalam fungsi handleConnection di app/peer/peer.go. Fungsi ini digunakan untuk menerima blok dari peer lain dan memperbarui blockchain lokal dengan blok yang diterima.

Dengan memahami implementasi di atas, Anda dapat membangun jaringan peer-to-peer yang sederhana untuk mendukung blockchain menggunakan Golang. Hal ini memungkinkan node dalam jaringan untuk berkomunikasi langsung satu sama lain tanpa memerlukan server pusat, menjadikannya lebih terdesentralisasi dan aman.

## 5.6 Clean Arsitektur
Untuk mempermudah pengelolaan kode, biasanay kode akan dipecah menjadi beberapa file sesuai dengan tujuan dari fungsi atau kode tersebut. Dalam project ini, saya membagi menjadi 4 file, yaitu: 

1. peer.go
File ini berisi definisi struktur data Peer dan P2PNetwork, serta metode yang berhubungan dengan pengelolaan peer dalam jaringan. Ini mencakup:

- Struktur Peer: Mendefinisikan atribut yang dimiliki oleh peer, dalam hal ini hanya alamat (address).
- Struktur P2PNetwork: Menyimpan daftar peer (peers) dan referensi ke objek Blockchain dan menyediakan metode untuk menambah peer dan mendengarkan koneksi.
  
```go
package peer

import (
	"myapp/app/blockchain"
)

type Peer struct {
	address string
}

type P2PNetwork struct {
	peers      []Peer
	Blockchain *blockchain.Blockchain
}

func NewP2PNetwork() *P2PNetwork {
	return &P2PNetwork{
		peers:      make([]Peer, 0),
		Blockchain: &blockchain.Blockchain{},
	}
}

func (p2p *P2PNetwork) AddPeer(address string) {
	peer := Peer{address: address}
	p2p.peers = append(p2p.peers, peer)
}
```

2. network.go
File ini berisi metode untuk menangani komunikasi jaringan dan mendengarkan koneksi dari peer lain. Ini termasuk:

Metode ListenForBlocks:
- Mengatur listener TCP untuk menerima koneksi dari peer lain.
- Menerima dan memproses koneksi menggunakan goroutine untuk menangani banyak koneksi secara bersamaan.

```go
package peer

import (
	"fmt"
	"net"
)

func (p2p *P2PNetwork) ListenForBlocks(port string) {
	ln, err := net.Listen("tcp", ":"+port)
	if err != nil {
		fmt.Println("Error setting up listener:", err)
		return
	}
	defer ln.Close()

	for {
		conn, err := ln.Accept()
		if err != nil {
			fmt.Println("Error accepting connection:", err)
			continue
		}
		go p2p.handleConnection(conn)
	}
}
```
3. handler.go
Jika ingin memiliki pemisahan yang lebih jelas antara pengolahan koneksi dan pengelolaan peer, kita dapat membuat file ini untuk mengelola semua handler yang berkaitan dengan permintaan dari peer lain. Ini termasuk:

Metode handleConnection:
- Mengelola koneksi yang diterima, termasuk decoding blok yang diterima dan menambahkannya ke blockchain.

```go
package peer

import (
	"encoding/json"
	"fmt"
	"net"
)

func (p2p *P2PNetwork) handleConnection(conn net.Conn) {
	defer conn.Close()

	decoder := json.NewDecoder(conn)
	var block blockchain.Block
	if err := decoder.Decode(&block); err != nil {
		fmt.Println("Error decoding block:", err)
		return
	}

	p2p.Blockchain.AddBlock(block.Data.Data)
	fmt.Println("Received new block:", block)
}
```

4. broadcast.go
File ini fokus pada metode untuk menyebarkan informasi ke semua peer dalam jaringan. Ini bisa mencakup:

Metode BroadcastBlock:
- Mengirim blok yang baru ditambahkan ke semua peer yang terdaftar.
- Menangani koneksi dan pengkodean blok ke format JSON untuk dikirim.

```go
package peer

import (
	"encoding/json"
	"fmt"
	"net"
	"myapp/app/blockchain"
)

func (p2p *P2PNetwork) BroadcastBlock(block blockchain.Block) {
	for _, peer := range p2p.peers {
		conn, err := net.Dial("tcp", peer.address)
		if err != nil {
			fmt.Println("Error connecting to peer:", err)
			continue
		}
		defer conn.Close()

		encoder := json.NewEncoder(conn)
		if err := encoder.Encode(block); err != nil {
			fmt.Println("Error encoding block:", err)
		}
	}
}
```

Dengan memecah file menjadi beberapa bagian seperti ini, akan meningkatkan modularitas dan menjaga tanggung jawab yang jelas untuk setiap bagian dari aplikasi. Ini memudahkan pemeliharaan dan pengembangan di masa depan. Setiap file sekarang memiliki fokus yang lebih spesifik, membuatnya lebih mudah untuk di-debug dan ditingkatkan sesuai kebutuhan.
