# Proposal Aplikasi Penilaian Pencak Silat
## *Sudut Silat*

> Versi: 1.0 — 3 Mei 2026
> Diajukan kepada: Panitia Pertandingan Pencak Silat
> Disusun oleh: Tim Pengembang Sudut Silat

---

## Sekilas Tentang Aplikasi Ini

**Sudut Silat** adalah aplikasi pencatatan nilai pertandingan Pencak Silat kategori Tanding. Aplikasi ini dirancang untuk membantu juri mencatat nilai secara digital, otomatis, dan langsung tersinkronisasi ke semua perangkat secara bersamaan — tanpa perlu kertas, tanpa perlu menghitung manual, dan tanpa perlu koneksi internet.

Tujuan utama aplikasi ini adalah:
- Mempercepat proses pencatatan nilai selama pertandingan berlangsung
- Mengurangi risiko kesalahan pencatatan yang bisa merugikan salah satu pihak
- Menyediakan rekaman lengkap setiap kejadian dalam pertandingan sebagai bahan evaluasi atau koreksi jika diperlukan

---

## Siapa Saja yang Terlibat?

Dalam satu pertandingan, ada **4 peran** yang menggunakan aplikasi ini masing-masing dengan perangkat (HP atau tablet) sendiri. Peran **Juri 1** dan **Juri 2** kini simetris — keduanya dapat mencatat nilai untuk sudut Merah maupun Biru, tidak lagi terikat pada satu sudut:

### 1. Juri 1
Bertugas mencatat nilai teknik yang berhasil selama pertandingan berlangsung. Berbeda dengan sistem lama, juri ini **tidak terikat pada satu sudut saja** — setiap kali ada teknik yang sah, juri memilih dulu sudut mana yang mencetak nilai (Merah atau Biru), lalu memilih jenis tekniknya. Ini mencerminkan cara kerja juri sungguhan yang mengamati seluruh pertandingan, bukan hanya satu pihak.

### 2. Juri 2
Sama dengan Juri 1 — dapat mencatat nilai untuk sudut Merah maupun Biru dalam setiap entri. Kedua juri ini mengamati pertandingan secara mandiri dan independen, kemudian sistem mencocokkan apakah keduanya sepakat atas teknik dan sudut yang sama.

### 3. Juri Pelanggaran
Bertugas mencatat setiap pelanggaran yang dilakukan oleh salah satu atlet. Juri ini yang menentukan jenis pelanggaran dan tingkat hukumannya, sehingga nilai pengurangan langsung otomatis diterapkan ke papan skor.

### 4. Operator
Bertugas mengendalikan jalannya pertandingan di aplikasi: memulai dan menghentikan timer babak, membuka fase Nilai Pencak di akhir babak, serta menerima dan memproses protes dari official atau pelatih sudut.

> **Catatan:** Papan skor digital (untuk layar proyektor atau TV) juga bisa ditampilkan dari perangkat terpisah, sehingga penonton atau juri kepala bisa memantau skor secara langsung.

---

## Bagaimana Alur Pertandingan Berjalan?

### Sebelum Babak Dimulai
Operator menekan tombol **Mulai** untuk memulai hitungan mundur babak (2 menit). Semua juri siap dengan layar masing-masing.

### Selama Babak Berlangsung

**Pencatatan Nilai Teknik**

Saat Juri Sudut Merah atau Juri Sudut Biru melihat teknik yang berhasil, ia menekan tombol sesuai teknik yang dilakukan atletnya:

| Tombol | Teknik | Nilai |
|---|---|---|
| Pukulan | Pukulan bersih tanpa tangkisan | 1 |
| Elak + Pukulan | Menghindar lalu memukul balik | 2 |
| Tendangan | Tendangan bersih tanpa tangkisan | 2 |
| Elak + Tendangan | Menghindar lalu menendang balik | 3 |
| Jatuhan | Menjatuhkan lawan ke matras | 3 |
| Elak + Jatuhan | Menghindar lalu menjatuhkan lawan | 4 |

**Alur Pencatatan Nilai (3 Langkah)**

Setiap kali juri melihat teknik yang berhasil, ia mengikuti 3 langkah sederhana:

