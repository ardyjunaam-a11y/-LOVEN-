# Setup Database Supabase — LOVEN

Versi ini pakai **1 file HTML** (`index.html`) untuk toko sekaligus admin. Yang menentukan tampilan admin vs customer adalah **email login Google**: kalau emailnya `ardyjunaam@gmail.com` → masuk ke Dashboard Admin, email lain → masuk ke halaman toko sebagai customer/member.

Jalankan semua query ini di **Supabase Dashboard → SQL Editor**, satu per satu (atau sekaligus).

## 1. Update tabel `pesanan` (tambah kolom baru)

```sql
alter table pesanan
  add column if not exists user_id uuid references auth.users(id),
  add column if not exists email_pembeli text,
  add column if not exists bukti_bayar_url text;
```

## 2. Buat tabel `pengaturan` (untuk QRIS & info pembayaran)

```sql
create table if not exists pengaturan (
  id int primary key,
  qris_url text,
  info_pembayaran text
);

insert into pengaturan (id, qris_url, info_pembayaran)
values (1, null, null)
on conflict (id) do nothing;
```

## 3. Aktifkan Row Level Security + policy

```sql
-- PRODUK: semua orang boleh lihat, hanya admin (lewat dashboard admin) yang insert/hapus
alter table produk enable row level security;

create policy "produk_select_public" on produk
  for select using (true);

create policy "produk_admin_all" on produk
  for all using (auth.jwt() ->> 'email' = 'ardyjunaam@gmail.com')
  with check (auth.jwt() ->> 'email' = 'ardyjunaam@gmail.com');

-- PESANAN: customer hanya boleh lihat & insert pesanan miliknya, admin boleh lihat & update semua
alter table pesanan enable row level security;

create policy "pesanan_insert_own" on pesanan
  for insert with check (auth.uid() = user_id);

create policy "pesanan_select_own" on pesanan
  for select using (auth.uid() = user_id);

create policy "pesanan_admin_all" on pesanan
  for all using (auth.jwt() ->> 'email' = 'ardyjunaam@gmail.com')
  with check (auth.jwt() ->> 'email' = 'ardyjunaam@gmail.com');

-- PENGATURAN: semua boleh baca (untuk halaman checkout), hanya admin boleh ubah
alter table pengaturan enable row level security;

create policy "pengaturan_select_public" on pengaturan
  for select using (true);

create policy "pengaturan_admin_write" on pengaturan
  for all using (auth.jwt() ->> 'email' = 'ardyjunaam@gmail.com')
  with check (auth.jwt() ->> 'email' = 'ardyjunaam@gmail.com');
```

> ⚠️ Kalau email admin kamu bukan `ardyjunaam@gmail.com`, ganti di **setiap** query di atas, dan juga di variabel `ADMIN_EMAIL` pada file `index.html`.

## 4. Buat Storage bucket untuk bukti pembayaran

Di **Supabase Dashboard → Storage**:
1. Klik **New bucket**, beri nama persis: `bukti-bayar`
2. Set **Public bucket** = ON (supaya admin bisa lihat gambarnya langsung)
3. Buka tab **Policies** bucket tersebut, tambahkan policy:

```sql
create policy "bukti_bayar_upload" on storage.objects
  for insert with check (bucket_id = 'bukti-bayar' and auth.role() = 'authenticated');

create policy "bukti_bayar_read" on storage.objects
  for select using (bucket_id = 'bukti-bayar');
```

## 5. Aktifkan Google OAuth (jika belum)

Di **Authentication → Providers → Google**, aktifkan dan isi Client ID/Secret dari Google Cloud Console. Redirect URL Supabase perlu ditambahkan di Google Cloud Console → Authorized redirect URIs.

## 6. Isi konfigurasi di `index.html`

Buka `index.html`, cari bagian ini di dalam `<script>` paling bawah, lalu ganti:
```js
const SUPABASE_URL = "URL_SUPABASE";
const SUPABASE_ANON_KEY = "ANON_KEY_SUPABASE";
const ADMIN_EMAIL = "ardyjunaam@gmail.com";
```
`SUPABASE_URL` dan `SUPABASE_ANON_KEY` diisi dari Supabase Settings → API. `ADMIN_EMAIL` biarkan seperti itu kalau memang email admin kamu `ardyjunaam@gmail.com`.

> Kalau bagian ini masih placeholder (belum diisi), halaman akan menampilkan pesan "Supabase Belum Dikonfigurasi" saat dibuka — bukan loading selamanya seperti sebelumnya. Setelah diisi dan disimpan, refresh halaman.

---

### Ringkasan alur

Sekarang **hanya ada 1 file: `index.html`**. Semua orang wajib login Google dulu sebelum bisa mengakses apa pun (tidak ada opsi daftar email/password lagi — login Google sekaligus otomatis mendaftarkan akun sebagai member).

- **Email = `ardyjunaam@gmail.com`** → otomatis diarahkan ke **Dashboard Admin**: tab Produk (kelola menu), tab Pesanan (lihat bukti bayar & ubah status: Diproses/Selesai/Dibatalkan), tab Pengaturan (atur gambar QRIS & info rekening).
- **Email lain** → diarahkan ke **halaman toko**: lihat produk, belanja, checkout, scan QRIS/transfer, upload bukti bayar, dan cek status di "Pesanan Saya". Order otomatis tersimpan dengan status **"Menunggu Konfirmasi"** sampai dicek admin.
