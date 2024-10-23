# Bab 8: Bootstrap Peer dan Pengelolaan Peer

Dalam jaringan blockchain yang terdistribusi, manajemen peer dan bootstrap peer memainkan peran penting dalam memastikan keandalan dan integritas sistem. Bootstrap peer adalah titik awal bagi node baru yang ingin bergabung dengan jaringan, menyediakan informasi yang diperlukan untuk menemukan peer lainnya. Dengan adanya bootstrap peer, proses penemuan dan koneksi antar node dapat dilakukan dengan efisien, memungkinkan jaringan untuk berkembang dan berfungsi dengan baik. Namun, tantangan keamanan juga muncul, terutama dalam hal kepercayaan terhadap bootstrap peer dan perlindungan terhadap node yang mungkin berusaha merusak jaringan.

Selain itu, pengelolaan peer yang efektif adalah kunci untuk menjaga kesehatan dan kinerja jaringan. Hal ini melibatkan pemantauan status node, menangani koneksi yang putus, dan memastikan bahwa node yang berfungsi dengan baik tetap terhubung. Dalam bab ini, kita akan membahas peran bootstrap peer, implementasinya dalam bahasa Go, serta teknik-teknik untuk penemuan peer dan pengelolaan keamanan dalam jaringan peer-to-peer. Dengan memahami aspek-aspek ini, kita dapat membangun jaringan blockchain yang lebih kuat dan responsif terhadap tantangan yang ada.

## 8.1 Peran Bootstrap Peer

**Fungsi dan Pentingnya Bootstrap Peer dalam Jaringan Blockchain**

Bootstrap peer memiliki fungsi krusial dalam jaringan blockchain dengan menyediakan titik awal bagi node baru untuk bergabung. Ketika sebuah node ingin terhubung ke jaringan blockchain, ia memerlukan informasi tentang node lain yang sudah ada. Bootstrap peer berfungsi sebagai penghubung yang memberikan daftar peer yang dapat dijangkau, memungkinkan node baru untuk mulai berkomunikasi dan mengunduh salinan blockchain. Tanpa bootstrap peer, proses penemuan dan koneksi antar node akan menjadi sulit dan memakan waktu, yang dapat menghambat pertumbuhan jaringan.

**Mengapa Diperlukan Node Bootstrap untuk Node Baru?**

Node bootstrap diperlukan untuk memfasilitasi integrasi node baru ke dalam jaringan. Ketika sebuah node baru terhubung, ia tidak memiliki informasi tentang node lain atau salinan blockchain. Dengan bantuan node bootstrap, node baru dapat dengan cepat menemukan peer yang dapat diajak berkomunikasi, sehingga mempercepat proses sinkronisasi dengan jaringan. Selain itu, node bootstrap membantu dalam meminimalkan beban jaringan dengan mengurangi jumlah permintaan yang perlu dilakukan oleh node baru, yang bisa berpotensi membanjiri jaringan dengan permintaan.

**Studi Kasus: Apa yang Terjadi Jika Bootstrap Peer Gagal (Fault-Tolerance)?**

Jika bootstrap peer gagal atau tidak dapat diakses, node baru yang ingin bergabung dengan jaringan akan mengalami kesulitan untuk menemukan peer lainnya. Hal ini dapat mengakibatkan kegagalan dalam proses sinkronisasi blockchain dan memisahkan node baru dari jaringan. Dalam situasi ini, penting untuk memiliki mekanisme fault-tolerance, seperti cadangan bootstrap peer atau sistem fallback yang memungkinkan node baru untuk mencoba terhubung ke peer lain yang diketahui. Tanpa langkah-langkah ini, ketahanan jaringan dapat terancam, dan node baru dapat terjebak dalam keadaan terisolasi.

**Alternatif: Daftar Statis vs. Dinamis Bootstrap Peer**

Terdapat dua pendekatan dalam pengelolaan bootstrap peer: daftar statis dan dinamis. Daftar statis terdiri dari alamat peer yang telah ditentukan sebelumnya, yang dihardcode dalam kode. Pendekatan ini sederhana tetapi kurang fleksibel, karena tidak dapat menyesuaikan dengan perubahan dalam jaringan. Di sisi lain, daftar dinamis memungkinkan node untuk memperbarui daftar bootstrap peer secara real-time berdasarkan informasi yang diterima dari peer yang sudah terhubung. Pendekatan ini memberikan fleksibilitas lebih besar, namun memerlukan implementasi yang lebih kompleks untuk mengelola dan memperbarui daftar peer. Memilih antara kedua pendekatan ini tergantung pada kebutuhan spesifik jaringan dan tujuan pengelolaan peer.

