# Bab 5: Membangun Peer-to-Peer Network

Jaringan peer-to-peer (P2P) adalah jaringan yang terdiri dari beberapa node atau komputer yang terhubung secara langsung satu sama lain tanpa memerlukan server pusat. Dalam konteks blockchain, jaringan P2P digunakan untuk mendistribusikan blok transaksi di antara node-node dalam jaringan.

## 5.1 Dasar-dasar Jaringan Peer-to-Peer

Di Golang, kita dapat membangun jaringan P2P untuk blockchain dengan menggunakan paket net untuk mengatur koneksi TCP dan encoding/json untuk mengirim dan menerima data dalam format JSON.

File app/peer/peer.go berisi implementasi untuk Peer dan P2PNetwork:

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

## 5.2 Implementasi Jaringan P2P di Golang

Implementasi di atas menggunakan Golang untuk:

- AddPeer: Menambahkan peer baru ke dalam jaringan P2P.
- BroadcastBlock: Mengirimkan blok transaksi ke semua peer dalam jaringan.
- ListenForBlocks: Mendengarkan koneksi masuk dari peer lain untuk menerima blok transaksi.
- handleConnection: Menangani koneksi dari peer lain, mendecode blok yang diterima, dan menambahkannya ke blockchain lokal.

## 5.3 Menghubungkan dan Berkomunikasi dengan Peers

Untuk menghubungkan node dalam jaringan, Anda perlu menjalankan node dengan port yang berbeda untuk mendengarkan koneksi dan mengirim blok. Contoh penggunaan:

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
	peerAddress := flag.String("peer", "", "Address of a peer to connect to")
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

	// Membroadcast blok genesis ke semua peer
	genesisBlock := blockchain.Block{Index: 0, Timestamp: 0, Data: blockchain.Data{Data: "Genesis Block"}}
	p2p.BroadcastBlock(genesisBlock)

	// Membaca input dari pengguna untuk menambahkan blok baru dan membroadcast-nya
	go readInput(p2p)

	// Menunggu agar program tetap berjalan
	select {}
}

```

Dalam contoh di atas, kita membuat jaringan P2P baru, menambahkan beberapa peer, mendengarkan koneksi untuk menerima blok, dan mengirim blok genesis ke semua peer dalam jaringan. Fungsi readInput digunakan untuk membaca input dari pengguna (stdin). Setiap kali ada input baru, sebuah blok baru dibuat dengan data tersebut, kemudian blok tersebut ditambahkan ke blockchain lokal dan dibroadcast ke semua peer dalam jaringan. Fungsi readInput dijalankan dalam goroutine terpisah untuk terus-menerus membaca input dari pengguna.

## 5.4 Menangani Koneksi dan Pemutusan Koneksi

Kode yang digunakan untuk menangani koneksi dan pemutusan koneksi dapat ditemukan dalam fungsi handleConnection di app/peer/peer.go. Fungsi ini digunakan untuk menerima blok dari peer lain dan memperbarui blockchain lokal dengan blok yang diterima.

Dengan memahami implementasi di atas, Anda dapat membangun jaringan peer-to-peer yang sederhana untuk mendukung blockchain menggunakan Golang. Hal ini memungkinkan node dalam jaringan untuk berkomunikasi langsung satu sama lain tanpa memerlukan server pusat, menjadikannya lebih terdesentralisasi dan aman.
