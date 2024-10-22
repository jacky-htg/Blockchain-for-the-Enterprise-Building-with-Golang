# Bab 6: Konsensus di Blockchain

Konsensus adalah mekanisme yang digunakan untuk mencapai kesepakatan di antara node dalam jaringan blockchain mengenai validitas transaksi dan urutan blok. Ini penting untuk memastikan bahwa semua node dalam jaringan memiliki salinan blockchain yang sama dan valid.

## 6.1 Memahami Konsensus di Blockchain

Konsensus di blockchain adalah proses di mana semua node dalam jaringan setuju pada satu versi dari data yang benar. Ini mencegah penipuan seperti double-spending dan memastikan integritas data. Mekanisme konsensus yang paling umum adalah Proof of Work (PoW) dan Proof of Stake (PoS), tetapi ada juga banyak alternatif lainnya.

PoW adalah mekanisme konsensus yang pertama kali digunakan di blockchain, sementara PoS hadir dengan pendekatan berbeda untuk mengatasi persoalan konsumsi energi. PoS adalah mekanisme konsensus yang memilih validator untuk membuat blok baru berdasarkan jumlah cryptocurrency yang mereka miliki dan "stake" dalam jaringan. PoS lebih hemat energi dibandingkan PoW.

## 6.2 Memilih Protokol Konsensus yang Tepat

Memilih protokol konsensus yang tepat tergantung pada berbagai faktor, termasuk kebutuhan keamanan, efisiensi energi, dan skala jaringan. PoW menawarkan keamanan tinggi tetapi tidak efisien secara energi, sementara PoS lebih efisien tetapi memerlukan mekanisme untuk mencegah sentralisasi kekuasaan.

Kesimpulan:

- Proof of Work (PoW): Baik untuk keamanan tinggi tetapi membutuhkan banyak energi.
- Proof of Stake (PoS): Lebih efisien energi, tetapi perlu pengaturan untuk memastikan desentralisasi.
- Alternatif lainnya: Seperti Delegated Proof of Stake (DPoS), Practical Byzantine Fault Tolerance (PBFT), dan lainnya yang menawarkan berbagai keseimbangan antara keamanan, efisiensi, dan desentralisasi.

Dengan memahami dan mengimplementasikan mekanisme konsensus ini, kita bisa memilih solusi yang paling sesuai dengan kebutuhan spesifik dari jaringan blockchain yang kita bangun.

Dalam bab ini, kita akan mengimplementasikan salah satu konsensus yang muncul paling awal, yaitu PoW.

## 6.3 Proof of Work (PoW) dan Implementasinya di Golang

Proof of Work (PoW) adalah mekanisme konsensus yang mengharuskan node untuk memecahkan teka-teki kriptografi yang kompleks untuk menambahkan blok baru ke blockchain. Berikut adalah implementasi PoW sederhana di Golang.

Struktur Folder:

```console
app
-- blockchain
---- data.go
---- block.go
---- blockchain.go
---- pow.go
-- peer
---- peer.go
main.go
go.mod
```

Pertama kita akan menambahkan Nonce di dalam struct Block. Update file app/blockchain/block.go

```go
type Block struct {
	Index     int
	Timestamp int64
	Data      Data
	PrevHash  []byte
	Hash      []byte
	Nonce     int
}
```

Kemudian buat File app/blockchain/pow.go:

```go
package blockchain

import (
	"bytes"
	"crypto/sha256"
	"fmt"
	"math/big"
)

const targetBits = 24

type ProofOfWork struct {
	block  *Block
	target *big.Int
}

func NewProofOfWork(b *Block) *ProofOfWork {
	target := big.NewInt(1)
	target.Lsh(target, uint(256-targetBits))
	return &ProofOfWork{b, target}
}

func (pow *ProofOfWork) prepareData(nonce int) []byte {
	data := bytes.Join(
		[][]byte{
			[]byte(fmt.Sprintf("%d", pow.block.Index)),
			[]byte(fmt.Sprintf("%d", pow.block.Timestamp)),
			[]byte(pow.block.Data.Data),
			pow.block.PrevHash,
			[]byte(fmt.Sprintf("%d", nonce)),
		},
		[]byte{},
	)
	return data
}

func (pow *ProofOfWork) Run() (int, []byte) {
	var hashInt big.Int
	var hash [32]byte
	nonce := 0

	for nonce < (1<<63 - 1) {
		data := pow.prepareData(nonce)
		hash = sha256.Sum256(data)
		hashInt.SetBytes(hash[:])

		if hashInt.Cmp(pow.target) == -1 {
			break
		} else {
			nonce++
		}
	}
	return nonce, hash[:]
}

func (pow *ProofOfWork) Validate() bool {
	var hashInt big.Int
	data := pow.prepareData(pow.block.Nonce)
	hash := sha256.Sum256(data)
	hashInt.SetBytes(hash[:])
	return hashInt.Cmp(pow.target) == -1
}
```
Dari struct ProofOfWork di atas, fungsi utama yang perlu dihighlight adalah fungsi Run(). Fungsi Run dalam ProofOfWork sangat penting untuk menjaga integritas dan keamanan blockchain. Mari kita bahas mengapa fungsi ini diperlukan dan bagaimana kaitannya dengan keamanan dari penipuan atau kecurangan:

