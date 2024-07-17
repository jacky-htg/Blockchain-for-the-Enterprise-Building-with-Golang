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

Update app/blockchain/block.go:

```go
type Block struct {
	Index     int
	Timestamp int64
	Data      Data
	PrevHash  []byte
	Hash      []byte
	Nonce     int
}

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
