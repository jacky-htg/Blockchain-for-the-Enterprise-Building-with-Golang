# Bab 9: Publikasi dan Subskripsi

Pada project aplikasi sistem pemilu yang telah dibuat di bab sebelumnya, hanya ada satu transaksi yaitu `BroadcastBlockchain` yanga mana node membroadcats seluruh blockchain (bukan hanya blok baru). Ketika ada blok baru dibuat maka akan ditambnahkan ke blockchain local, kemudian seluruh blockchain ini akan dibroadcast ke sleuruh peer. Pilihan untuk membroadcast seluruh blockchain dilakukan untuk menangani aturan rantai terpanjang. Ketika ada blockchain yang dibroadcast, peer bisa mengecek apakah rantai dari incoming blockchain lebih panjang dari local blockchain. Jika ranbtai incoming blockchain lebih panjang, maka peer akan mengganti local blockchain dengan incoming blockchain. 

Dengan sistem seperti ini, saya belum melihat kebutuhan untuk menerapkan pub sub dalam project sistem pemilu yang telah dibuat. Awalnya saya berencana untuk menghapus Bab 9 ini. Tapi setelah saya pikir kembali, saya akan tetap menulis Bab 9 untuk mengeksplorasi arsitektur Publish-Subscribe (Pub/Sub) dan aplikasinya dalam jaringan blockchain. 

Dalam jaringan terdesentralisasi, komunikasi antar node merupakan tantangan besar, terutama ketika jaringan berkembang dan skalabilitas menjadi krusial. Dengan Pub/Sub, pesan-pesan yang perlu disebarkan dalam jaringan tidak lagi harus di-broadcast langsung ke seluruh node. Sebaliknya, pesan dapat disampaikan hanya kepada node-node yang berlangganan topik tertentu, sehingga beban jaringan dapat berkurang secara signifikan.

Bab ini akan membahas dasar dari arsitektur Pub/Sub dan bagaimana sistem ini dapat diterapkan dalam jaringan blockchain untuk meningkatkan efisiensi komunikasi. Selain itu, kita akan melihat contoh implementasi dengan NATS sebagai sistem Pub/Sub yang populer, termasuk integrasinya dalam jaringan P2P yang telah dibangun. Dengan memahami Pub/Sub, pembaca akan mendapatkan wawasan tentang teknik komunikasi alternatif yang mampu mendukung jaringan blockchain skala besar dan meningkatkan fleksibilitas sistem.

## 9.1 Apa itu Pub/Sub?
Arsitektur Publish-Subscribe (Pub/Sub) adalah model komunikasi di mana pengirim pesan (publisher) dan penerima pesan (subscriber) tidak berinteraksi secara langsung satu sama lain. Alih-alih, pesan dikirimkan melalui saluran atau topik tertentu yang memungkinkan pesan hanya diterima oleh pihak-pihak yang berlangganan topik tersebut. Dengan cara ini, Pub/Sub memisahkan logika penerimaan dan pengiriman pesan, sehingga sistem menjadi lebih fleksibel dan efisien, terutama dalam jaringan besar.

Dalam konteks blockchain, Pub/Sub dapat membantu mengurangi lalu lintas komunikasi dengan memungkinkan node untuk menerima hanya informasi yang relevan, seperti notifikasi blok baru atau transaksi tertentu. Pub/Sub juga sangat cocok untuk skenario di mana node ingin terus memantau perkembangan tertentu dalam jaringan tanpa harus terhubung langsung ke semua node lainnya. Arsitektur ini tidak hanya mengurangi beban komunikasi di jaringan, tetapi juga membantu memastikan bahwa setiap node mendapatkan informasi yang dibutuhkan secara real-time.

## 9.2 Implementasi Pub/Sub di Jaringan Blockchain
Implementasi Publish-Subscribe (Pub/Sub) di jaringan blockchain memungkinkan node untuk berkomunikasi secara lebih efisien dengan hanya menerima pesan yang diperlukan, tanpa harus menerima setiap transaksi atau blok yang muncul di jaringan. Dengan model Pub/Sub, node dalam jaringan dapat berlangganan pada topik-topik tertentu, seperti blok baru atau transaksi valid, sehingga hanya menerima informasi yang relevan. Pendekatan ini sangat efektif untuk mengurangi beban jaringan, terutama dalam sistem blockchain besar dengan volume data tinggi.

Dalam jaringan blockchain, Pub/Sub dapat digunakan untuk berbagai keperluan, seperti distribusi blok baru ke node-node yang berlangganan, notifikasi tentang transaksi tertentu, atau bahkan sinkronisasi status antar node. Dengan model ini, setiap node dapat bertindak sebagai publisher atau subscriber sesuai kebutuhan, memungkinkan fleksibilitas yang lebih besar. Misalnya, dalam jaringan blockchain yang mendukung pemilu, setiap node dapat menerima hanya pembaruan yang berkaitan dengan transaksi HandleVote, tanpa harus menerima semua transaksi di jaringan. Implementasi Pub/Sub juga mendukung scalability, karena memungkinkan komunikasi yang lebih selektif dan terdistribusi dalam jaringan besar.