Sebagai inti dari PoW, fungsi Run() mempunyai 2 tujuan penting, yaitu:
1. **Menemukan Nonce yang Valid**. Fungsi ini mencari nilai nonce yang menghasilkan hash blok yang memenuhi syarat target kesulitan (target). Hash yang dihasilkan harus lebih kecil dari nilai target yang telah ditentukan.
2. **Menyediakan Bukti Kerja**. Proof of Work (PoW) adalah mekanisme yang memastikan bahwa penambangan blok memerlukan usaha komputasi yang signifikan. Ini membuat proses penambangan memakan waktu dan energi.

### Proses dalam Fungsi Run
1. Menggabungkan Data. Fungsi prepareData menggabungkan beberapa elemen blok (seperti indeks, timestamp, data, hash sebelumnya, dan nonce) menjadi satu data yang akan di-hash.
2. Hashing dan Perbandingan. sha256.Sum256 digunakan untuk menghasilkan hash dari data yang sudah digabungkan.
Hash tersebut dikonversi menjadi bilangan besar (hashInt). Fungsi Cmp membandingkan hash dengan target. Jika hash lebih kecil dari target, maka nonce ditemukan dan loop berhenti.

### Mengapa Ini Penting untuk Keamanan
1. Menghindari Blok Palsu. Dengan memerlukan bukti kerja yang valid, sulit bagi seseorang untuk membuat blok palsu karena harus melakukan usaha komputasi yang sama.
2. Mencegah Penipuan. Proses PoW memastikan bahwa setiap blok yang ditambahkan ke blockchain adalah hasil dari usaha komputasi yang sah. Ini mencegah penambang dari mencoba menambahkan blok palsu atau merubah blok yang sudah ada tanpa melakukan usaha yang diperlukan.
3. Keamanan Terhadap Serangan Sybil. PoW membuat serangan Sybil (di mana seorang penyerang membuat banyak identitas palsu untuk menguasai jaringan) menjadi sangat mahal secara komputasi, karena setiap identitas palsu harus menyelesaikan PoW yang sah.

### Tanpa PoW
Jika tidak ada mekanisme seperti PoW, maka siapa saja bisa dengan mudah membuat dan menambahkan blok baru tanpa perlu usaha yang signifikan. Ini akan membuka pintu bagi berbagai bentuk penipuan dan kecurangan, seperti:

1. Blok Palsu. Penambang bisa dengan mudah menambahkan blok palsu ke blockchain tanpa perlu usaha komputasi.
2. Serangan Double-Spending. Penyerang bisa dengan mudah menciptakan blok baru untuk membalikkan transaksi dan melakukan pengeluaran ganda.
3. Pengendalian Jaringan. Tanpa mekanisme PoW, jaringan bisa dikuasai oleh pihak yang menciptakan banyak identitas palsu (serangan Sybil), sehingga mereka bisa mengendalikan blockchain dan memanipulasi data.

Jadi, fungsi Run dalam ProofOfWork adalah bagian krusial dari mekanisme keamanan yang menjaga integritas dan ketahanan blockchain terhadap berbagai bentuk serangan dan kecurangan.

Dalam implementasi dasar Proof of Work (PoW) yang telah kita bahas di atas, fokus utamanya adalah pada proses penambangan dan validasi blok oleh masing-masing node individu. Namun, PoW sebenarnya adalah bagian dari mekanisme konsensus di mana banyak node di jaringan perlu mencapai kesepakatan tentang blok mana yang akan ditambahkan ke blockchain.

