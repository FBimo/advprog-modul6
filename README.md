# Refleksi Tutorial Pemrograman Lanjut

<details>
<summary style="font-size:24px">Tutorial 6</summary>

## Commit 1 Reflection Notes
Berikut merupakan penjelasan mengenai fungsi `handle_connection`,

```
...
fn handle_connection(mut stream: TcpStream)
...
```
Baris kode di atas berfungsi untuk mengambil _ownership_ dari `TcpStream` yang akan merepresentasikan koneksi TCP.


```
...
let buf_reader = BufReader::new(&mut stream);
...
```
Baris kode di atas akan membuat _instance_ `BufReader` yang nantinya akan digunakan untuk membaca teks dari _stream_ secara efisien.

```
...
let http_request: Vec<_> = buf_reader 
    .lines() 
    .map(|result| result.unwrap())
    .take_while(|line| !line.is_empty()) 
    .collect();
...
```
- `buf_reader .lines()` berguna untuk membaca setiap baris dari _buffered reader_.
- `.map(|result| result.unwrap())` melakukan pemetaan setiap _result_ ke _unwrapped value_-nya.
- `.take_while(|line| !line.is_empty())` digunakan untuk mengambil setiap baris yang ada sampai bertemu dengan baris kosong yang menandakan berakhirnya _request headers_ HTTP.
- `Vec<_>` berfungsi untuk menyimpan setiap baris yang sudah diambil dengan tipe yang menyesuaikan. 

```
...
println!("Request: {:#?}", http_request);
...
```
Mencetak setiap baris yang sudah dikumpulkan yang berisi _request_ HTTP. Jadi secara singkat, fungsi ini akan membaca _request_ HTTP dari _stream_ TCP dan akan berhenti apabila bertemu dengan baris yang kosong. 

## Commit 2 Reflection Notes

![Commit 2 screen capture](image.png)

Ada beberapa bagian kode yang ditambahkan untuk memunculkkan hasil yang diperoleh pada gambar di atas,
- mendefinisikan status `"HTTP/1.1 200 OK"`
- membaca konten dari `hello.html` dengan mengubahnya ke variabel _string_ `contents` menggunakan `fs::read_to_string("hello.html").unwrap()`
- melakukan kalkulasi panjang dari `contents` dengan `contents.len()`
- mengonstruksi _string_ respons HTTP termasuk di dalamnya ada status, panjang konten, dan konten dari `hello.html`
- kode perlu juga menuliskan respons HTTP kembali ke TCP _stream_ menggunakan `stream.write_all(response.as_bytes()).unwrap()`.

Berikut merupakan baris kode yang perlu ditambahkan di dalam fungsi `handle_connection`,

```
...
 let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!("{status_line}\r\nContent-Length:
        {length}\r\n\r\n{contents}");
        stream.write_all(response.as_bytes()).unwrap();
...
```
## Commit 3 Reflection Notes

![Commit 3 screen capture](image-1.png)

```
...
else {
            let status_line = "HTTP/1.1 404 NOT FOUND";
            let contents = fs::read_to_string("404.html").unwrap();
            let length = contents.len();
    
            let response = format!(
                "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
            );
    
            stream.write_all(response.as_bytes()).unwrap();
    }
...

```

Baris kode di atas ditambahkan untuk memunculkan halaman ketika suatu _file_ html yang di-_request_ tidak tersedia. Kode di atas dapat berjalan dengan baik sesuai dengan gambar yang tertera, namun kode di atas terdapat duplikasi penulisan kode yang dapat direfaktorisasi. Berikut merupakan hasil refaktorisasinya,

```
...
let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
            ("HTTP/1.1 200 OK", "hello.html")
        } else {
            ("HTTP/1.1 404 NOT FOUND", "404.html")
        };
    
        let contents = fs::read_to_string(filename).unwrap();
        let length = contents.len();
    
        let response =
            format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
    
        stream.write_all(response.as_bytes()).unwrap();
...
```

Dengan ada refaktorisasi di atas, kode yang ada terlihat semakin ringkas, mudah dipahami, dan apabila kita perlu untuk memodifikasi sesuatu, kita tidak perlu lagi sampai memodifikasi dua kode yang identik. 

## Commit 4 Reflection Notes

```
...
let (status_line, filename) = match &request_line[..] {
            "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
            "GET /sleep HTTP/1.1" => {
                thread::sleep(Duration::from_secs(5));
                ("HTTP/1.1 200 OK", "hello.html")
            }
           _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
        };
...
```

Modifikasi kode di atas berguna untuk melakukan tes terhadap kondisi server kita ketika menerapkan `sleep` sebelum melanjutkan _request_ lainnya. Pada contoh di atas durasi `sleep` atau waktu yang dibutuhkan server sebelum melakukan _request_ lainnya adalah lima detik. Tentu saja hal itu akan berdampak apabila terdapat beberapa pengguna yang akan melakukan _request_ secara beriringan. Pengguna harus menunggu setidaknya lima detik agar server bisa mengeksekusi _request_-nya. Hal ini terjadi karena kita sekarang memiliki tiga kondisi untuk ditangani sehingga diperlukan metode `match` yang berbeda dari metode `if-else` biasa. `match` tidak bisa melakukan _referencing_ atau _deferencing_ secara otomatis seperti metode _equality_.

## Commit 5 Reflection Notes

_ThreadPool_ adalah mekanisme dalam pemrograman yang digunakan untuk mengeksekusi banyak tugas secara bersamaan dengan menggunakan kumpulan _thread_ yang telah dibuat sebelumnya. Berikut merupakan penjabaran singkat cara kerjanya,

1. *Inisialisasi ThreadPool*: Pada awalnya, _ThreadPool_ dibuat dengan menentukan jumlah maksimum _thread_ yang ingin digunakan. Setiap _thread_ dalam _pool_ ini dapat digunakan untuk mengeksekusi tugas yang diberikan.

2. *Penjadwalan Tugas*: Ketika tugas baru dikirim untuk dieksekusi ke _ThreadPool_, _ThreadPool_ akan menempatkan tugas tersebut dalam antrian tugas yang menunggu. Antrian ini mungkin bersifat FIFO (_First In, First Out_) atau menggunakan prioritas tertentu tergantung pada implementasinya.

3. *Pengambilan Tugas*: Setiap _thread_ dalam _ThreadPool_ akan terus-menerus memeriksa antrian tugas yang menunggu. Ketika _thread_ tersedia, _thread_ mengambil tugas berikutnya dari antrian.

4. *Eksekusi Tugas*: Setelah mengambil tugas dari antrian, _thread_ mulai mengeksekusi tugas tersebut. Eksekusi tugas bisa memakan waktu yang berbeda tergantung pada kompleksitasnya.

5. *Pengembalian Hasil*: Setelah tugas selesai dieksekusi, hasilnya bisa dikembalikan ke pemanggil asli atau disimpan dalam suatu struktur data yang dapat diakses nanti.

6. *Penghentian ThreadPool*: Setelah selesai mengeksekusi semua tugas, _ThreadPool_ bisa dihentikan dan semua _thread_-nya dihentikan dan dibersihkan.

_ThreadPool_ sangat berguna dalam situasi ketika kita memiliki banyak tugas yang perlu dieksekusi secara bersamaan, seperti saat memproses _request_ HTTP dalam server web. Dengan menggunakan _ThreadPool_, kita dapat menghindari _overhead_ pembuatan dan penghancuran _thread_ yang terlalu sering sehingga meningkatkan efisiensi aplikasi dan kinerja secara keseluruhan.

</details>
