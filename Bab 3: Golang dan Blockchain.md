# Bab 3: Golang dan Blockchain

## 3.1 Mengapa Memilih Golang untuk Blockchain?

Golang, atau Go, adalah bahasa pemrograman yang semakin populer dalam pengembangan aplikasi blockchain, terutama untuk proyek-proyek dengan skala besar dan kebutuhan kinerja yang tinggi. Berikut beberapa alasan mengapa Golang menjadi pilihan yang baik untuk pengembangan blockchain:

1. Performa Tinggi: Golang dirancang dengan fokus pada performa tinggi. Bahasa ini memiliki pengelolaan memori yang efisien dan kemampuan untuk menjalankan proses konkurensi dengan mudah, yang penting untuk memproses transaksi dalam blockchain dengan cepat dan efisien.

2. Concurrency Model: Golang menyediakan model konkurensi yang kuat dengan goroutines dan channels. Ini memungkinkan pengembang untuk dengan mudah menangani ribuan transaksi secara bersamaan, yang krusial dalam jaringan blockchain yang harus menangani banyak operasi secara simultan.

3. Pengembangan Mudah: Golang memiliki sintaksis yang sederhana dan jelas, membuatnya mudah dipelajari dan dipahami oleh pengembang baru maupun yang berpengalaman. Hal ini mempercepat proses pengembangan dan debugging dalam proyek blockchain yang kompleks.

4. Library dan Framework yang Kuat: Golang memiliki perpustakaan standar yang kaya dan framework-framework seperti Gorilla untuk web dan gRPC untuk komunikasi jaringan. Ini memudahkan pengembangan aplikasi blockchain yang terintegrasi dengan berbagai komponen seperti klien dan server, serta memungkinkan penggunaan teknologi-teknologi terbaru seperti microservices.

5. Skalabilitas: Dengan kemampuan untuk menangani konkurensi dengan baik dan performa tinggi, Golang cocok untuk pengembangan solusi blockchain yang skalabel. Ini penting mengingat tantangan dalam meningkatkan kapasitas jaringan blockchain tanpa mengorbankan kecepatan atau keamanan.

6. Dukungan Komunitas: Golang memiliki komunitas pengembang yang aktif dan berkembang pesat. Dukungan ini tidak hanya berarti adanya dokumentasi yang baik dan tutorial, tetapi juga adanya kontribusi berbagai pihak untuk meningkatkan ekosistem Golang secara keseluruhan.

Dengan kombinasi fitur-fitur ini, Golang menjadi pilihan yang kuat untuk pengembangan solusi blockchain yang efisien, scalable, dan dapat diandalkan. Konsistensi performa tinggi dan kemudahan dalam mengelola konkurensi membuatnya menjadi bahasa yang ideal untuk aplikasi yang membutuhkan keandalan dan kecepatan seperti yang diperlukan dalam teknologi blockchain.

## 3.2 Pengenalan Golang

Golang, atau Go, adalah bahasa pemrograman open source yang dikembangkan oleh Google pada tahun 2007 dan dirilis secara resmi pada tahun 2009. Disebut juga sebagai Go, bahasa ini dirancang dengan tujuan menyediakan keseimbangan antara kecepatan eksekusi, kemudahan pengembangan, dan efisiensi dalam penggunaan sumber daya.

### Fitur Utama Golang:

1. Sintaksis Sederhana: Golang didesain dengan sintaksis yang bersih dan sederhana, membuatnya mudah dipelajari dan dipahami oleh pengembang baru maupun yang berpengalaman.

2. Pengelolaan Memori Otomatis: Golang menggunakan garbage collector (pengumpul sampah) yang efisien untuk mengelola alokasi dan dealokasi memori, mengurangi beban tugas pengembang dalam manajemen memori secara manual.

3. Concurrency yang Kuat: Golang mendukung konkurensi dengan goroutines dan channels. Goroutines adalah unit eksekusi ringan yang memungkinkan pemrogram untuk menangani ribuan tugas secara simultan, sementara channels digunakan untuk komunikasi dan sinkronisasi antar goroutines.

