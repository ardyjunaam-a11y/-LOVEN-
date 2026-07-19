# LOVEN — Cara Pakai (tanpa database, tanpa Supabase)

Sekarang cuma **1 file: `index.html`**. Gak ada login, gak ada Supabase, gak ada setup SQL. Tinggal buka filenya, edit 3 bagian di bawah, upload ke hosting mana aja (atau buka langsung di browser).

Cara kerjanya: customer pilih kue → isi nama/WA/alamat → klik "Kirim Pesanan via WhatsApp" → otomatis kebuka chat WhatsApp ke nomormu berisi detail pesanan. Bukti bayar tinggal customer foto & kirim di chat itu juga.

## Yang perlu diedit

Buka `index.html`, cari bagian ini paling atas di dalam `<script>`:

```js
const ADMIN_WA = "6281234567890";
```
Ganti dengan nomor WhatsApp kamu sendiri. Format: kode negara `62` + nomor tanpa angka `0` di depan.
Contoh: nomor `0812-3456-7890` → ditulis `"6281234567890"`.

```js
const QRIS_IMAGE_URL = "";
const INFO_PEMBAYARAN = "Transfer ke BCA 1234567890 a.n. Nama Kamu";
```
`QRIS_IMAGE_URL` diisi link gambar QRIS kalau ada (boleh dikosongin). `INFO_PEMBAYARAN` diisi info rekening/e-wallet kamu.

```js
const PRODUK = [
  { id: 1, nama: "Red Velvet Slice", harga: 25000, deskripsi: "...", gambar: "https://..." },
  ...
];
```
Ini daftar menu kamu. Mau tambah kue baru, tinggal copy satu baris `{ id: ..., nama: ..., harga: ..., deskripsi: ..., gambar: ... }`, ganti isinya, dan pastikan `id` beda-beda tiap produk. Mau hapus produk, tinggal hapus barisnya.

`gambar` diisi link foto kue (bisa upload ke Google Drive/Imgur lalu pakai link-nya, atau apa saja yang menghasilkan link gambar langsung).

## Setelah diedit

Simpan file, lalu tinggal buka `index.html` di browser atau upload ke hosting statis apa saja (Netlify, GitHub Pages, dll). Selesai — gak ada langkah lain.