1. **Pilih sudut** — Merah atau Biru (atlet mana yang mencetak nilai)
2. **Ada elak / tangkisan?** — Ya (+1 poin bonus) atau Tidak
3. **Pilih jenis serangan** — Pukulan, Tendangan, atau Jatuhan

**Konfirmasi Nilai (Mekanisme Kesepakatan Juri)**

Setelah satu juri selesai memilih ketiga langkah di atas, aplikasi menunggu juri *yang satunya* untuk memilih **sudut dan teknik yang sama** dalam batas waktu singkat (standar: 2 detik, dapat disesuaikan).

- Jika **kedua juri sepakat** (sudut sama + teknik sama) → nilai sah, papan skor langsung bertambah
- Jika **tidak ada kesepakatan** dalam batas waktu → nilai gugur, tidak dicatat

Mekanisme ini mencerminkan cara kerja juri sungguhan: nilai hanya sah jika lebih dari satu juri secara mandiri mengakui teknik yang sama dari atlet yang sama.

**Pencatatan Pelanggaran**

Juri Pelanggaran mencatat setiap pelanggaran dengan memilih:
1. Atlet mana yang melanggar (Merah atau Biru)
2. Tingkat pelanggaran, sesuai aturan resmi:

| Tingkat | Kondisi | Pengurangan Nilai |
|---|---|---|
| Teguran | Pelanggaran ringan pertama | Tidak ada |
| Peringatan I | Pelanggaran ringan berulang / pelanggaran berat pertama | −1 |
| Peringatan II | Pelanggaran berlanjut | −2 |
| Hukuman I | Akumulasi pelanggaran berat | −5 |
| Hukuman II / Diskualifikasi | Pelanggaran sangat berat / berulang | −10 / Gugur |

Nilai pengurangan langsung diterapkan ke papan skor secara otomatis.

### Akhir Babak — Fase Nilai Pencak

Setelah timer babak habis, Operator membuka **Fase Nilai Pencak**. Pada fase ini, setiap juri secara mandiri dan independen memberikan 1 nilai tambahan kepada salah satu atlet (atau tidak memberikan sama sekali), berdasarkan penilaian subjektif atas kualitas gerak pencak yang ditampilkan selama babak tersebut.

Setelah semua juri selesai memberikan penilaian, hasilnya otomatis ditambahkan ke skor babak, lalu pertandingan masuk ke waktu istirahat sebelum babak berikutnya.

### Penentuan Pemenang

Pertandingan terdiri dari **3 babak** (masing-masing 2 menit). Atlet yang memenangkan **2 babak berturut-turut** langsung dinyatakan menang. Jika masing-masing atlet menang 1 babak, nilai total dari semua babak dijumlahkan untuk menentukan pemenang.

---

## Bagaimana Proses Protes / Keberatan?

Jika pelatih sudut atau official merasa ada kesalahan pencatatan nilai, mereka dapat menyampaikan keberatan kepada wasit secara lisan. Wasit kemudian menginstruksikan Operator untuk memproses protes tersebut di aplikasi.

**Langkah-langkah proses protes:**

1. **Operator menghentikan timer** (jika belum berhenti)
2. **Operator memilih alasan protes**, yang bisa berupa:
   - Pilihan cepat dari daftar alasan umum (misalnya: *"Nilai tidak tercatat"*, *"Pelanggaran salah dicatat"*, dll.)
   - Pengetikan alasan secara bebas jika tidak ada yang sesuai
   - Atau sekadar notifikasi kepada semua juri bahwa ada protes yang sedang dipertimbangkan (tanpa pemungutan suara)
3. **Semua layar juri menampilkan dialog protes** secara bersamaan, berisi alasan protes dan tiga tombol pilihan:
   - 🔴 **Merah** — setuju, koreksi nilai untuk atlet merah
   - 🔵 **Biru** — setuju, koreksi nilai untuk atlet biru
   - ❌ **Tidak Sah** — tolak protes, nilai tidak berubah
4. Masing-masing juri menekan pilihan mereka secara mandiri (tanpa saling tahu pilihan juri lain)
5. **Keputusan diambil berdasarkan suara terbanyak** (minimal 2 dari 3 juri)
6. Hasil langsung muncul di semua layar dan papan skor diperbarui
7. Operator melanjutkan pertandingan