4. Performa Tinggi: Golang diketahui memiliki performa yang sangat baik dalam menangani aplikasi dengan beban tinggi dan skala besar. Ini membuatnya ideal untuk pengembangan aplikasi server-side dan sistem terdistribusi seperti blockchain.

5. Library Standar yang Kaya: Golang dilengkapi dengan perpustakaan standar yang lengkap, memudahkan pengembang untuk mengimplementasikan berbagai fitur seperti pengolahan string, HTTP server, enkripsi, dan lainnya tanpa harus mengandalkan perpustakaan pihak ketiga.

6. Komunitas yang Aktif: Golang memiliki komunitas pengembang yang besar dan aktif, yang berkontribusi dalam pengembangan bahasa dan ekosistemnya. Dukungan ini tidak hanya mencakup dokumentasi yang baik, tetapi juga library dan framework-framework yang terus berkembang.

### Penggunaan Golang dalam Blockchain:

Penggunaan Golang dalam pengembangan aplikasi blockchain semakin populer karena kemampuannya dalam mengelola konkurensi, performa tinggi, dan mudahnya penggunaan. Dalam konteks blockchain, Golang digunakan untuk mengimplementasikan berbagai komponen seperti node, smart contracts, konsensus mechanisms, dan aplikasi terdistribusi lainnya.

Dengan fitur-fitur yang mencakup konkurensi yang kuat, manajemen memori otomatis, dan performa tinggi, Golang menjadi pilihan yang tepat untuk pengembangan solusi blockchain yang efisien dan scalable.

## 3.3 Instalasi dan Konfigurasi Lingkungan Pengembangan Golang

Untuk memulai pengembangan dengan Golang, langkah pertama yang perlu dilakukan adalah menginstal dan mengkonfigurasi lingkungan pengembangan. Berikut adalah langkah-langkah umum untuk melakukan hal ini:

1. Instalasi Golang:

Unduh Paket Instalasi: Kunjungi situs resmi Golang di [golang.org](https://go.dev) dan unduh paket instalasi yang sesuai dengan sistem operasi yang Anda gunakan (Windows, macOS, atau Linux).

Instalasi pada Windows:

Buka paket instalasi yang sudah diunduh (.msi) dan ikuti wizard instalasi.
Pastikan untuk menambahkan path bin Golang ke PATH environment variables.

Instalasi pada macOS:

Buka paket .pkg yang sudah diunduh dan ikuti langkah-langkah instalasi.
Setelah selesai, pastikan untuk menambahkan PATH ke direktori bin Golang ke dalam file .bash_profile, .bashrc, atau .zshrc.

Instalasi pada Linux:

Untuk distribusi Linux yang berbeda, instalasi dapat dilakukan dengan menggunakan package manager seperti apt, yum, atau dnf.
Setelah instalasi selesai, tambahkan PATH ke direktori bin Golang ke dalam file .bashrc atau .zshrc.

2. Konfigurasi Lingkungan Pengembangan:

Setting GOPATH: Golang menggunakan GOPATH sebagai lokasi default untuk menyimpan proyek dan dependensinya. Pastikan untuk mengatur GOPATH Anda ke direktori yang sesuai dengan struktur proyek Anda.

Menggunakan Go Modules (Opsional): Go Modules adalah fitur baru dalam Golang untuk mengelola dependensi proyek secara efisien. Anda dapat mengaktifkan Go Modules dengan membuat proyek baru di luar GOPATH atau dengan mengatur GO111MODULE=on di environment variables.

3. Verifikasi Instalasi:

Buka terminal atau command prompt dan ketikkan go version untuk memastikan bahwa instalasi Golang telah berhasil dan versi yang tepat telah terpasang.

4. Pengaturan Editor atau IDE:

Pilih editor atau IDE yang mendukung Golang, seperti Visual Studio Code, GoLand, atau editor favorit Anda.
Instal ekstensi atau plugin Golang untuk editor Anda agar mendukung penyorotan sintaks, autocompletion, dan fitur pengembangan lainnya.5. Mulai Pengembangan:

Buat proyek Golang pertama Anda dengan menggunakan perintah go mod init nama_proyek untuk menggunakan Go Modules, atau langsung mulai menulis kode Golang Anda dalam struktur direktori GOPATH yang sesuai.

Dengan mengikuti langkah-langkah di atas, Anda siap untuk memulai pengembangan aplikasi menggunakan Golang dengan lingkungan pengembangan yang sudah terkonfigurasi dengan baik.

## 3.4 Basic Golang

Golang, atau biasa disebut Go, adalah bahasa pemrograman yang dikembangkan oleh Google pada tahun 2007 dan dirilis secara resmi pada tahun 2009. Golang dirancang dengan tujuan untuk menyediakan kinerja komputasi yang efisien, kemudahan dalam penulisan kode, dan penanganan konkurensi yang baik. Berikut adalah beberapa konsep dasar yang perlu dipahami dalam pemrograman dengan Golang:

1. Hello World:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

Program di atas adalah program Hello World sederhana dalam Golang. fmt adalah paket bawaan (standard library) untuk formatting dan output. fmt.Println digunakan untuk mencetak teks ke konsol.

2. Variabel dan Tipe Data:

```go
package main

import "fmt"

func main() {
    // Deklarasi variabel
    var nama string = "John Doe"
    umur := 30 // Tipe data bisa disimpulkan (inferred)

    fmt.Printf("Nama: %s\n", nama)
    fmt.Printf("Umur: %d\n", umur)
}
```

Golang mendukung deklarasi variabel dengan tipe data eksplisit (var nama string) atau menggunakan penentuan tipe otomatis (umur := 30). Tipe data dasar meliputi int, string, bool, dan lainnya.

3. Struktur Kontrol:

```go
package main

import "fmt"

func main() {
    // Penggunaan if else
    umur := 18
    if umur >= 18 {
        fmt.Println("Dewasa")
    } else {
        fmt.Println("Remaja")
    }

    // Penggunaan for loop
    for i := 1; i <= 5; i++ {
        fmt.Println(i)
    }
}
```

Golang menggunakan struktur kontrol seperti if, else, dan for yang familiar. Perhatikan penggunaan {} untuk menandai awal dan akhir dari blok kode dalam Golang.

4. Fungsi:

```go
package main

import "fmt"

func tambah(a, b int) int {
    return a + b
}

func main() {
    hasil := tambah(10, 20)
    fmt.Println("Hasil penjumlahan:", hasil)
}
```

Fungsi dalam Golang dideklarasikan dengan menggunakan kata kunci func. Parameter fungsi didefinisikan dengan format nama tipe_data, dan tipe data kembalian dituliskan setelah parameter.

5. Struktur Data:

```go
package main

import "fmt"

type Mahasiswa struct {
    Nama    string
    Umur    int
    Jurusan string
}

func main() {
    mhs := Mahasiswa{"Alice", 20, "Informatika"}
    fmt.Println("Nama:", mhs.Nama)
    fmt.Println("Umur:", mhs.Umur)
    fmt.Println("Jurusan:", mhs.Jurusan)
}
```

Golang mendukung definisi struktur data menggunakan struct, yang memungkinkan pengelompokan data dengan tipe-tipe yang berbeda di dalam satu unit.

6. Konkurensi:

```go
package main

import (
    "fmt"
    "time"
)

func cetak(nama string) {
    for i := 1; i <= 5; i++ {
        fmt.Println(nama, ":", i)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    go cetak("Goroutine 1")
    go cetak("Goroutine 2")

    time.Sleep(1 * time.Second)
}
```

Golang didesain dengan dukungan konkurensi yang kuat, yang memungkinkan untuk menjalankan fungsi-fungsi secara paralel dengan menggunakan goroutines. Penggunaan go sebelum pemanggilan fungsi menandakan pembuatan goroutine baru.

Golang menawarkan banyak fitur lain seperti handling error, paket-paket bawaan yang kaya, dan performa yang baik, menjadikannya pilihan populer untuk pengembangan aplikasi blockchain dan banyak aplikasi lainnya. 

Jika Anda ingin mempelajari golang lebih lanjut, bisa membaca ["Go Guidance"](https://golang-microservices.rijalasepnugroho.com).