Berikut ini adalah tambahan penjelasan tentang bagaimana konsensus di jaringan PoW melibatkan "voting" dari node di jaringan.

### Bagaimana Konsensus PoW Bekerja dalam Jaringan
1. Penambangan (Mining):
   - Setiap node di jaringan berlomba-lomba untuk memecahkan teka-teki kriptografi dengan menghitung hash dari blok yang memenuhi target kesulitan.
   - Node yang berhasil menemukan hash yang valid terlebih dahulu akan menambahkan blok tersebut ke blockchain-nya dan menyiarkan (broadcast) blok tersebut ke jaringan.

2. Voting oleh Node di Jaringan:
   - Ketika node menerima blok baru yang valid, mereka memverifikasi blok tersebut dengan memeriksa apakah hash-nya memenuhi target kesulitan dan apakah semua transaksi di dalam blok valid.
   - Jika blok valid, node akan menambahkan blok tersebut ke blockchain mereka. Ini adalah bentuk "voting" di mana node menyetujui blok yang baru ditambang. Jadi proses voting berlangsung secara implisit.

3. Kesepakatan Mayoritas:
   - Konsensus dicapai ketika mayoritas node di jaringan telah menambahkan blok yang sama ke blockchain mereka. Ini berarti mayoritas node telah "memvoting" blok tersebut sebagai valid.

4. Rantai Terpanjang:
   - Jika terdapat lebih dari satu blok valid yang diusulkan dalam waktu yang bersamaan, node akan terus menambang blok berikutnya. Blok yang menjadi bagian dari rantai terpanjang (dengan bukti kerja terbanyak) dianggap sebagai rantai yang valid oleh mayoritas node.
Ini dikenal sebagai aturan "rantai terpanjang", di mana rantai terpanjang akan menjadi blockchain yang diadopsi oleh seluruh jaringan.

### Memvalidasi PoW pada Fungsi handleConnection
Untuk memverifikasi bahwa blok yang diterima valid menurut PoW, kita perlu menambahkan validasi PoW pada fungsi handleConnection di file app/peer/peer.go

```go
func (p2p *P2PNetwork) handleConnection(conn net.Conn) {
	defer conn.Close()

	decoder := json.NewDecoder(conn)
	var block blockchain.Block
	if err := decoder.Decode(&block); err != nil {
		fmt.Println("Error decoding block:", err)
		return
	}

	pow := blockchain.NewProofOfWork(&block)
	if pow.Validate() {
		if p2p.Blockchain.AddBlock(block) {
			fmt.Println("Received and validated new block:", block)
			p2p.BroadcastBlock(block)
		}
	} else {
		fmt.Println("Received invalid block")
	}
}
```

Dengan implementasi ini, node di jaringan akan menerima blok baru, memverifikasi keabsahannya menggunakan PoW, dan kemudian menambahkan blok tersebut ke blockchain mereka jika valid. Node juga akan membroadcast blok baru yang mereka terima, memungkinkan seluruh jaringan mencapai konsensus berdasarkan aturan rantai terpanjang.

### Proses Penambahan Blok dengan PoW di Jaringan P2P
Berikut adalah contoh bagaimana menjalankan peer dengan PoW untuk proses penambahan blok.

```go
package main

import (
	"bufio"
	"flag"
	"fmt"
	"myapp/app/blockchain"
	"myapp/app/peer"
	"os"
	"time"
)

func readInput(p2p *peer.P2PNetwork) {
	scanner := bufio.NewScanner(os.Stdin)
	for scanner.Scan() {
		data := scanner.Text()
		newBlock := blockchain.Block{
			Index:     len(p2p.Blockchain.Blocks),
			Timestamp: time.Now().Unix(),
			Data:      blockchain.Data{Data: data},
			PrevHash:  p2p.Blockchain.Blocks[len(p2p.Blockchain.Blocks)-1].Hash,
		}
		pow := blockchain.NewProofOfWork(&newBlock)
		nonce, hash := pow.Run()
		newBlock.Hash = hash
		newBlock.Nonce = nonce

		if pow.Validate() {
			if p2p.Blockchain.AddBlock(newBlock) {
				p2p.BroadcastBlock(newBlock)
				fmt.Println("Added new block:", newBlock)
			}
		} else {
			fmt.Println("Failed to validate block")
		}
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
		// Membuat dan membroadcast blok genesis
		genesisBlock := blockchain.Block{Index: 0, Timestamp: time.Now().Unix(), Data: blockchain.Data{Data: "Genesis Block"}}
		pow := blockchain.NewProofOfWork(&genesisBlock)
		nonce, hash := pow.Run()
		genesisBlock.Hash = hash
		genesisBlock.Nonce = nonce

		if pow.Validate() {
			if p2p.Blockchain.AddBlock(genesisBlock) {
				p2p.BroadcastBlock(genesisBlock)
				fmt.Println("Added new block:", genesisBlock)
			}
		} else {
			fmt.Println("Failed to validate block")
		}
	}
	
	// Membaca input dari pengguna untuk menambahkan blok baru
	go readInput(p2p)

	// Menunggu agar program tetap berjalan
	select {}
}
```

