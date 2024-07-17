# Bab 5: Membangun Peer-to-Peer Network

## 5.1 Dasar-dasar Jaringan Peer-to-Peer

Jaringan Peer-to-Peer (P2P) adalah infrastruktur jaringan yang terdiri dari beberapa node atau komputer yang saling terhubung secara langsung tanpa memerlukan server pusat atau hierarki yang terpusat. Dalam model ini, setiap node berperan ganda sebagai klien dan server, memungkinkan mereka untuk berbagi sumber daya, informasi, atau layanan dengan node lain dalam jaringan.

Keunggulan Jaringan Peer-to-Peer:

1. Desentralisasi: Tidak adanya otoritas tunggal atau server pusat membuat jaringan P2P lebih tahan terhadap kegagalan satu titik. Setiap node memiliki peran yang setara dalam jaringan, mempromosikan keamanan dan keberlanjutan sistem secara kolektif.

2. Skalabilitas: Jaringan P2P dapat diperluas dengan mudah dengan menambahkan node baru tanpa memerlukan perubahan besar pada infrastruktur yang ada. Ini membuatnya ideal untuk aplikasi yang memerlukan penyebaran besar dan cepat.

3. Efisiensi: Komunikasi langsung antar node memungkinkan pertukaran data yang efisien dan minim latensi. Hal ini mengurangi ketergantungan pada infrastruktur pusat dan mengoptimalkan penggunaan sumber daya jaringan.

4. Resiliensi: Jaringan P2P cenderung lebih tahan terhadap serangan atau gangguan karena tidak ada titik pusat yang dapat diserang. Data dan layanan dapat terus berjalan bahkan jika beberapa node mengalami kegagalan atau tidak aktif.

## 5.2 Implementasi Jaringan P2P di Golang

Implementasi jaringan P2P di Golang melibatkan penggunaan package standar net untuk manajemen koneksi dan komunikasi antar node. Di bawah adalah contoh sederhana bagaimana menginisialisasi node dan mendengarkan koneksi masuk:

```go
package main

import (
    "fmt"
    "net"
)

type Node struct {
    Address string
}

func main() {
    node := Node{
        Address: "localhost:6000",
    }

    listener, err := net.Listen("tcp", node.Address)
    if err != nil {
        fmt.Println("Error listening:", err.Error())
        return
    }
    defer listener.Close()

    fmt.Println("Listening on", node.Address)

    for {
        conn, err := listener.Accept()
        if err != nil {
            fmt.Println("Error accepting:", err.Error())
            return
        }
        fmt.Println("Connection accepted from", conn.RemoteAddr().String())
        go handleConnection(conn)
    }
}

func handleConnection(conn net.Conn) {
    // Handle incoming connections here
    defer conn.Close()
}
```

## 5.3 Menghubungkan dan Berkomunikasi dengan Peers

Untuk menghubungkan node dengan peers lain dalam jaringan, setiap node perlu dapat mengelola daftar peer yang dikenal dan mencoba melakukan koneksi ke mereka. Berikut adalah contoh sederhana bagaimana node dapat menghubungkan diri dengan peer:

```go
func (node *Node) connectToPeer(peerAddress string) error {
    conn, err := net.Dial("tcp", peerAddress)
    if err != nil {
        fmt.Println("Error connecting to peer:", err.Error())
        return err
    }
    defer conn.Close()
    fmt.Println("Connected to peer", peerAddress)
    // Handle further communication with peer
    return nil
}
```

## 5.4 Menangani Koneksi dan Pemutusan Koneksi

Dalam jaringan P2P, penting untuk menangani koneksi dan pemutusan koneksi dengan baik untuk menjaga integritas dan konsistensi jaringan. Anda dapat menggunakan goroutine untuk menangani setiap koneksi masuk dan keluar, serta mempertahankan daftar peer yang terhubung dan menangani skenario pemutusan koneksi dengan aman.

```go
func handleConnection(conn net.Conn) {
    defer conn.Close()
    // Handle incoming messages or data exchange here
}

func main() {
    // ...
    for {
        conn, err := listener.Accept()
        if err != nil {
            fmt.Println("Error accepting:", err.Error())
            return
        }
        fmt.Println("Connection accepted from", conn.RemoteAddr().String())
        go handleConnection(conn)
    }
}
```

Dengan memahami dasar-dasar jaringan P2P dan implementasinya di Golang, Anda dapat membangun infrastruktur yang mendukung komunikasi antar node secara langsung, sesuai dengan filosofi desentralisasi yang mendasari teknologi blockchain dan aplikasi terkait.
