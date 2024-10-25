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
.
├── cmd
│   └── main.go
├── data
│   └── peers.json
├── internal
│   └── bootstrap
│       ├── handler.go
│       ├── peer.go
│       └── server.go
├── pkg
│   └── models.go
└── go.mod
```
Jalankan `go mod init bootstrap-peer` untuk membuat project/module. Kemudian buat file `internal/bootstrap/peer.go` yang berguna untuk mengelola semua transaksi terkait peer:

```go
package bootstrap

import (
	"os"
	"sync"

	"github.com/bytedance/sonic"
)

// PeerManager mengelola daftar peers dengan thread-safe.
type PeerManager struct {
	peers []Peer
	mu    sync.Mutex
}

type Peer struct {
	Address string `json:"address"`
}

// NewPeerManager membuat instance baru dari PeerManager.
func NewPeerManager() *PeerManager {
	return &PeerManager{
		peers: make([]Peer, 0),
	}
}

// RegisterPeer menambahkan peer baru ke dalam daftar.
func (pm *PeerManager) RegisterPeer(peerAddress string) (bool, string) {
	pm.mu.Lock()
	defer pm.mu.Unlock()

	peer := Peer{Address: peerAddress}

	for _, p := range pm.peers {
		if p.Address == peerAddress {
			return false, "Peer already registered"
		}
	}

	pm.peers = append(pm.peers, peer)
	return true, "Peer registered successfully"
}

// GetAllPeers mengembalikan daftar semua peers.
func (pm *PeerManager) GetAllPeers() []Peer {
	pm.mu.Lock()
	defer pm.mu.Unlock()

	return pm.peers
}

// RemovePeer menghapus peer dari daftar.
func (pm *PeerManager) RemovePeer(peerAddress string) (bool, string) {
	pm.mu.Lock()
	defer pm.mu.Unlock()

	for i, p := range pm.peers {
		if p.Address == peerAddress {
			pm.peers = append(pm.peers[:i], pm.peers[i+1:]...)
			return true, "Peer removed successfully"
		}
	}

	return false, "Peer not found"
}

func (pm *PeerManager) loadPeers(jsonPath string) error {
	file, err := os.ReadFile(jsonPath)
	if err != nil {
		if os.IsNotExist(err) {
			return nil
		}
		return err
	}

	return sonic.Unmarshal(file, &pm.peers)
}

func (pm *PeerManager) savePeers(jsonPath string) error {
	file, err := sonic.Marshal(pm.peers)
	if err != nil {
		return err
	}

	return os.WriteFile(jsonPath, file, 0644)
}

```

Kemudian buat file `pkg/models.go` untuk menyimpan struct dari payload request:
```go
package pkg

// Request adalah model untuk request dari client.
type Request struct {
	Type    string `json:"type"`
	Payload string `json:"payload"`
}
```

Kemudian buat file `internal/bootstrap/handler.go` yang beguna sebagai handler dari semua service yang dimiliki oleh bootstrap server :
```go
package bootstrap

import (
	"bootstrap-server/pkg"
	"bufio"
	"fmt"
	"net"
	"os"
	"os/signal"
	"syscall"

	"github.com/bytedance/sonic"
)

// handleConnection menangani koneksi dan menentukan jenis request.
func (s *Server) handleConnection(conn net.Conn) {
	defer conn.Close()

	reader := bufio.NewReader(conn)

	// Baca request dari client
	data, err := reader.ReadBytes('\n')
	if err != nil {
		fmt.Println("Error reading from connection:", err)
		return
	}

	var req pkg.Request
	if err := sonic.Unmarshal(data, &req); err != nil {
		fmt.Println("Invalid request format:", err)
		return
	}

	// Handle sesuai tipe request
	switch req.Type {
	case "REGISTER":
		s.registerPeer(req.Payload, conn)
	case "GET_PEERS":
		s.getAllPeers(conn)
	case "REMOVE":
		s.removePeer(req.Payload, conn)
	default:
		fmt.Println("Invalid request type")
	}
}

func (s *Server) handleShutdown() {
	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt, syscall.SIGTERM)

	<-c
	fmt.Println("Shutdown initiated, saving peers...")

	if err := s.pm.savePeers(s.jsonPath); err != nil {
		fmt.Println("Gagal menyimpan peers:", err)
	} else {
		fmt.Println("Peers saved successfully.")
	}
	os.Exit(0)
}

// registerPeer menambahkan peer baru ke daftar.
func (s *Server) registerPeer(peer string, conn net.Conn) {
	success, msg := s.pm.RegisterPeer(peer)
	fmt.Fprintln(conn, msg)

	if !success {
		fmt.Println("Failed to register peer:", peer)
	}
}

func (s *Server) getAllPeers(conn net.Conn) {
	peers := s.pm.GetAllPeers()

	data, err := sonic.Marshal(peers)
	if err != nil {
		fmt.Println("Error encoding peers:", err)
		return
	}
	conn.Write(append(data, '\n'))
}

func (s *Server) removePeer(peer string, conn net.Conn) {
	success, msg := s.pm.RemovePeer(peer)
	fmt.Fprintln(conn, msg)

	if !success {
		fmt.Println("Failed to remove peer:", peer)
	}
}
```

Kemudian buat file `internal/bootstrap/server.go` sebagai server bootsrap, mengelola strat server dan shutdown server:
```go
package bootstrap

import (
	"fmt"
	"net"
)

// Server menyimpan data peer dan menangani koneksi.
type Server struct {
	port     string
	peers    map[string]bool
	pm       *PeerManager
	jsonPath string
}

// NewServer membuat instance baru server.
func NewServer(port string, jsonPath string) *Server {
	return &Server{
		port:     port,
		peers:    make(map[string]bool),
		pm:       NewPeerManager(),
		jsonPath: jsonPath,
	}
}

