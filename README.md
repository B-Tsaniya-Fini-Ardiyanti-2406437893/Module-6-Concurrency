# Module-6-Concurrency

## Commit 1 Reflection
Pada milestone ini, saya mempelajari cara kerja server TCP dasar di Rust menggunakan `TcpListener` dan `TcpStream`. `TcpListener::bind("127.0.0.1:7878")` membuat soket yang mendengarkan koneksi masuk pada port 7878, dan `listener.incoming()` mengembalikan iterator atas aliran koneksi masuk.

Fungsi `handle_connection` bertanggung jawab untuk membaca dan mengurai permintaan HTTP yang dikirim oleh browser. Fungsi ini membungkus `TcpStream` di dalam `BufReader` untuk memungkinkan pembacaan baris demi baris yang di-buffer, yang lebih efisien daripada membaca byte demi byte. Iterator `.lines()` membaca setiap baris permintaan HTTP, dan `.take_while(|line| !line.is_empty())` berhenti mengumpulkan baris setelah menemukan baris kosong, yang menandai akhir bagian header HTTP.

Baris yang terkumpul disimpan dalam `Vec<String>` bernama `http_request`, yang kemudian dicetak ke konsol menggunakan `println!` dengan format `{:#?}` untuk pencetakan yang rapi. Dari output, saya dapat melihat bahwa permintaan HTTP terdiri dari baris permintaan (misalnya, `GET / HTTP/1.1`) diikuti oleh beberapa header seperti `Host`, `User-Agent`, dan `Accept`. Ini menunjukkan bahwa bahkan permintaan browser sederhana pun membawa cukup banyak metadata.

Salah satu pengamatan yang menarik adalah bahwa browser terkadang mengirimkan beberapa pesan "Connection established!". Ini terjadi karena browser secara otomatis mencoba kembali permintaan ketika tidak menerima respons, yang merupakan perilaku browser normal. Pada tahap ini, server hanya membaca permintaan tetapi belum mengirimkan respons apa pun.

## Commit 2 Reflection
Pada milestone ini, saya memodifikasi fungsi `handle_connection` sehingga server sekarang dapat mengirimkan respons HTTP aktual yang berisi halaman HTML kembali ke browser. Sebelumnya, server hanya membaca dan mencetak permintaan masuk tanpa membalas apa pun, itulah sebabnya browser menampilkan halaman kosong.

Penambahan kuncinya adalah `fs::read_to_string("hello.html")`, yang membaca seluruh isi file `hello.html` ke dalam `String`. File ini harus berada di direktori yang sama tempat `cargo run` dijalankan, jika tidak, program akan mengalami panic. Header `Content-Length` juga diperlukan dalam respons sehingga browser tahu persis berapa banyak byte yang diharapkan dalam isi respons.

Respons HTTP dibuat menggunakan makro `format!`, menggabungkan tiga bagian: baris status (`HTTP/1.1 200 OK`), header (`Content-Length: {length}`), dan isi (konten HTML). Pemisah `\r\n` penting karena protokol HTTP membutuhkan CRLF (Carriage Return + Line Feed) sebagai akhiran baris, dan baris kosong (`\r\n\r\n`) harus memisahkan header dari isi.

Terakhir, `stream.write_all(response.as_bytes())` mengkonversi string respons menjadi byte dan mengirimkannya melalui aliran TCP ke browser. Setelah perubahan ini, mengakses `http://127.0.0.1:7878` di browser sekarang menampilkan halaman HTML dengan benar dengan judul "Hello!" dan pesan khusus.