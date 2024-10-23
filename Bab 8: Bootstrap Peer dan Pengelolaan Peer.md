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
	"encoding/json"
	"fmt"
	"net"
	"os"
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

	return json.Unmarshal(data, &bs.Peers)
}

// SavePeers saves the current peers to a JSON file
func (bs *BootstrapServer) SavePeers() error {
	data, err := json.Marshal(bs.Peers)
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

// Handle incoming connections from new peers
func (bs *BootstrapServer) handleConnection(conn net.Conn) {
	defer conn.Close()

	var peer Peer
	decoder := json.NewDecoder(conn)
	if err := decoder.Decode(&peer); err != nil {
		fmt.Println("Error decoding peer:", err)
		return
	}

	// Check if the peer is already in the list
	for _, p := range bs.Peers {
		if p.Address == peer.Address {
			fmt.Println("Peer already exists:", peer.Address)
			return
		}
	}

	// Add new peer to the list
	bs.Peers = append(bs.Peers, peer)
	fmt.Println("New peer added:", peer.Address)

	// Save the updated list of peers
	if err := bs.SavePeers(); err != nil {
		fmt.Println("Error saving peers:", err)
	}
}
```

Kemudian buat file main.go yang berisi:
```go
package main

import (
	"bootstrap-peer/bootstrap" // Ganti dengan path modul Anda
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