## 8.2 Implementasi Bootstrap Peer di Golang
Kita akan membuatv server bootstrap peer sederhana yang menyimpan daftar peer secara permanen ke dalam file dalam format JSON. Pertama, buat project/folder dengan `bootstrap-peer`, dengan struktur folder seperti berikut:

```console
├── bootstrap
│   └── bootstrap.go
├── main.go
└── peers.json
```
Jalankan `go mod init bootstrap-peer` untuk membuat project/module. Kemudian buat file bootstrap/bootstrap.go yang berisi:

```go
package bootstrap

import (
	"fmt"
	"net"
	"os"

	"github.com/bytedance/sonic"
)

// Peer struct
type Peer struct {
	Address string `json:"address"`
}

// BootstrapServer struct
type BootstrapServer struct {
	Peers []Peer
	File  string
}

// NewBootstrapServer initializes the BootstrapServer
func NewBootstrapServer(file string) *BootstrapServer {
	return &BootstrapServer{
		File: file,
	}
}

// LoadPeers loads peers from a JSON file
func (bs *BootstrapServer) LoadPeers() error {
	data, err := os.ReadFile(bs.File)
	if err != nil {
		return err
	}

	return sonic.Unmarshal(data, &bs.Peers)
}

// SavePeers saves the current peers to a JSON file
func (bs *BootstrapServer) SavePeers() error {
	data, err := sonic.Marshal(bs.Peers)
	if err != nil {
		return err
	}

	return os.WriteFile(bs.File, data, 0644)
}

// Listen starts the bootstrap server
func (bs *BootstrapServer) Listen(port string) {
	ln, err := net.Listen("tcp", ":"+port)
	if err != nil {
		fmt.Println("Error setting up listener:", err)
		return
	}
	defer ln.Close()

	fmt.Printf("Bootstrap server listening on port %s\n", port)

	for {
		conn, err := ln.Accept()
		if err != nil {
			fmt.Println("Error accepting connection:", err)
			continue
		}
		go bs.handleConnection(conn)
	}
}

// handleConnection menangani koneksi dari peer baru dan menambahkannya ke daftar peers.
func (bs *BootstrapServer) handleConnection(conn net.Conn) {
	defer conn.Close()

	// Baca data dari koneksi
	var peer Peer
	buf := make([]byte, 1024) // Buffer untuk membaca data
	n, err := conn.Read(buf)
	if err != nil {
		fmt.Println("Error reading from connection:", err)
		return
	}

	// Deserialisasi data JSON menggunakan Sonic
	if err := sonic.Unmarshal(buf[:n], &peer); err != nil {
		fmt.Println("Error decoding peer:", err)
		return
	}

	// Cek apakah peer sudah ada
	for _, p := range bs.Peers {
		if p.Address == peer.Address {
			fmt.Println("Peer already exists:", peer.Address)
			return
		}
	}

	// Tambahkan peer baru ke daftar
	bs.Peers = append(bs.Peers, peer)
	fmt.Println("New peer added:", peer.Address)

	// Simpan daftar peers yang diperbarui
	if err := bs.SavePeers(); err != nil {
		fmt.Println("Error saving peers:", err)
	}
}
```

Kemudian buat file main.go yang berisi:
```go
package main

import (
	"bootstrap-peer/bootstrap"
	"fmt"
)

func main() {
	const port = "4000"
	const peersFile = "peers.json"

	// Initialize the Bootstrap server
	bs := bootstrap.NewBootstrapServer(peersFile)

	// Load existing peers from the file
	if err := bs.LoadPeers(); err != nil {
		fmt.Println("Error loading peers:", err)
		return
	}

	// Start the Bootstrap server
	bs.Listen(port)
}
```

Sementara file peers.json berisi array kosong atau `[]`.

Untuk menjalankan server, gunakan perintah `go run main.go`. Nah saat ini kita sudah berhasil membuat server bootstrap peer sederhana.

## 8.2 Integrasi Bootstrap Peer dengan Aplikasi Blockchain

