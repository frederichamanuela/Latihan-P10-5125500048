Semua perbaikan dikerjakan di cabang `fix/katalog`, satu commit per bug, lalu digabung ke `main`.

BUG-01 (G1): logo navbar tidak tampil
- **Reproduksi:** Buka halaman, di navbar yang muncul cuma ikon gambar rusak.
- **Hipotesis 1:** Mungkin file `logo.svg` hilang atau rusak. Ternyata bukan: di Explorer VS Code, `assets/logo.svg` ada. [cek: bisa dibuka langsung di /assets/logo.svg]
- **Hipotesis 2:** Path di `index.html` salah folder. Terbukti.
- **Bukti DevTools:** Tab Network menampilkan permintaan ke `http://127.0.0.1:5500/img/logo.svg` dengan status 404 dan Content-Type `text/html` (halaman error Live Server, bukan SVG). Folder `img/` memang tidak ada, yang ada `assets/`.
- **Perbaikan:** `index.html:13`, `img/logo.svg` diganti `assets/logo.svg` (commit `1df60fb`).
- **Uji ulang:** Setelah refresh, `logo.svg` berstatus 200 dan logo tampil di navbar.

## BUG-02 (G2): kartu tersusun satu kolom
- **Reproduksi:** Di lebar 1200 px kartu tetap menumpuk satu kolom dan cuma memakai sekitar sepertiga lebar halaman.
- **Hipotesis 1:** CSS buatan sendiri menimpa layout. Gugur: di tab Computed, `display` milik `#galeri` bernilai `block` dengan sumber *user agent stylesheet*, jadi tidak ada CSS lain yang ikut campur.
- **Hipotesis 2:** `#galeri` kekurangan class `row`. Terbukti.
- **Bukti DevTools:** Di Elements, `#galeri` hanya punya `class="g-3"`, sedangkan pembungkus filter di atasnya memakai `row g-2 mb-3`. Kartunya sudah memakai `col-12 col-sm-6 col-lg-4`, tapi kolom baru membagi lebar kalau induknya `row` (flex). Tadinya aku mencari class `row` di `#galeri` dan tidak ketemu, dan justru itulah masalahnya.
- **Perbaikan:** `index.html:34`, `class="g-3"` menjadi `class="row g-3"` (commit `8dc923c`).
- **Uji ulang:** Computed `#galeri` berubah jadi `display: flex` dari `.row`. Di 1200 px kartu berjajar 3 kolom, dan di 400 px kembali satu kolom.

## BUG-03 (G3): filter kategori tidak mengubah daftar
- **Reproduksi:** Memilih Desain di dropdown tidak mengubah apa pun, kartu tetap 6. Console menampilkan `Uncaught TypeError: Cannot read properties of null (reading 'addEventListener')` di `main.js:52:43`. Teks galatnya sempat tertutup dropdown yang masih terbuka.
- **Hipotesis 1:** Logika filter di `tampil()` salah. Gugur: galat muncul di baris 52 saat halaman dimuat, sebelum `tampil()` pernah dipanggil.
- **Hipotesis 2:** Selector di JavaScript tidak cocok dengan id di HTML. Terbukti.
- **Bukti DevTools:** HTML memakai `id="filterKategori"`, tapi `main.js:52` mencari `#filter-kategori`. Di Console, `document.querySelector('#filter-kategori')` menghasilkan `null`, sedangkan `document.querySelector('#filterKategori')` menampilkan elemen `<select>`.
- **Perbaikan:** `main.js:52`, selector diganti jadi `#filterKategori` mengikuti HTML (commit `7b620cb`).
- **Uji ulang:** Galat hilang dari Console. Memilih Desain menyisakan 2 kartu Desain, dan Semua kategori mengembalikan 6 kartu.

