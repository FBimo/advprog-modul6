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
</details>