Sebelum mengintegrasikan bootsrap peer dan aplikasi blockchain, kita akan memecah file `app/peer/peer.go` menjadi beberapa file sesuai dengan fungsionalitasnya.

Pindahkan sebagian kode terjait handler, ke file baru `app/peer/handler.go`

```go
package peer

import (
	"encoding/json"
	"fmt"
	"myapp/app/blockchain"
	"net"
)

// Handler untuk setiap koneksi peer
func (p2p *P2PNetwork) handleConnection(conn net.Conn) {
	defer conn.Close()

	decoder := json.NewDecoder(conn)
	var incomingBlockchain blockchain.Blockchain
	if err := decoder.Decode(&incomingBlockchain); err != nil {
		fmt.Println("Error decoding blockchain:", err)
		return
	}

	// Verifikasi dan update blockchain
	if p2p.VerifyAndUpdateBlockchain(&incomingBlockchain) {
		fmt.Println("Updated local blockchain with incoming blockchain.")
	} else {
		fmt.Println("Received invalid or shorter blockchain.")
	}
}

// Verifikasi dan update blockchain jika lebih panjang
func (p2p *P2PNetwork) VerifyAndUpdateBlockchain(incoming *blockchain.Blockchain) bool {
	if !incoming.IsValid() {
		return false
	}

	if len(incoming.Blocks) > len(p2p.Blockchain.Blocks) {
		p2p.Blockchain = incoming
		p2p.BroadcastBlockchain()
		return true
	}

	return false
}
```

Pindahkan sebagaan kode terkait broadcast ke file baru `app/peer/broadcast.go`. Pada kesempatan ini kita akan sekaligus mengganti json dengan sonic untuk performa yang lebih baik.

```go
package peer

import (
    "fmt"
    "github.com/bytedance/sonic"
    "net"
)

// BroadcastBlockchain mengirimkan seluruh blockchain ke semua peers yang terhubung.
func (p2p *P2PNetwork) BroadcastBlockchain() {
    for _, peer := range p2p.peers {
        conn, err := net.Dial("tcp", peer.address)
        if err != nil {
            fmt.Println("Error connecting to peer:", err)
            continue
        }
        defer conn.Close()

        // Serialisasi blockchain menggunakan Sonic
        data, err := sonic.Marshal(p2p.Blockchain)
        if err != nil {
            fmt.Println("Error encoding blockchain:", err)
            continue
        }

        // Kirim data blockchain ke peer
        if _, err := conn.Write(data); err != nil {
            fmt.Println("Error sending blockchain:", err)
        }
    }
}
```

Pindahkan sebagian kode terkait network ke file baru `app/peer/network.go`

```go
package peer

import (
	"encoding/json"
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
Dengan perubahan ini, file `app/peer/peer.go` hanya akan berisi kode-kode terkait peer saja :

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
	for _, peer := range p2p.peers {
		if peer.address == address {
			return // Peer sudah ada
		}
	}
	peer := Peer{address: address}
	p2p.peers = append(p2p.peers, peer)
}
```

Untuk mengintegrasikan server bootstrap peer dengan aplikasi blockchain, langkah-langkah berikut akan membantu memastikan bahwa peer yang baru bergabung bisa menemukan dan terhubung ke peers yang sudah ada. Kita akan memodifikasi aplikasi blockchain agar dapat:

1. Menghubungi bootstrap server saat memulai dan mendapatkan daftar peer.
2. Mendaftarkan dirinya sendiri ke bootstrap server.
3. Berkomunikasi dengan peers lain untuk sinkronisasi blockchain.

Untuk point pertama dan kedua, kita akan membuat fungsi RegisterToBootstrap di file `app/peer/peer.go` :