Terakhir, di file app/blockchain/blockchain.go, update fungsi AddBlock() agar hanya menambhakan block baru jika belum ada di blockcahin. Jika sudah ada, maka tidak perlu melakukan apa-apa.

```go
func (bc *Blockchain) AddBlock(newBlock Block) bool {
	for _, block := range bc.Blocks {
		if bytes.Equal(block.Hash, newBlock.Hash) {
			return false
		}
	}
	bc.Blocks = append(bc.Blocks, newBlock)
	return true
}
```

Dengan perubahan fungsi AddBlock(), maka fungsi createBlock menjadi tidak ada yang menggunakan. Update app/blockchain/blockchain.go untuk menghapus createBlock().

Dengan pendekatan ini, Anda dapat melihat bagaimana proses konsensus bekerja dalam konteks PoW tanpa perlu voting eksplisit. Penambang yang pertama kali menemukan solusi yang valid memenangkan hak untuk menambahkan blok ke blockchain, dan seluruh jaringan setuju pada blok baru tersebut melalui aturan rantai terpanjang.

### Aturan Rantai Terpanjang

Saat ini, aplikasi blockchain yang dibangun belum mengimplementasikan konsep "Aturan Rantai Terpanjang". Ini dilakukan dengan membandingkan blockchain yang dibroadcast dengan blockchain di local. Jika blockchain yang dibroadcast lebih panjang, maka kita akan mengganti blockchain local dengan blockchain yang diterima.

Kita akan melakukannya dengan memodifikasi file app/peer/peer.go 

1. BroadcastBlock diganti menjadi BroadcastBlockchain
   
```go
func (p2p *P2PNetwork) BroadcastBlockchain() {
	// Mengirim blockchain ke semua peer
	for _, peer := range p2p.peers {
		conn, err := net.Dial("tcp", peer.address)
		if err != nil {
			fmt.Println("Error connecting to peer:", err)
			continue
		}
		defer conn.Close()

		encoder := json.NewEncoder(conn)
		if err := encoder.Encode(p2p.Blockchain); err != nil {
			fmt.Println("Error encoding blockchain:", err)
		}
	}
}
```

2. Fungsi handleConnection() ketika menerima blockchain yang dibroadcast, akan melakukan verifikasi blockchain, jika incoming blockchain lebih panjang dari local blockchain, maka update local blockchain dengan incomeing blockchain.

```go
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
```

3. Untuk itu perlu dibuat fungsi VerifyAndUpdateBlockchain(), yang logic dasarnya memanggil fungsi Blockchain.IsValid() dan ditambahakn logic untuk mengecek rantai terpanjang antara incoming dan local blockchain.

```go
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

4. Karena hash sekarang dihandle oleh fungsi pow.Run() maka ubah fungsi IsValid pada blockchain menjadi :

```go
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
``` 

5. Ubah kode di file main.go, dimana sebelumnya ada memanggil fungsi `p2p.BroadcastBlock(genesisBlock)` diubah menjadi `p2p.BroadcastBlockchain()` dan yang sebelumnya `p2p.BroadcastBlockc(newBlock)` doiubah menjadi `p2p.BroadcastBlockchain()`


Untuk mencoba menjalankan P2P, bisa lakukan seperti langkah sebelumnya.

```console
go run main.go -port=3001
```

```console
go run main.go -port=3002
```

dan 

```console
go run main.go -port=3000
```