## 9.3 Menggunakan NATS Pub/Sub
NATS adalah sistem messaging berbasis Pub/Sub yang dikenal cepat, ringan, dan cocok untuk aplikasi terdistribusi. Dalam konteks blockchain, NATS dapat digunakan untuk mendistribusikan pesan antara node dengan latensi rendah dan keandalan yang tinggi. Dengan NATS, node blockchain dapat bertindak sebagai publisher atau subscriber, tergantung kebutuhan spesifiknya. Sistem ini mendukung komunikasi real-time dan memastikan bahwa pesan dapat dikirim secara efisien ke banyak node sekaligus.

Implementasi NATS di jaringan blockchain melibatkan pengaturan channel atau subject yang berfungsi sebagai topik, seperti NewBlock atau NewTransaction, yang relevan bagi node-node yang membutuhkan informasi tersebut. Setiap node yang berlangganan (subscribe) pada topik tertentu akan menerima pembaruan saat pesan dipublikasikan. Contoh di bawah ini menunjukkan cara sederhana untuk mengintegrasikan NATS dalam jaringan blockchain, memungkinkan node untuk berlangganan pembaruan blok baru.

``go
import (
	"fmt"
	"log"
	"github.com/nats-io/nats.go"
)

// Mengirim blok baru dengan NATS
func publishNewBlock(nc *nats.Conn, blockData string) {
	subject := "NewBlock"
	if err := nc.Publish(subject, []byte(blockData)); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Published new block to subject %s\n", subject)
}

// Menerima blok baru dengan NATS
func subscribeToNewBlock(nc *nats.Conn) {
	subject := "NewBlock"
	nc.Subscribe(subject, func(msg *nats.Msg) {
		fmt.Printf("Received new block data: %s\n", string(msg.Data))
	})
	fmt.Printf("Subscribed to subject %s\n", subject)
}

func main() {
	// Menghubungkan ke server NATS
	nc, _ := nats.Connect(nats.DefaultURL)
	defer nc.Close()

	// Memulai langganan
	subscribeToNewBlock(nc)

	// Mengirim blok baru
	publishNewBlock(nc, "Block data here")

	select {} // Menjaga program tetap berjalan
}
```

Dalam contoh ini, setiap node yang berlangganan pada topik NewBlock akan menerima data blok baru ketika dipublikasikan oleh node lainnya. Hal ini memungkinkan sinkronisasi jaringan secara efisien, karena hanya node yang relevan akan menerima pembaruan blok. Integrasi dengan NATS juga mempermudah pengelolaan pesan secara terdistribusi, mengurangi beban jaringan, dan memastikan setiap node selalu memiliki informasi terbaru tanpa harus mem-broadcast keseluruhan blockchain.

## 9.4 Integrasi Pub/Sub dengan Jaringan P2P
Integrasi sistem Pub/Sub dengan jaringan P2P dalam blockchain menciptakan arsitektur yang lebih efisien dan responsif. Dengan menggabungkan Pub/Sub dalam jaringan P2P, setiap node dapat berlangganan topik tertentu dan hanya menerima pesan yang relevan, daripada menerima semua pesan melalui mekanisme broadcast. Hal ini meningkatkan skalabilitas jaringan dengan mengurangi lalu lintas data yang tidak perlu, yang sangat penting dalam jaringan dengan banyak node dan volume transaksi yang tinggi.

Dalam konteks jaringan P2P berbasis blockchain, Pub/Sub dapat digunakan untuk menangani beberapa topik penting, seperti pembaruan blok baru, transaksi baru, atau pembaruan status node. Setiap node dalam jaringan dapat memilih untuk berlangganan topik yang spesifik, seperti NewBlock atau NewTransaction, yang memungkinkan komunikasi antar node secara lebih selektif dan efisien. Contoh di bawah ini menunjukkan cara mengintegrasikan Pub/Sub dengan jaringan P2P yang sudah ada, menggunakan NATS sebagai sistem pesan utama.

```go
// Fungsi untuk mempublikasikan transaksi baru
func publishTransaction(nc *nats.Conn, transactionData string) {
	subject := "NewTransaction"
	if err := nc.Publish(subject, []byte(transactionData)); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Published new transaction to subject %s\n", subject)
}

// Fungsi untuk berlangganan transaksi baru dalam jaringan P2P
func subscribeToTransaction(nc *nats.Conn) {
	subject := "NewTransaction"
	nc.Subscribe(subject, func(msg *nats.Msg) {
		fmt.Printf("Received new transaction data: %s\n", string(msg.Data))
		// Lakukan validasi transaksi, tambahkan ke mempool, atau proses lebih lanjut
	})
	fmt.Printf("Subscribed to subject %s for new transactions\n", subject)
}
```

Dengan integrasi ini, transaksi atau blok baru dapat diteruskan ke node-node yang berlangganan tanpa mengganggu node yang tidak memerlukannya. Sistem ini juga membantu mempermudah sinkronisasi node dengan mengirimkan pembaruan hanya ke node yang membutuhkan, tanpa perlu melakukan broadcast blockchain penuh. Hal ini menjadikan integrasi Pub/Sub sebagai pendekatan yang lebih hemat bandwidth dan ramah sumber daya untuk jaringan P2P berbasis blockchain, memungkinkan node untuk tetap sinkron secara optimal tanpa mengorbankan performa jaringan.