```go
package peer

import (
	"fmt"
	"myapp/app/blockchain"
	"net"

	"github.com/bytedance/sonic"
)

type Peer struct {
	address string
}

type P2PNetwork struct {
	peers      []Peer
	Blockchain *blockchain.Blockchain
	bootstrap  string 
	localAddr  string 
}

func NewP2PNetwork(bootstrap, localAddr string) *P2PNetwork {
	return &P2PNetwork{
		peers:      make([]Peer, 0),
		Blockchain: &blockchain.Blockchain{},
		bootstrap:  bootstrap,
		localAddr:  localAddr,
	}
}

func (p2p *P2PNetwork) AddPeer(address string) {
	for _, peer := range p2p.peers {
		if peer.address == address {
			return // Peer sudah ada
		}
	}
	peer := Peer{address: address}
	p2p.peers = append(p2p.peers, peer)
}

func (p2p *P2PNetwork) RegisterToBootstrap() error {
	conn, err := net.Dial("tcp", p2p.bootstrap)
	if err != nil {
		return fmt.Errorf("gagal terhubung ke bootstrap server: %v", err)
	}
	defer conn.Close()

	// Membuat payload untuk didaftarkan ke bootstrap
	peer := map[string]string{"address": p2p.localAddr}
	data, err := sonic.Marshal(peer)
	if err != nil {
		return fmt.Errorf("gagal melakukan marshal data: %v", err)
	}

	// Mengirim data ke bootstrap server
	if _, err := conn.Write(data); err != nil {
		return fmt.Errorf("gagal mengirim data ke bootstrap: %v", err)
	}

	fmt.Println("Berhasil mendaftar ke bootstrap server")
	return nil
}
```

Kita juga perlu menambahkan fungsi `GetPeersFromBootstrap()` di file `app/peer/network.go` :

```go
func (p2p *P2PNetwork) GetPeersFromBootstrap() ([]string, error) {
	conn, err := net.Dial("tcp", p2p.bootstrap)
	if err != nil {
		return nil, fmt.Errorf("gagal terhubung ke bootstrap server: %v", err)
	}
	defer conn.Close()

	// Membaca data dari koneksi
	data, err := io.ReadAll(conn)
	if err != nil {
		return nil, fmt.Errorf("gagal membaca data dari bootstrap server: %v", err)
	}

	// Mendekode data menggunakan sonic
	var peers []string
	if err := sonic.Unmarshal(data, &peers); err != nil {
		return nil, fmt.Errorf("gagal menerima daftar peers: %v", err)
	}

	fmt.Println("Daftar peers diterima:", peers)
	return peers, nil
}
```

Di file main.go akan ada penyesuaian dalam membuat jaringan P2PNetwork :
```go
	p2p := peer.NewP2PNetwork("localhost:4000", "localhost:"+*port)
```
Kemudian ubah penambahan peer secara manual dengan memanggil fungsi `GetPeersFromBootstrap()`:
```go
	// Menambahkan peer (bootstrap).
	peers, err := p2p.GetPeersFromBootstrap()
	if err != nil {
		fmt.Println("Error:", err)
		return
	}

	for _, peer := range peers {
		p2p.AddPeer(peer)
	}

	p2p.RegisterToBootstrap()
```

Untuk poin ketiga, dimana peer yang baru bergabung perlu melakukan sync existing blockchain yang ada di jaringan, perlu dibuatkan penyesuaian di Blockchain agar support sync.Mutex serta membuat fungsu baru yaitu `SyncWithPeer()`. Modifikasi file `app/blockchain/blockchain.go` menjadi :

```go
package blockchain

import (
	"bytes"
	"sync"
)

type Blockchain struct {
	Blocks   []Block
	Election *Election
	mu       sync.Mutex
}

func (bc *Blockchain) AddBlock(newBlock Block) bool {
	bc.mu.Lock()
	defer bc.mu.Unlock()

	for _, block := range bc.Blocks {
		if bytes.Equal(block.Hash, newBlock.Hash) {
			return false
		}
	}
	bc.Blocks = append(bc.Blocks, newBlock)
	return true
}

func (bc *Blockchain) IsValid() bool {
	for i := 1; i < len(bc.Blocks); i++ {
		currentBlock := bc.Blocks[i]
		prevBlock := bc.Blocks[i-1]

		if !bytes.Equal(currentBlock.PrevHash, prevBlock.Hash) {
			return false
		}

		pow := NewProofOfWork(&currentBlock)
		if !pow.Validate() {
			return false
		}
	}
	return true
}

func (bc *Blockchain) SyncWithPeer(peerBlocks []Block) {
	bc.mu.Lock()
	defer bc.mu.Unlock()

	if len(peerBlocks) > len(bc.Blocks) {
		bc.Blocks = peerBlocks
	}
}
```


## 8.3 Peer Discovery
Saat ada peer baru bergabung, peer-peer lain perlu melakukan discovery peer.