## BUG-04 (G4): teks jumlah karya tidak berubah
- **Reproduksi:** Memilih Animasi, Desain, atau Video menyisakan 2 kartu, tapi teksnya tetap "Menampilkan 6 dari 6 karya".
- **Hipotesis 1:** Fungsi `saring()` salah. Gugur: kartunya sudah tersaring jadi 2.
- **Hipotesis 2:** Teks dihitung dari daftar penuh, bukan hasil saringan. Terbukti.
- **Bukti DevTools:** Di tab Sources aku pasang breakpoint di `main.js:36`. Panel Scope menampilkan `hasil: (2)` dan `KARYA: (6)`, padahal baris itu memakai `KARYA.length` untuk kedua angka.
- **Perbaikan:** `main.js:36`, angka pertama diganti `hasil.length`, angka kedua tetap `KARYA.length` (commit `2957d97`).
- **Uji ulang:** Memilih Animasi menampilkan "Menampilkan 2 dari 6 karya", Semua kategori menampilkan "6 dari 6 karya".

## BUG-05 (G5): tombol Detail mati setelah mencari
- **Reproduksi:** Saat halaman baru dibuka, Detail berfungsi. Setelah mengetik "Video" di kotak pencarian, tombol Detail tidak bereaksi dan panel tetap menampilkan "Animasi Logo Himpunan" dari klik sebelumnya.
- **Hipotesis 1:** Data detail salah (`data-id` atau `KARYA.find`). Gugur: sebelum mencari datanya benar, dan setelah mencari panel sama sekali tidak berubah, artinya klik tidak diproses.
- **Hipotesis 2:** Listener klik hilang karena tombolnya dibuat ulang. Terbukti.
- **Bukti DevTools:** Di Elements > Event Listeners, tombol Detail sebelum mencari punya `click` dari `main.js:41`. Sesudah mencari, tombol hasil pencarian menampilkan "No event listeners". Sempat salah pilih elemen (`body`, lalu `div#galeri`) sehingga listener tampak kosong, jadi harus memilih tombolnya langsung. `tampil()` memakai `replaceChildren` yang membuat tombol baru, sedangkan `pasangDetail()` hanya dipanggil sekali di akhir berkas.
- **Perbaikan:** `main.js:36`, `pasangDetail();` ditambahkan di dalam `tampil()` setelah `replaceChildren`, dan panggilan yang berdiri sendiri di bagian bawah berkas dihapus agar tombol tidak punya dua listener. Indentasi `tampil()` ikut dirapikan (commit `b9a848c`).
- **Uji ulang:** Mencari "Video" lalu klik Detail menampilkan "Video Profil Laboratorium — Video, dibuat oleh Kelompok B (2026)".

## BUG-06 (G6): pencarian huruf kecil tidak menemukan karya
- **Reproduksi:** Mengetik "Animasi" menampilkan 2 kartu, tapi "animasi" tidak menampilkan satu pun.
- **Hipotesis 1:** Data judul salah. Gugur: "Animasi" menemukan 2 karya, jadi datanya ada.
- **Hipotesis 2:** Perbandingan teks membedakan huruf besar dan kecil. Terbukti.
- **Bukti DevTools:** Di Console, `'Animasi Logo Himpunan'.includes('Animasi')` bernilai `true`, sedangkan `.includes('animasi')` bernilai `false`.
- **Perbaikan:** `main.js:8`, `cari.value.trim()` menjadi `cari.value.trim().toLowerCase()`. `main.js:12`, `k.judul.includes(kata)` menjadi `k.judul.toLowerCase().includes(kata)` (commit `1ddd92c`).
- **Uji ulang:** "animasi" menampilkan 2 kartu, dan "ANIMASI" serta "Animasi" memberi hasil yang sama.

## Aset dan penggabungan
- Satu-satunya 404 di tab Network adalah `favicon.ico`. Aset `assets/logo.svg` sudah benar namanya, jadi tidak ada yang diganti. Favicon diarahkan ke logo yang ada, di commit `chore:` terpisah (`62d424f`).
- Cabang `master` diganti nama jadi `main`, lalu `fix/katalog` digabung dengan `git merge --no-ff` (commit merge `7912892`).