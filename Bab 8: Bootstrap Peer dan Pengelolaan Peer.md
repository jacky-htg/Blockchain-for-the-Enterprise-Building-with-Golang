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
Kita akan membuat server bootstrap peer sederhana yang menyimpan daftar peer secara permanen ke dalam file dalam format JSON. Pertama, buat project/folder dengan `bootstrap-server`, dengan struktur folder seperti berikut:

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
Jalankan `go mod init bootstrap-server` untuk membuat project/module. Kemudian buat file `internal/bootstrap/peer.go` yang berguna untuk mengelola semua transaksi terkait peer:

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
	Payload interface{} `json:"payload"`
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
		s.registerPeer(req.Payload.(string), conn)
	case "GET_PEERS":
		s.getAllPeers(conn)
	case "REMOVE":
		s.removePeer(req.Payload.(string), conn)
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
	pm       *PeerManager
	jsonPath string
}

// NewServer membuat instance baru server.
func NewServer(port string, jsonPath string) *Server {
	return &Server{
		port:     port,
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

Jalankan client `go run client.go register localhost:3000`, `go run client.go get_peers` dan `go run client.go remove localhost:3000` untuk menguji interaksi antara client dengan server bootstrap.

Jika diperhatikan, saat server bootstrap di-start, server akan membaca file json untuk melihat peer apa saja yang ada di data, kemudian menyimpannya di memory. Jika server bootstrap di-shutdown, server akan memyimpan seluruh peer ke dalam file peer.json. Operasi ini berjalan dengan baik jika jumlah node masih sedikit. Jika jumlah node sudah sampai jutaan, maka proses load data json yang berformat text akan membebani kinerja server. Praktek yang baik adalah dengan menyimpan data peer ke dalam database yang efisien. Kita bisa menggunakan database yang ringans eperti sqllite, redis, RocksDB, atau BoltDB. Saya pribadi menyarankan untuk menggunakan database BadgerDB yang mempunyai performa handal. Berikut beberapa keunggulan BadgerDB :

1. Kinerja Tinggi: Key-Value Store berbasis LSM tree (seperti Redis) yang cepat untuk operasi tulis dan baca, sehingga cocok untuk aplikasi real-time seperti blockchain dan bootstrap server.
2. Built for Go: Ditulis sepenuhnya dalam Go dan mudah diintegrasikan dengan aplikasi Go, tanpa dependensi eksternal.
3. Skalabilitas dan Konsistensi: BadgerDB dirancang untuk menangani jutaan keys dengan konsumsi memori rendah, tanpa memuat semua data ke dalam memori.
4. Durable & Persistent: Data disimpan di disk, tetapi performa tetap tinggi karena memanfaatkan buffer dan caching internal.

Format text (json) dalam serialisasi dan deserialisasi data juga cukup memakan memory, untuk itu, kadang dipelrukan format serialisasi/deserialisasi lain yang lebih efisien. Salah satu serialisasi/deserialisasi yang efisien adalah protokol buffer (protobuff).

Saya telah membuat server bootstrap peer ini dalam versi badger dan protobuf. Jika tertarik, silahkan mengakses ke https://github.com/jacky-htg/bootstrap-peer-v2

## 8.2 Integrasi Bootstrap Peer dengan Aplikasi Blockchain

Untuk mengintegrasikan server bootstrap peer dengan aplikasi blockchain, langkah-langkah berikut akan membantu memastikan bahwa peer yang baru bergabung bisa menemukan dan terhubung ke peers yang sudah ada. Kita akan memodifikasi aplikasi blockchain agar dapat:

1. Menghubungi bootstrap server saat memulai dan mendapatkan daftar peer.
2. Mendaftarkan dirinya sendiri ke bootstrap server.
3. Berkomunikasi dengan peers lain untuk sinkronisasi blockchain.

Untuk point pertama dan kedua, kita akan membuat fungsi RegisterToBootstrap di file `app/peer/peer.go` :

```go
package peer

import (
	"bufio"
	"fmt"
	"myapp/app/blockchain"
	"net"

	"github.com/bytedance/sonic"
)

type RegisterRequest struct {
	Type    string `json:"type"`
	Payload string `json:"payload"`
}

// Peer struct
type Peer struct {
	Address string `json:"address"`
}

type P2PNetwork struct {
	peers      []Peer
	Blockchain *blockchain.Blockchain
	bootstrap  string // Alamat bootstrap server
	localAddr  string // Alamat lokal peer
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
		if peer.Address == address {
			return // Peer sudah ada
		}
	}
	peer := Peer{Address: address}
	p2p.peers = append(p2p.peers, peer)
}

func (p2p *P2PNetwork) RegisterToBootstrap() error {
	conn, err := net.Dial("tcp", p2p.bootstrap)
	if err != nil {
		return fmt.Errorf("gagal terhubung ke bootstrap server: %v", err)
	}
	defer conn.Close()

	// Membuat payload untuk didaftarkan ke bootstrap
	payload := RegisterRequest{
		Type:    "REGISTER",
		Payload: p2p.localAddr,
	}
	data, err := sonic.Marshal(payload)
	if err != nil {
		return fmt.Errorf("gagal melakukan marshal data: %v", err)
	}

	writer := bufio.NewWriter(conn)
	data = append(data, '\n')

	_, err = writer.WriteString(string(data))
	if err != nil {
		return fmt.Errorf("gagal mengirim request: %v", err)
	}
	writer.Flush()

	fmt.Println("Berhasil mendaftar ke bootstrap server")
	return nil
}
```

Kita juga perlu menambahkan fungsi `GetPeersFromBootstrap()` di file `app/peer/network.go` :

```go
func (p2p *P2PNetwork) GetPeersFromBootstrap() ([]Peer, error) {
	conn, err := net.DialTimeout("tcp", p2p.bootstrap, 30*time.Second)
	if err != nil {
		return nil, fmt.Errorf("gagal terhubung ke bootstrap server: %v", err)
	}
	defer conn.Close()

	reader := bufio.NewReader(conn)

	req := BaseRequest{Type: "GET_PEERS"}
	data, err := sonic.Marshal(req)
	if err != nil {
		return nil, fmt.Errorf("gagal membuat request JSON: %v", err)
	}
	writer := bufio.NewWriter(conn)
	data = append(data, '\n')

	_, err = writer.WriteString(string(data))
	if err != nil {
		return nil, fmt.Errorf("gagal mengirim request: %v", err)
	}
	writer.Flush()

	// Membaca response dari server
	response, err := reader.ReadString('\n')
	if err != nil {
		return nil, fmt.Errorf("gagal membaca response: %v", err)
	}

	var peers []Peer

	err = sonic.Unmarshal([]byte(strings.TrimSpace(response)), &peers)
	if err != nil {
		return nil, fmt.Errorf("gagal melakukan unmarshal response: %v", err)
	}

	return peers, nil
}
```

Di file main.go akan ada penyesuaian dalam membuat jaringan P2PNetwork, serta menambhakan untuk registrasi peer ke server bootstrap :
```go
	bootstrapAddress := "localhost:4000"
	loocalAddress := "localhost:" + *port

	// Membuat jaringan P2P dan kontrak voting.
	p2p := peer.NewP2PNetwork(bootstrapAddress, loocalAddress)

	p2p.RegisterToBootstrap()
```
Kemudian ubah penambahan peer secara manual dengan memanggil fungsi `GetPeersFromBootstrap()`:
```go
	peers, err := p2p.GetPeersFromBootstrap()
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	for _, peer := range peers {
		if peer.Address != loocalAddress {
			p2p.AddPeer(peer.Address)
		}
	}
```

Untuk poin ketiga, dimana peer yang baru bergabung perlu melakukan sync existing blockchain yang ada di jaringan, perlu dibuatkan penyesuaian di Blockchain agar support sync.Mutex serta membuat fungsu baru yaitu `SyncWithPeer()`. Sebelum itu, karena saat ini kita membutuhkan 2 endpoint, yaitu, `BroadcastBlockchain()` dan `SyncWithPeer()`, kita perlu mengubah `handleConnection()` agar support untuk menerima 2 enpoint yang berbeda.

Modifikasi file `app/peer/handler.go` menjadi:
```go
package peer

import (
	"bufio"
	"fmt"
	"myapp/app/blockchain"
	"net"
	"time"

	"github.com/bytedance/sonic"
)

type MessageType string

const (
	BlockchainUpdate MessageType = "blockchain_update"
)

type Message struct {
	Type MessageType `json:"type"`
	Data []byte      `json:"data"`
}

// Handler untuk setiap koneksi peer
func (p2p *P2PNetwork) handleConnection(conn net.Conn) {
	defer conn.Close()

	reader := bufio.NewReader(conn)

	// Baca request dari client
	data, err := reader.ReadBytes('\n')
	if err != nil {
		fmt.Println("Error reading from connection:", err)
		return
	}

	var incomingMessage Message
	if err := sonic.Unmarshal(data, &incomingMessage); err != nil {
		fmt.Println("Error decoding blockchain:", err)
		return
	}

	switch incomingMessage.Type {
	case BlockchainUpdate:
		// Verifikasi dan update blockchain
		var blockchaiData *blockchain.Blockchain
		if err := sonic.Unmarshal(incomingMessage.Data, &blockchaiData); err != nil {
			fmt.Println("Error decoding blockchain:", err)
			return
		}
		if p2p.VerifyAndUpdateBlockchain(blockchaiData) {
			fmt.Println("Updated local blockchain with incoming blockchain.")
		} else {
			fmt.Println("Received invalid or shorter blockchain.")
		}
	default:
		fmt.Println("Received unknown message type:", incomingMessage.Type)
	}
}

// Verifikasi dan update blockchain jika lebih panjang
func (p2p *P2PNetwork) VerifyAndUpdateBlockchain(incoming *blockchain.Blockchain) bool {
	if !incoming.IsValid() {
		return false
	}

	if len(incoming.Blocks) > len(p2p.Blockchain.Blocks) {
		p2p.Blockchain = incoming
		return true
	}

	return false
}

// Fungsi untuk menangani suara dari voter dan menambahkan blok baru.
func (p2p *P2PNetwork) HandleVote(voterID, candidateID string) {
	voteData := blockchain.VoteData{
		VoterID:     voterID,
		CandidateID: candidateID,
		Timestamp:   time.Now().Unix(),
	}

	newBlock := blockchain.Block{
		Index:     len(p2p.Blockchain.Blocks),
		Timestamp: time.Now().Unix(),
		Data:      voteData,
		PrevHash:  []byte{},
	}

	if len(p2p.Blockchain.Blocks) > 0 {
		newBlock.PrevHash = p2p.Blockchain.Blocks[len(p2p.Blockchain.Blocks)-1].Hash
	}

	pow := blockchain.NewProofOfWork(&newBlock)
	nonce, hash := pow.Run()
	newBlock.Hash = hash
	newBlock.Nonce = nonce

	if pow.Validate() {
		if p2p.Blockchain.AddBlock(newBlock) {
			err := p2p.Blockchain.Election.Vote(voterID, candidateID) // Pastikan Election ada di blockchain
			if err != nil {
				fmt.Println("Error while voting:", err)
			} else {
				p2p.BroadcastBlockchain()
				fmt.Println("Vote successful for voter:", voterID)
			}
		}
	} else {
		fmt.Println("Failed to validate block")
	}
}
```

Perubahan kode di atas adalah menambahkan messageType untuk mendeteksi request apakah request blockchain_update atau messageType yang lainnya, misal ke depan akan ditambahkan messageType sync_with_peer. Kemudian untuk struct message berisi messageType untuk deteksi endpoint, dan Data untuk payload request. Data type datanya []byte agar bisa diisi dengan struct yang bervariasi, karena tiap endpoint pasti melempar payload yang berbeda. Saya menambahkan penghapusan broadcast di fungsi `VerifyAndUpdateBlockchain()` agar tidak membuat flooding jaringan ketika setiap peer membroadcast ulang ketika mereka mendapat pembaruan blockchain.

Karena `handleConnection()` berubah, maka fungsi `BroadcastBlockchain()` juga harus disesuaikan agar mengirim payload berupa messageType blockchain_update dan Data dalam bentuk []byte. Modifikasi fungsi `BroadcastBlockchain()` di file `app/peer/broadcast.go` menjadi :

```go
func (p2p *P2PNetwork) BroadcastBlockchain() {
	for _, peer := range p2p.peers {
		conn, err := net.Dial("tcp", peer.Address)
		if err != nil {
			fmt.Println("Error connecting to peer:", err)
			continue
		}
		defer conn.Close()

		blockchainData, err := sonic.Marshal(p2p.Blockchain)
		if err != nil {
			fmt.Println("Gagal membuat JSON untuk Blockchain:", err)
			continue
		}
		message := Message{
			Type: BlockchainUpdate,
			Data: blockchainData,
		}
		data, err := sonic.Marshal(message)
		if err != nil {
			fmt.Println("gagal membuat request JSON:", err)
			continue
		}
		writer := bufio.NewWriter(conn)
		data = append(data, '\n')

		_, err = writer.WriteString(string(data))
		if err != nil {
			fmt.Println("gagal mengirim request:", err)
			continue
		}
		writer.Flush()
	}
}
```

Dalam kode di atas, saya juga mengubah pengrimiman data menggunakan bufio. Ini memungkinkan buffering saat membaca atau menulis, sehingga tidak langsung mengirim data byte per byte. Dengan bufio, data dikumpulkan dalam buffer terlebih dahulu, lalu ditulis atau dibaca dalam jumlah besar sekaligus. Hal ini mengurangi jumlah operasi I/O, yang bisa mempercepat proses, terutama saat mengirim data melalui jaringan. Karena mengurangi frekuensi operasi I/O yang mahal, bufio sering kali dapat mempercepat transfer data pada koneksi jaringan, sehingga mengurangi latensi dan mempercepat penulisan.

Setelah melakukan adjustmen terhadap fitur-fitur lama, kini saatnya kita menambah fitur baru, yaitu `SyncWithPeer()`. Seperti yang dibahas sebelumnya, operasi sync existing blockchain yang ada di jaringan, membutuhkan sync.Mutex untuk Blockchain, untuk itu mari ubah file `app/blockchain/blockchain.go` menjadi :

```go
package blockchain

import (
	"bytes"
	"fmt"
	"sync"
	"time"
)

type Blockchain struct {
	Blocks   []Block
	Election *Election
	mu       sync.Mutex
}

func (bc *Blockchain) SetGenesisBlock() bool {
	if len(bc.Blocks) == 0 {
		// Membuat dan membroadcast blok genesis.
		genesisBlock := Block{
			Index:     0,
			Timestamp: time.Now().Unix(),
			Data:      VoteData{VoterID: "system", CandidateID: "Genesis Block"},
		}
		pow := NewProofOfWork(&genesisBlock)
		nonce, hash := pow.Run()
		genesisBlock.Hash = hash
		genesisBlock.Nonce = nonce

		if pow.Validate() {
			if bc.AddBlock(genesisBlock) {
				fmt.Println("Added genesis block:", genesisBlock.Data.VoterID)
				return true
			}
		} else {
			fmt.Println("Failed to validate genesis block")
		}
	}

	return false
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

func (bc *Blockchain) SyncWithPeer(peerBlocks []Block, election *Election) {
	bc.mu.Lock()
	defer bc.mu.Unlock()

	if len(peerBlocks) > len(bc.Blocks) {
		bc.Blocks = peerBlocks
		bc.Election = election
	}
}

```

Agar peer baru dapat melakukan sinkronisasi blockchain saat bergabung, kita dapat menambahkan beberapa langkah tambahan ke dalam arsitektur. Berikut ini langkah-langkah yang dapat dilakukan:

1. Menambahkan Mekanisme Permintaan Sync pada Join Peer Baru
Ketika peer baru bergabung ke jaringan, ia harus meminta salinan blockchain terbaru dari peer lain. Buat fungsi yang akan meminta data blockchain dari satu atau beberapa peer yang sudah ada di jaringan. Fungsi ini bisa bernama RequestBlockchainFromPeers. 

2. Menerapkan Endpoint untuk Menyediakan Blockchain bagi Peer Lain
Peer existing harus memiliki cara untuk menanggapi permintaan sync dari peer baru dengan mengirim salinan blockchain lokal mereka. Kita dapat menambahkan endpoint baru di dalam jaringan P2P, misalnya HandleRequestBlockchain, yang mengirimkan salinan blockchain sebagai respons.

3. Memperbarui main.go untuk memanggil Fungsi untuk Memulai Sinkronisasi
Pada saat peer baru bergabung, lakukan proses sinkronisasi dengan memanggil RequestBlockchainFromPeers agar peer baru mendapatkan salinan blockchain terbaru. Jika peer baru menerima blockchain, ia akan menggunakan SyncWithPeer untuk memperbarui blockchain lokalnya.

Berikut implmentasinya, update file `app/peer/hanlder.go` menjadi :

```go
package peer

import (
	"bufio"
	"fmt"
	"myapp/app/blockchain"
	"net"
	"time"

	"github.com/bytedance/sonic"
)

type MessageType string

const (
	BlockchainUpdate  MessageType = "blockchain_update"
	RequestBlockchain MessageType = "request_blockchain"
)

type Message struct {
	Type MessageType `json:"type"`
	Data []byte      `json:"data"`
}

// Handler untuk setiap koneksi peer
func (p2p *P2PNetwork) handleConnection(conn net.Conn) {
	defer conn.Close()

	reader := bufio.NewReader(conn)

	// Baca request dari client
	data, err := reader.ReadBytes('\n')
	if err != nil {
		fmt.Println("Error reading from connection:", err)
		return
	}

	var incomingMessage Message
	if err := sonic.Unmarshal(data, &incomingMessage); err != nil {
		fmt.Println("Error decoding blockchain:", err)
		return
	}

	switch incomingMessage.Type {
	case BlockchainUpdate:
		// Verifikasi dan update blockchain
		var blockchaiData *blockchain.Blockchain
		if err := sonic.Unmarshal(incomingMessage.Data, &blockchaiData); err != nil {
			fmt.Println("Error decoding blockchain:", err)
			return
		}
		if p2p.VerifyAndUpdateBlockchain(blockchaiData) {
			fmt.Println("Updated local blockchain with incoming blockchain.")
		} else {
			fmt.Println("Received invalid or shorter blockchain.")
		}

	case RequestBlockchain:
		blockchainData, _ := sonic.Marshal(p2p.Blockchain)
		message := Message{Type: BlockchainUpdate, Data: blockchainData}
		response, _ := sonic.Marshal(message)
		conn.Write(response)

	default:
		fmt.Println("Received unknown message type:", incomingMessage.Type)
	}
}

// Verifikasi dan update blockchain jika lebih panjang
func (p2p *P2PNetwork) VerifyAndUpdateBlockchain(incoming *blockchain.Blockchain) bool {
	if !incoming.IsValid() {
		return false
	}

	if len(incoming.Blocks) > len(p2p.Blockchain.Blocks) {
		p2p.Blockchain = incoming
		for _, b := range p2p.Blockchain.Blocks {
			fmt.Println(b.Data)
		}
		return true
	}

	return false
}

// Fungsi untuk menangani suara dari voter dan menambahkan blok baru.
func (p2p *P2PNetwork) HandleVote(voterID, candidateID string) {
	for _, block := range p2p.Blockchain.Blocks {
		if block.Data.VoterID == voterID {
			fmt.Println("Voter already voted:", voterID)
			return
		}
	}

	voteData := blockchain.VoteData{
		VoterID:     voterID,
		CandidateID: candidateID,
		Timestamp:   time.Now().Unix(),
	}

	newBlock := blockchain.Block{
		Index:     len(p2p.Blockchain.Blocks),
		Timestamp: time.Now().Unix(),
		Data:      voteData,
		PrevHash:  []byte{},
	}

	if len(p2p.Blockchain.Blocks) > 0 {
		newBlock.PrevHash = p2p.Blockchain.Blocks[len(p2p.Blockchain.Blocks)-1].Hash
	}

	pow := blockchain.NewProofOfWork(&newBlock)
	nonce, hash := pow.Run()
	newBlock.Hash = hash
	newBlock.Nonce = nonce

	if pow.Validate() {
		if p2p.Blockchain.AddBlock(newBlock) {
			err := p2p.Blockchain.Election.Vote(voterID, candidateID) // Pastikan Election ada di blockchain
			if err != nil {
				fmt.Println("Error while voting:", err)
			} else {
				p2p.BroadcastBlockchain()
				fmt.Println("Vote successful for voter:", voterID)
			}
		}
	} else {
		fmt.Println("Failed to validate block")
	}
}

func (p2p *P2PNetwork) RequestBlockchainFromPeers() {
	for _, peer := range p2p.peers {
		conn, err := net.Dial("tcp", peer.Address)
		if err != nil {
			fmt.Println("Error connecting to peer:", err)
			continue
		}
		defer conn.Close()

		// Kirim permintaan untuk mendapatkan blockchain
		message := Message{Type: RequestBlockchain}
		data, _ := sonic.Marshal(message)
		writer := bufio.NewWriter(conn)
		data = append(data, '\n')

		_, err = writer.WriteString(string(data))
		if err != nil {
			fmt.Println("gagal mengirim request:", err)
			continue
		}
		writer.Flush()

		// Terima blockchain dari peer
		reader := bufio.NewReader(conn)
		peerData, _ := reader.ReadBytes('\n')
		var receivedMessage Message
		if err := sonic.Unmarshal(peerData, &receivedMessage); err != nil {
			fmt.Println("Error decoding blockchain from peer:", err)
			continue
		}

		// Pastikan type data adalah BlockchainUpdate sebelum disinkronkan
		if receivedMessage.Type == BlockchainUpdate {
			var peerBlockchain *blockchain.Blockchain
			if err := sonic.Unmarshal(receivedMessage.Data, &peerBlockchain); err != nil {
				fmt.Println("Error decoding blockchain blocks:", err)
				continue
			}
			p2p.Blockchain.SyncWithPeer(peerBlockchain.Blocks, peerBlockchain.Election)
			fmt.Println("Synchronized blockchain with peer:", peer.Address)
			break // Stop setelah sinkronisasi dengan satu peer
		}
	}
}
```

dan update file `main.go` menjadi:
```go
package main

import (
	"flag"
	"fmt"
	"myapp/app/blockchain"
	"myapp/app/peer"
	"os"
	"time"
)

func main() {
	port := flag.String("port", "5000", "Port to listen on")
	flag.Parse()

	bootstrapAddress := "localhost:4000"
	loocalAddress := "localhost:" + *port

	// Membuat jaringan P2P dan kontrak voting.
	p2p := peer.NewP2PNetwork(bootstrapAddress, loocalAddress)

	p2p.RegisterToBootstrap()

	// Inisialisasi blockchain dengan instance Election
	p2p.Blockchain = &blockchain.Blockchain{
		Blocks:   []blockchain.Block{},
		Election: blockchain.NewElection([]string{}),
	}

	peers, err := p2p.GetPeersFromBootstrap()
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	for _, peer := range peers {
		if peer.Address != loocalAddress {
			println(peer.Address)
			p2p.AddPeer(peer.Address)
		}
	}

	if *port == "3000" {
		p2p.Blockchain.Election.AddCandidate("Alice")
		p2p.Blockchain.Election.AddCandidate("Bob")
		p2p.Blockchain.Election.AddCandidate("Charlie")

		if !p2p.Blockchain.SetGenesisBlock() {
			p2p.BroadcastBlockchain()
		}
	} else {
		// Sinkronisasi blockchain untuk peer baru
		p2p.RequestBlockchainFromPeers()
	}

	// Mendengarkan koneksi untuk menerima blok.
	go p2p.ListenForBlocks(*port)

	// Contoh: Memproses suara untuk testing.
	go func() {
		time.Sleep(2 * time.Second) // Simulasi jeda waktu.
		if len(p2p.Blockchain.Blocks) > 0 {
			// Contoh pemrosesan suara
			p2p.HandleVote("voter123", "Alice")
			p2p.HandleVote("voter456", "Bob")

			// Menampilkan hasil voting
			p2p.Blockchain.Election.DisplayResults()
		}
	}()

	// Menjaga agar program tetap berjalan.
	select {}
}
```

## 8.3 Peer Discovery
Saat ada peer baru bergabung, peer-peer lain perlu melakukan discovery peer. Untuk melakukan peer discovery ini, ada beberapa pendekatan yang bisa dilakukan:

 1. Menggunakan Broadcast dari Bootstrap Server
Setiap kali peer baru bergabung, bootstrap server dapat memberitahu semua peer yang sudah ada tentang peer baru tersebut. Flow yang dilakukan adalah setelah peer yang baru join melakukan registrasi peer ke bootstrap, maka bootstrap akan melakukan broadcast ke seluruh existing peer agar mereka bisa menambhkan ke daftar peer mereka.

2. Periodic Peer Discovery (Polling)
Setiap peer melakukan polling berkala ke bootstrap server untuk mendapatkan daftar terbaru peers.

3. Broadcast dari Peer yang Baru
Setelah mendapatkan list peer yang ada di jaringan, peer baru bisa melanjutkan untuk melakukan broadcast keberadaan-nya ke seluruh peer yang ada.

5. Gossip protokol
Peer baru akan memberi tahu keberadaan dirinya ke salah satu peer. Kemudian peer yang diberi tahu akan memberi tahu ke peer lainya, berikut berantai sepertio sebuah gossip menyebar luas.

Karena kita sudah memiliki sebuah server bootstrap peer, saya akan menggunakan pendektan yang pertama. Berikut implementasinya:

Di aplikasi bootstrap, update file internal/bootstrap/handler.go, di dalam registerPeer, jika sudah sukses melakukan registrasi, call fungsi `s.broadcastPeers(peer)`. Kemudian masioh di file yang sama, buat fungsi broadcastPeers tersebut.

```go
func (s *Server) registerPeer(peer string, conn net.Conn) {
	success, msg := s.pm.RegisterPeer(peer)
	fmt.Fprintln(conn, msg)

	if !success {
		fmt.Println("Failed to register peer:", peer)
	} else {
		s.broadcastPeers(peer)
	}
}

// ... existing code

func (s *Server) broadcastPeers(peerAddress string) {
	s.pm.mu.Lock()
	defer s.pm.mu.Unlock()

	for _, existingPeer := range s.pm.peers {
		if existingPeer.Address != peerAddress {
			go s.notifyPeer(existingPeer.Address, peerAddress)
		}
	}
}

func (s *Server) notifyPeer(existingPeer, newPeerAddress string) {
	conn, err := net.Dial("tcp", existingPeer)
	if err != nil {
		fmt.Println("Error notifying peer:", err)
		return
	}
	defer conn.Close()

	message := Message{
		Type: NewPeerJoined,
		Data: []byte(newPeerAddress),
	}
	data, _ := sonic.Marshal(message)

	writer := bufio.NewWriter(conn)
	data = append(data, '\n')

	_, err = writer.WriteString(string(data))
	if err != nil {
		fmt.Println("Gagal mengirim request:", err)
		return
	}
	writer.Flush()
}
```

Untuk inetragsi dengan aplikasi blockchain, di dalam file `app/peer/handler.go` tambahakan endpoint `NewPeerJoined`

```go
const (
	BlockchainUpdate  MessageType = "blockchain_update"
	RequestBlockchain MessageType = "request_blockchain"
	NewPeerJoined     MessageType = "new_peer_joined"
)

// ... existing code

	case RequestBlockchain:
		blockchainData, _ := sonic.Marshal(p2p.Blockchain)
		message := Message{Type: BlockchainUpdate, Data: blockchainData}
		response, _ := sonic.Marshal(message)
		conn.Write(response)

	case NewPeerJoined:
		p2p.AddPeer(string(incomingMessage.Data))

	default:
		fmt.Println("Received unknown message type:", incomingMessage.Type)
	}
```

## 8.4 Penanganan Shutdown Peer
Jika ada satu peer yang mati/shutdown, maka peer tersebut harus memberitahu server bottstrap dengan memanggil endpoint `Remove`. Kemudian Server bootstrap harus membroadcast ke seluruh peer yang ada agar mereka menghapus peer tersebut.
