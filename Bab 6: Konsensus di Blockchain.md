# Bab 6: Konsensus di Blockchain

Konsensus adalah mekanisme yang digunakan untuk mencapai kesepakatan di antara node dalam jaringan blockchain mengenai validitas transaksi dan urutan blok. Ini penting untuk memastikan bahwa semua node dalam jaringan memiliki salinan blockchain yang sama dan valid.

## 6.1 Memahami Konsensus di Blockchain

Konsensus di blockchain adalah proses di mana semua node dalam jaringan setuju pada satu versi dari data yang benar. Ini mencegah penipuan seperti double-spending dan memastikan integritas data. Mekanisme konsensus yang paling umum adalah Proof of Work (PoW) dan Proof of Stake (PoS), tetapi ada juga banyak alternatif lainnya.

## 6.2 Proof of Work (PoW) dan Implementasinya di Golang

Proof of Work (PoW) adalah mekanisme konsensus yang mengharuskan node untuk memecahkan teka-teki kriptografi yang kompleks untuk menambahkan blok baru ke blockchain. Berikut adalah implementasi PoW sederhana di Golang.

Struktur Folder:

```shell
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

File app/blockchain/pow.go:

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

Update app/blockchain/blockchain.go untuk menambahkan pow saat createBlock():

```go
func createBlock(index int, data string, prevHash []byte) Block {
	newData := Data{Data: data}
	block := Block{
		Index:     index,
		Timestamp: time.Now().Unix(),
		Data:      newData,
		PrevHash:  prevHash,
		Hash:      []byte{},
	}
	pow := NewProofOfWork(&block)
	nonce, hash := pow.Run()
	block.Hash = hash
	block.Nonce = nonce
	return block
}
```

Dalam implementasi dasar Proof of Work (PoW) yang telah kita bahas, fokus utamanya adalah pada proses penambangan dan validasi blok oleh masing-masing node individu. Namun, PoW sebenarnya adalah bagian dari mekanisme konsensus di mana banyak node di jaringan perlu mencapai kesepakatan tentang blok mana yang akan ditambahkan ke blockchain.

Berikut ini adalah tambahan penjelasan tentang bagaimana konsensus di jaringan PoW melibatkan "voting" dari node di jaringan.

### Bagaimana Konsensus PoW Bekerja dalam Jaringan
1. Penambangan (Mining):
   - Setiap node di jaringan berlomba-lomba untuk memecahkan teka-teki kriptografi dengan menghitung hash dari blok yang memenuhi target kesulitan.
   - Node yang berhasil menemukan hash yang valid terlebih dahulu akan menambahkan blok tersebut ke blockchain-nya dan menyiarkan (broadcast) blok tersebut ke jaringan.

2. Voting oleh Node di Jaringan:
   - Ketika node menerima blok baru yang valid, mereka memverifikasi blok tersebut dengan memeriksa apakah hash-nya memenuhi target kesulitan dan apakah semua transaksi di dalam blok valid.
   - Jika blok valid, node akan menambahkan blok tersebut ke blockchain mereka. Ini adalah bentuk "voting" di mana node menyetujui blok yang baru ditambang.

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
		p2p.Blockchain.AddBlock(block.Data.Data)
		fmt.Println("Received and validated new block:", block)
		p2p.BroadcastBlock(block)
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
			p2p.Blockchain.Blocks = append(p2p.Blockchain.Blocks, newBlock)
			p2p.BroadcastBlock(newBlock)
			fmt.Println("Added new block:", newBlock)
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

	// Membuat dan membroadcast blok genesis
	genesisBlock := blockchain.Block{Index: 0, Timestamp: time.Now().Unix(), Data: blockchain.Data{Data: "Genesis Block"}}
	p2p.Blockchain.AddBlock(genesisBlock.Data.Data)
	p2p.BroadcastBlock(genesisBlock)

	// Membaca input dari pengguna untuk menambahkan blok baru
	go readInput(p2p)

	// Menunggu agar program tetap berjalan
	select {}
}
```
Dengan pendekatan ini, Anda dapat melihat bagaimana proses konsensus bekerja dalam konteks PoW tanpa perlu voting eksplisit. Penambang yang pertama kali menemukan solusi yang valid memenangkan hak untuk menambahkan blok ke blockchain, dan seluruh jaringan setuju pada blok baru tersebut melalui aturan rantai terpanjang.

### 6.3 Proof of Stake (PoS) dan Alternatif Lainnya

Proof of Stake (PoS) adalah mekanisme konsensus yang memilih validator untuk membuat blok baru berdasarkan jumlah cryptocurrency yang mereka miliki dan "stake" dalam jaringan. PoS lebih hemat energi dibandingkan PoW.

Pseudo-Implementation:

```go
package blockchain

import (
	"math/rand"
	"time"
)

type Validator struct {
	Address string
	Stake   int
}

type PoSNetwork struct {
	Validators []Validator
}

func NewPoSNetwork() *PoSNetwork {
	return &PoSNetwork{Validators: []Validator{}}
}

func (pos *PoSNetwork) AddValidator(address string, stake int) {
	validator := Validator{Address: address, Stake: stake}
	pos.Validators = append(pos.Validators, validator)
}

func (pos *PoSNetwork) SelectValidator() Validator {
	totalStake := 0
	for _, v := range pos.Validators {
		totalStake += v.Stake
	}
	rand.Seed(time.Now().Unix())
	r := rand.Intn(totalStake)
	runningTotal := 0
	for _, v := range pos.Validators {
		runningTotal += v.Stake
		if runningTotal > r {
			return v
		}
	}
	return pos.Validators[0]
}
```

## 6.4 Memilih Protokol Konsensus yang Tepat

Memilih protokol konsensus yang tepat tergantung pada berbagai faktor, termasuk kebutuhan keamanan, efisiensi energi, dan skala jaringan. PoW menawarkan keamanan tinggi tetapi tidak efisien secara energi, sementara PoS lebih efisien tetapi memerlukan mekanisme untuk mencegah centralisasi kekuasaan.

Kesimpulan:

- Proof of Work (PoW): Baik untuk keamanan tinggi tetapi membutuhkan banyak energi.
- Proof of Stake (PoS): Lebih efisien energi, tetapi perlu pengaturan untuk memastikan desentralisasi.
- Alternatif lainnya: Seperti Delegated Proof of Stake (DPoS), Practical Byzantine Fault Tolerance (PBFT), dan lainnya yang menawarkan berbagai keseimbangan antara keamanan, efisiensi, dan desentralisasi.

Dengan memahami dan mengimplementasikan mekanisme konsensus ini, kita bisa memilih solusi yang paling sesuai dengan kebutuhan spesifik dari jaringan blockchain yang kita bangun.
