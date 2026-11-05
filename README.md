# Konten online Kucing dan Ikan (update tanpa build ulang APK)

Game membaca `config.json` dari internet setiap kali dibuka, menyimpannya di HP, lalu memakainya.
Kalau HP sedang offline, game memakai salinan terakhir. Ubah file ini → semua pemain mendapat
konten baru saat membuka game berikutnya. Tidak perlu build APK atau unggah ulang ke Play Store.

## Pasang sekali (GitHub Pages, gratis)
1. Buat repositori **publik** baru di GitHub, misalnya `kucing-dan-ikan-konten`.
2. Unggah `config.json` dari folder ini ke repositori itu.
3. Buka **Settings → Pages**, pilih *Deploy from a branch*, branch `main`, folder `/ (root)`, lalu simpan.
4. Setelah 1–2 menit file bisa dibuka di `https://NAMA-ANDA.github.io/kucing-dan-ikan-konten/config.json`.
5. Di `www/index.html` cari `var CONTENT_URL='';`, isi dengan alamat di atas, lalu build APK **sekali** lagi.
   Mulai saat itu, cukup edit `config.json` di GitHub.

## Isi config.json (semua bagian boleh dihapus)
| Bagian | Fungsi |
|---|---|
| `news` | Pengumuman yang muncul sekali per `id`. Ganti `id` untuk pengumuman baru. |
| `event` | Event bertanggal (`from`/`to` format `YYYY-MM-DD`). `mult` = pengali koin dari level (maks 5). |
| `season` | (opsional) Paksa tema musim: `{"id":"ramadan","from":"2027-02-07","to":"2027-03-09"}`. `id`: `halloween`, `natal`, `tahunbaru`, `ramadan`, `lebaran`, `merdeka`, atau `none` untuk mematikan. Tanpa ini, tema muncul otomatis sesuai tanggal (Ramadan/Idulfitri memakai perkiraan kalender 2026–2030). |
| `skins` | Skin tambahan di toko. `e` emoji, `f` filter CSS untuk 2D, `c`/`d`/`b`/`r` warna 3D (hex tanpa `#`: badan, belang, perut, kalung). |
| `prices` | Ubah harga skin (`skins.<id>`) dan power-up (`pu.shield`, `pu.bomb`, `pu.skip`). |
| `levels` | Level bonus (dunia ✨ Bonus Online, maks 60). Format sama dengan `LEVELS` di game; `par` = tarikan minimum. |
| `daily` | (opsional) Paksa level tantangan untuk tanggal tertentu: `{"2026-12-25": {g,p,k,par}}`. Tanpa ini, level dibuat otomatis dari tanggal — sama untuk semua pemain. |
| `leaderboard.url` | Alamat Firebase Realtime Database untuk papan peringkat Endless (lihat bawah). Kosong = nonaktif. |
| `latest` | Kalau `version` > versi di aplikasi (`APP_VERSION`), pemain diajak update di Play Store (`url`). |

Kode sel level: `H` kucing, `F` ikan, `L` api, `W` air, `R` batu, `B` bata, `#` dinding, `>` `<` ban berjalan, `.` kosong.
`p` = daftar pin `[baris,kolom]`, `k` = kunci `[barisPin,kolomPin,barisSyarat,kolomSyarat]`.
Level yang tidak valid diabaikan otomatis, jadi salah ketik tidak membuat game rusak.
Pastikan level bisa diselesaikan: cara termudah, minta Claude membuatkan level dengan pembuat level bawaan game.

## Papan peringkat (opsional, Firebase gratis)
1. Buka console.firebase.google.com → buat proyek → **Realtime Database** → *Create database*.
2. Di tab **Rules**, tempel aturan ini (membatasi isi data) lalu *Publish*:
```json
{
  "rules": {
    "endless": {
      ".read": true,
      ".indexOn": ["s"],
      "$pid": {
        ".write": true,
        ".validate": "newData.hasChildren(['n','s','t']) && newData.child('n').isString() && newData.child('n').val().length <= 16 && newData.child('s').isNumber() && newData.child('s').val() <= 500 && (!data.exists() || newData.child('s').val() >= data.child('s').val())"
      }
    }
  }
}
```
3. Salin alamat database (mis. `https://nama-proyek-default-rtdb.asia-southeast1.firebasedatabase.app`) ke `leaderboard.url`.

Catatan: papan peringkat tanpa login seperti ini bisa dicurangi oleh orang yang paham teknis.
Untuk game santai biasanya cukup. Kalau nanti butuh yang lebih aman, tambahkan Firebase Authentication.
