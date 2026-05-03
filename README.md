# Modul 10 Asynchronous Programming - Tutorial Timer

## Experiment 1.2: Understanding How It Works 
- ![Understanding how it works image](/assets/images/UnderstandingHowItWorks.png)
    - Explanation: 
        - "Nadia's Komputer: hey hey!" muncul lebih dulu sebelum "Nadia's Komputer: howdy!" karena `spawner.spawn()` hanya mendaftarkan task ke dalam antrian dan tidak langsung menjalankannya. Task baru benar-benar dieksekusi waktu `executor.run()` dipanggil. Jadinya kode` println!("Nadia's Komputer: hey hey!")` yang berada setelah `spawn()` itu langsung dijalankan secara sync, sementara task async menunggu turnnya di antrian. Kemudian ada jeda 2 secs antara "Nadia's Komputer: howdy!" dan "Nadia's Komputer: done!" karena di dalam task async ada `TimerFuture::new(Duration::new(2, 0)).await`. Waktu `.await` dipanggil, future akan mengembalikan `Poll::Pending` dan melepaskan kontrol thread sambil nunggu timernya selesai. Setelah 2 detik baru thread timer panggil `waker.wake()` untuk memberitahu executor jika task siap dilanjutkan, lalu executor mengpoll future lagi dan cetak "Nadia's Komputer: done!".


## Multiple Spawn and Removing Drop