// ListenAndServe memulai server untuk menerima koneksi dari client.
func (s *Server) ListenAndServe() error {
	if err := s.pm.loadPeers(s.jsonPath); err != nil {
		return fmt.Errorf("gagal membaca file peers: %v", err)
	}

	ln, err := net.Listen("tcp", ":"+s.port)
	if err != nil {
		return fmt.Errorf("failed to listen on port %s: %v", s.port, err)
	}
	defer ln.Close()

	go s.handleShutdown()

	for {
		conn, err := ln.Accept()
		if err != nil {
			fmt.Println("Failed to accept connection:", err)
			continue
		}
		go s.handleConnection(conn)
	}
}
```
Kemudian buat file `cmd/main.go` yang berisi: 
```go
package main

import (
	"log"
	"os"

	"bootstrap-server/internal/bootstrap"
)

func main() {
	// Konfigurasi server (misalnya port diambil dari ENV atau default ke 4000)
	port := os.Getenv("BOOTSTRAP_PORT")
	if port == "" {
		port = "4000"
	}

	server := bootstrap.NewServer(port, "data/peers.json")
	log.Printf("Bootstrap server listening on port %s\n", port)

	// Jalankan server
	if err := server.ListenAndServe(); err != nil {
		log.Fatalf("Error starting server: %v", err)
	}
}
```

Sementara file `data/peers.json` berisi array kosong atau `[]`.

Untuk menjalankan server, gunakan perintah `go run cmd/main.go`. Nah saat ini kita sudah berhasil membuat server bootstrap peer sederhana.

Untuk melakukan testing buatlah file golang semisal `client.go` yang berisi :
```go
package main

import (
	"bufio"
	"encoding/json"
	"fmt"
	"net"
	"os"
	"strings"
	"time"
)

type Request struct {
	Type    string `json:"type"`
	Payload string `json:"payload"`
}

// Fungsi utama client untuk memanggil perintah registrasi, get peers, dan remove peer
func main() {
	if len(os.Args) < 2 {
		fmt.Println("Usage: go run main.go <command>")
		fmt.Println("Commands: register | get_peers | remove")
		return
	}

	command := os.Args[1]

	if command != "get_peers" && len(os.Args) < 3 {
		fmt.Println("Usage: go run main.go <command> <peer_address>")
		fmt.Println("Commands: register | remove")
		fmt.Println("Example: go run main.go register localhost:3000")
		return
	}

	var peerAddress string
	if len(os.Args) >= 3 {
		peerAddress = os.Args[2]
	}

	// Membuka koneksi TCP ke bootstrap server
	conn, err := net.DialTimeout("tcp", "localhost:4000", 10*time.Second)
	if err != nil {
		fmt.Println("Gagal terhubung ke server:", err)
		return
	}
	defer conn.Close()

	switch strings.ToLower(command) {
	case "register":
		sendRequest(conn, "REGISTER", peerAddress)
	case "get_peers":
		sendRequest(conn, "GET_PEERS", peerAddress)
	case "remove":
		sendRequest(conn, "REMOVE", peerAddress)
	default:
		fmt.Println("Perintah tidak dikenali:", command)
	}
}

// Fungsi untuk mengirim request dan membaca response dari server
func sendRequest(conn net.Conn, command, peerAddress string) {
	reader := bufio.NewReader(conn)

	req := Request{Type: command, Payload: peerAddress}
	data, err := json.Marshal(req)
	if err != nil {
		fmt.Println("gagal membuat request JSON: ", err)
		return
	}
	writer := bufio.NewWriter(conn)
	data = append(data, '\n')
	
	_, err = writer.WriteString(string(data))
	if err != nil {
		fmt.Println("Gagal mengirim request:", err)
		return
	}
	writer.Flush()

	// Membaca response dari server
	response, err := reader.ReadString('\n')
	if err != nil {
		fmt.Println("Gagal membaca response:", err)
		return
	}

	fmt.Println("Response dari server:", strings.TrimSpace(response))
}
```

Jalankan client `go run client.go register localhost:3000`, `go run client.go get_peers` dan `go run client.go remove localhost:3000` untuk menguji interaksi anatara client dengan server bootstrap.

Jika diperhatikan, saat server bootstrap di-start, server akan membaca file json untuk melihat peer apa saja yang ada di data, kemudian menyimpannya di memory. Jika server bootstrap di shutdown, server akan memyimpan seluruh peer ke dalam file peer.json. Operasi ini berjalan dengan baik jika jumlah node masih sedikit. Jika jumalah node sudah sampai jutaan, maka proses load data json yang berformat text akan sangat membebani kinerja server. Dan proses penyimpanan seluruh peer di dalam memory akan sangat memebebani server. Praktek yang baik adalah dengan menyimpan data peer ke dalam database yang efisien. Saya menyarakan untuk menggunakan database BadgerDB yang mempunyai performa handal. Berikut salah satu keunggulan BadgerDB :

1. Kinerja Tinggi: Key-Value Store berbasis LSM tree (seperti Redis) yang cepat untuk operasi tulis dan baca, sehingga cocok untuk aplikasi real-time seperti blockchain dan bootstrap server.
2. Built for Go: Ditulis sepenuhnya dalam Go dan mudah diintegrasikan dengan aplikasi Go, tanpa dependensi eksternal.
3. Skalabilitas dan Konsistensi: BadgerDB dirancang untuk menangani jutaan keys dengan konsumsi memori rendah, tanpa memuat semua data ke dalam memori.
4. Durable & Persistent: Data disimpan di disk, tetapi performa tetap tinggi karena memanfaatkan buffer dan caching internal.

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

