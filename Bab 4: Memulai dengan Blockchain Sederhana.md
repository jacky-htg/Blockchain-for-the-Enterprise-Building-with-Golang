# Bab 4: Memulai dengan Blockchain Sederhana

Blockchain adalah rangkaian blok yang diurutkan secara kronologis, dihubungkan satu sama lain menggunakan hash. Setiap blok menyimpan data transaksi serta hash dari blok sebelumnya. Di bawah ini adalah implementasi dasar blockchain menggunakan Golang.

## 4.1 Implementasi Blok dan Blockchain

Di Golang, kita mulai dengan mendefinisikan struktur Block yang mencakup Timestamp, Data, PrevBlockHash, dan Hash. Timestamp menandai waktu pembuatan blok, Data menyimpan informasi transaksi, PrevBlockHash adalah hash blok sebelumnya, dan Hash adalah hash dari blok saat ini.

```go
package main

import (
    "crypto/sha256"
    "encoding/hex"
    "time"
)

type Block struct {
    Timestamp     int64
    Data          []byte
    PrevBlockHash []byte
    Hash          []byte
}
```

## 4.2 Menghitung Hash dan Validasi Blok

Fungsi calculateHash digunakan untuk menghasilkan hash dari sebuah blok berdasarkan Timestamp, Data, dan PrevBlockHash.

```go
func calculateHash(block Block) []byte {
    record := string(block.Timestamp) + string(block.Data) + string(block.PrevBlockHash)
    h := sha256.New()
    h.Write([]byte(record))
    hashed := h.Sum(nil)
    return hashed
}

func isBlockValid(newBlock, oldBlock Block) bool {
    if oldBlock.Hash != newBlock.PrevBlockHash {
        return false
    }

    if calculateHash(newBlock) != newBlock.Hash {
        return false
    }

    return true
}
```

Fungsi isBlockValid memvalidasi keabsahan blok baru berdasarkan hash blok sebelumnya dan hash yang dihitung.

## 4.3 Membuat Genesis Block

Genesis block adalah blok pertama dalam blockchain. Ia diinisialisasi tanpa PrevBlockHash dan hash-nya dihitung berdasarkan data awal.

```go
func main() {
    genesisBlock := Block{
        Timestamp:     time.Now().Unix(),
        Data:          []byte("Genesis Block"),
        PrevBlockHash: []byte{},
    }
    genesisBlock.Hash = calculateHash(genesisBlock)

    blockchain := []Block{genesisBlock}

    newData := []byte("Data transaksi pertama")
    newBlock := generateBlock(blockchain[len(blockchain)-1], newData)
    blockchain = append(blockchain, newBlock)

    for _, block := range blockchain {
        // Output blok
    }
}
```

Dalam main(), kita membuat genesis block, menghitung hash-nya, dan menambahkannya ke blockchain. Setelah itu, kita membuat blok baru dan menambahkannya ke blockchain menggunakan fungsi generateBlock.

Dengan memahami langkah-langkah di atas, Anda dapat memulai membangun struktur dasar blockchain menggunakan Golang. Implementasi ini memberikan landasan yang solid untuk memahami konsep dasar blockchain dan bagaimana ia diimplementasikan dalam sebuah aplikasi menggunakan bahasa pemrograman Golang.