> Selama proses protes berlangsung, **semua tombol pencatatan nilai terkunci** — tidak ada nilai baru yang bisa masuk sampai protes selesai.

---

## Rekaman Pertandingan (Log)

Setiap kejadian selama pertandingan dicatat secara otomatis oleh sistem — mulai dari nilai yang masuk, nilai yang gugur, pelanggaran, protes, hingga hasil akhir. Catatan ini tersimpan sebagai file di perangkat yang menjalankan aplikasi.

**Manfaatnya:**
- Jika ada kejadian yang tidak terduga (misalnya layar juri mati sejenak, atau ada tombol yang tidak sengaja tertekan), catatan ini bisa digunakan untuk memeriksa apa yang sebenarnya terjadi
- Panitia dapat melakukan koreksi berdasarkan catatan tersebut setelah pertandingan selesai
- Catatan bersifat **tidak bisa dihapus** — koreksi dilakukan dengan menambahkan entri baru, sehingga riwayat selalu lengkap dan transparan

---

## Cara Aplikasi Dijalankan

Tidak perlu internet. Tidak perlu langganan server berbayar. Cukup:

1. **Satu laptop/komputer** dihidupkan dan menjalankan aplikasi sebagai "pusat"
2. Laptop tersebut terhubung ke jaringan Wi-Fi lokal (bisa dari hotspot HP atau router event)
3. **Setiap juri** membuka browser di HP masing-masing, lalu mengetik alamat yang diberikan (atau scan QR code yang tersedia di layar Operator)
4. Masing-masing juri memilih perannya → aplikasi siap digunakan

Tidak ada yang perlu di-*install* di HP juri. Cukup buka browser, seperti membuka website biasa.

---

## Hal-hal yang Masih Perlu Dikonfirmasi

Sebelum pengembangan dimulai, ada beberapa hal yang kami mohon konfirmasinya dari pihak panitia:

1. **Apakah proses protes sudah sesuai** dengan cara yang biasa digunakan di kejuaraan yang Anda selenggarakan? Jika ada perbedaan, mohon jelaskan alur yang biasa dipakai.

2. **Apakah Nilai Pencak** diberikan oleh semua juri (termasuk Juri Pelanggaran), atau hanya Juri Sudut Merah dan Biru?

3. **Siapa yang berwenang** untuk memulai/menghentikan timer? Apakah hanya Operator, atau juga wasit kepala?

4. **Apakah ada kebutuhan cetak** rekaman pertandingan setelah selesai, atau cukup disimpan sebagai file digital?

5. **Pelanggaran berat tertentu** seperti meninggalkan arena 3 kali atau dijatuhkan 3 kali dalam satu babak mengakibatkan kekalahan — apakah fitur pencatatan ini perlu ada di versi awal?

---

## Ringkasan Fitur yang Akan Dibangun (Versi Pertama)

| Fitur | Keterangan |
|---|---|
| Pencatatan nilai teknik | 3 langkah: pilih sudut → ada elak? → pilih serangan; tombol besar, mudah ditekan |
| Konfirmasi nilai oleh 2 juri | Nilai hanya sah jika kedua juri memilih sudut dan teknik yang sama dalam batas waktu |
| Pencatatan pelanggaran | 5 tingkat pelanggaran sesuai aturan, pengurangan otomatis |
| Nilai Pencak akhir babak | Diberikan secara mandiri oleh setiap juri di akhir tiap babak |
| Papan skor real-time | Dapat ditampilkan di layar terpisah untuk penonton |
| Protes / keberatan | Dialog pemungutan suara muncul di semua layar juri sekaligus |
| Timer babak dan istirahat | Dikontrol oleh Operator, durasi dapat disesuaikan |
| Rekaman otomatis | Setiap kejadian dicatat, tersimpan di perangkat |
| Bahasa antarmuka | Bahasa Indonesia sepenuhnya |
| Tanpa internet | Berjalan di jaringan Wi-Fi lokal, tidak perlu server berbayar |

---

*Dokumen ini merupakan gambaran awal sistem. Setiap bagian terbuka untuk diskusi dan penyesuaian sebelum pengembangan dimulai. Silakan sampaikan pertanyaan, masukan, atau koreksi kepada tim pengembang.*
