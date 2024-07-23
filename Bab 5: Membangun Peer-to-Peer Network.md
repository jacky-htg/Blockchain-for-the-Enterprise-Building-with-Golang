# Bab 5: Membangun Peer-to-Peer Network

Dalam bab ini, kita akan membangun jaringan peer-to-peer (P2P) untuk blockchain kita. Jaringan P2P memungkinkan setiap node dalam jaringan untuk berkomunikasi langsung satu sama lain tanpa perlu otoritas pusat. Kita akan mengimplementasikan dasar-dasar jaringan P2P, termasuk menambahkan peer, berkomunikasi dengan peer, dan menangani koneksi masuk.

## 5.1 Dasar-dasar Jaringan Peer-to-Peer

Jaringan Peer-to-Peer (P2P) adalah jaringan di mana setiap node berfungsi sebagai klien dan server sekaligus, memungkinkan pertukaran data langsung antara node tanpa perantara. Dalam konteks blockchain, jaringan P2P memungkinkan distribusi dan sinkronisasi blok di antara semua node dalam jaringan.

## 5.2 Implementasi Jaringan P2P di Golang

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

## 5.3 Menghubungkan dan Berkomunikasi dengan Peers

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

## 5.4 Menangani Koneksi dan Pemutusan Koneksi

Kode yang digunakan untuk menangani koneksi dan pemutusan koneksi dapat ditemukan dalam fungsi handleConnection di app/peer/peer.go. Fungsi ini digunakan untuk menerima blok dari peer lain dan memperbarui blockchain lokal dengan blok yang diterima.

Dengan memahami implementasi di atas, Anda dapat membangun jaringan peer-to-peer yang sederhana untuk mendukung blockchain menggunakan Golang. Hal ini memungkinkan node dalam jaringan untuk berkomunikasi langsung satu sama lain tanpa memerlukan server pusat, menjadikannya lebih terdesentralisasi dan aman.
