# Modul 10 Asynchronous Programming - Tutorial Timer

## Experiment 1.2: Understanding How It Works 
- ![Understanding how it works image](/assets/images/UnderstandingHowItWorks.png)
    - Explanation: 
        - "Nadia's Komputer: hey hey!" muncul lebih dulu sebelum "Nadia's Komputer: howdy!" karena `spawner.spawn()` hanya mendaftarkan task ke dalam antrian dan tidak langsung menjalankannya. Task baru benar-benar dieksekusi waktu `executor.run()` dipanggil. Jadinya kode` println!("Nadia's Komputer: hey hey!")` yang berada setelah `spawn()` itu langsung dijalankan secara sync, sementara task async menunggu turnnya di antrian. Kemudian ada jeda 2 secs antara "Nadia's Komputer: howdy!" dan "Nadia's Komputer: done!" karena di dalam task async ada `TimerFuture::new(Duration::new(2, 0)).await`. Waktu `.await` dipanggil, future akan mengembalikan `Poll::Pending` dan melepaskan kontrol thread sambil nunggu timernya selesai. Setelah 2 detik baru thread timer panggil `waker.wake()` untuk memberitahu executor jika task siap dilanjutkan, lalu executor mengpoll future lagi dan cetak "Nadia's Komputer: done!".


## Experiment 1.3: Multiple Spawn and Removing Drop
- Without `drop(spawner)`:
    - ![Multiple spawn without drop image](/assets/images/MultipleSpawnWithoutDrop.png)
        - Explanation: Program ngeprint semua "Nadia's Komputer: howdy!" dan "Nadia's Komputer: done!" tapi nggak berhenti sendiri dan harus distop paksa pakai Ctrl+C karena `executor.run()` itu kerjanya nunggu task dari channel. Jika `spawner` nggak didrop, channelnya masih terbuka dan executor bakal nunggu terus siapa tau akan ada task baru yang masuk, padahal udah tidak ada task lagi, jadi programnya nyangkut selamanya.
- With `drop(spawner)`:
    - ![Multiple spawn with drop image](/assets/images/MultipleSpawnWithDrop.png)
        - Explanation: Program berjalan dengan normal dan berhenti sendiri setelah semua task selesai. Ketika `drop(spawner)` dipanggil, channel sendernya ditutup. Executor yang lagi nunggu di `ready_queue.recv()` akhirnya dapat sinyal bahwa tidak akan ada task baru lagi, jadi dia bisa keluar dari loop dan program selesai.
- Explanation tambahan: Urutan "Nadia's Komputer: done"nya berbeda-beda karena ketiga task berjalan secara asynchronous, jadi siapa yang timernya selesai duluan, dia yang lanjut duluan. Urutannya jadi bisa beda-beda tiap kali dijalankan.

