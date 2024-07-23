# Bab 4: Memulai dengan Blockchain Sederhana

Blockchain adalah rangkaian blok yang diurutkan secara kronologis, dihubungkan satu sama lain menggunakan hash. Setiap blok menyimpan data transaksi serta hash dari blok sebelumnya. Di bawah ini adalah implementasi dasar blockchain menggunakan Golang.

## 4.1 Implementasi Blok dan Blockchain

Struktur aplikasi kita akan berupa :
```shell
app
-- blockchain
---- data.go
---- block.go
---- blockchain.go
main.go
go.mod
```

Untuk membuat modul, jalankan perintah ini di shell 

```shell
go mod init myapp
```

File app/blockchain/data.go berisi : 
```go
package blockchain

type Data struct {
	Data string
}
```

Di Golang, kita mulai dengan mendefinisikan struktur Block yang mencakup Index, Timestamp, Data, PrevHash, dan Hash. Index menandai posisi block berada di indeks ke berapa dalam blockchain, Timestamp menandai waktu pembuatan blok, Data menyimpan informasi transaksi, PrevHash adalah hash blok sebelumnya, dan Hash adalah hash dari blok saat ini. berikut kode app/blockchain/block.go

```go
package blockchain

type Block struct {
	Index     int
	Timestamp int64
	Data      Data
	PrevHash  []byte
	Hash      []byte
}
```

Sementara Blockchain berisi dari kumpulan Block. 

```go
package blockchain

import (
	"bytes"
	"time"
)

type Blockchain struct {
	Blocks []Block
}

func (bc *Blockchain) AddBlock(data string) Block {
	var newBlock Block
	if len(bc.Blocks) == 0 {
		newBlock = createBlock(0, "Genesis Block", []byte{})
	} else {
		prevBlock := bc.Blocks[len(bc.Blocks)-1]
		newBlock = createBlock(prevBlock.Index+1, data, prevBlock.Hash)
	}
	bc.Blocks = append(bc.Blocks, newBlock)
	return newBlock
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
	block.Hash = block.calculateHash()
	return block
}
```

## 4.2 Menghitung Hash dan Validasi Blok

Fungsi calculateHash digunakan untuk menghasilkan hash dari sebuah blok berdasarkan Timestamp, Data, dan PrevHash. Update file app/blockchain/block.go untuk menambahkan fungsi calculateHash() :

```go
import (
	"crypto/sha256"
	"fmt"
)

func (b *Block) calculateHash() []byte {
	record := fmt.Sprintf("%d%d%s%s", b.Index, b.Timestamp, b.Data.Data, b.PrevHash)
	h := sha256.New()
	h.Write([]byte(record))
	hashed := h.Sum(nil)
	return hashed
}
```

Untuk memvalidasi keabsahan blockchain, kita akan menambahkan Fungsi IsValid. Tambahkan kode berikut di file app/blockchain/blockchain.go

```
func (bc *Blockchain) IsValid() bool {
	for i := 1; i < len(bc.Blocks); i++ {
		currentBlock := bc.Blocks[i]
		prevBlock := bc.Blocks[i-1]

		if !bytes.Equal(currentBlock.Hash, currentBlock.calculateHash()) {
			return false
		}
		if !bytes.Equal(currentBlock.PrevHash, prevBlock.Hash) {
			return false
		}
	}
	return true
}
```

## 4.3 Membuat Genesis Block

Genesis block adalah blok pertama dalam blockchain. Ia diinisialisasi tanpa PrevHash dan hash-nya dihitung berdasarkan data awal.

```go
package main

import (
	"fmt"
	"time"

	"myapp/app/blockchain"
)

func main() {
	// Membuat blockchain baru
	bc := blockchain.Blockchain{}

	// Menambahkan genesis block
	bc.AddBlock("Genesis Block")

	// Menambahkan beberapa transaksi
	bc.AddBlock("Transaction 1")
	bc.AddBlock("Transaction 2")

	// Menampilkan semua blok dalam blockchain
	fmt.Println("Blockchain:")
	for _, block := range bc.Blocks {
		fmt.Printf("Index: %d\n", block.Index)
		fmt.Printf("Timestamp: %s\n", time.Unix(block.Timestamp, 0))
		fmt.Printf("Data: %s\n", block.Data.Data)
		fmt.Printf("PrevHash: %x\n", block.PrevHash)
		fmt.Printf("Hash: %x\n", block.Hash)
		fmt.Println("------------------------------")
	}

	// Validasi blockchain
	fmt.Println("Is blockchain valid?", bc.IsValid())
}

```

Dalam main(), kita membuat genesis block. Setelah itu, kita membuat blok baru dan menambahkannya ke blockchain. Kita juga membuat transaksi baru dan menambahkan-nya ke dalam blockchain, serta melakukan pengecekan apakah blockchain valid atau tidak.

Kita bisa mengetes program di atas dengan menjalankan di shell

```shell
go run main.go
```

Dengan memahami langkah-langkah di atas, Anda dapat memulai membangun struktur dasar blockchain menggunakan Golang. Implementasi ini memberikan landasan yang solid untuk memahami konsep dasar blockchain dan bagaimana ia diimplementasikan dalam sebuah aplikasi menggunakan bahasa pemrograman Golang.
