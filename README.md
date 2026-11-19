# log-harian-skill

Skill Claude untuk mengotomasi **Log Aktivitas Harian (PD-6)** lewat **DINTA Bot** di Slack.

Cukup simpan screenshot kerja seperti biasa. Claude yang akan:

1. Membaca screenshot hari itu dan me-rename file-nya jadi `M-D Melakukan …`
2. Menyusun draft deskripsi log dan memilih Task yang paling cocok
3. Menunggu kamu bilang **"oke"**
4. Mengirim screenshot ke DINTA Bot, membuka form lewat **Jadikan laporan**, mengisi tanggal dan Task, lalu Submit
5. Mengecek balasan bot "sukses terunggah"

Setelah itu DINTA Bot yang memetakan Task ke Inisiatif/EPIC dan menulis ke Google Sheet, seperti biasa.

---

## Yang kamu butuhkan

| Kebutuhan | Keterangan |
|---|---|
| Akun Claude berbayar (Pro, Max, Team, atau Enterprise) | Untuk memakai skill dan akses folder komputer |
| Aplikasi **Claude desktop** | Supaya Claude bisa membaca folder screenshot di komputermu |
| Ekstensi **Claude in Chrome** | Wajib, karena browser bawaan Claude tidak bisa upload file. Pasang dari [claude.ai/chrome](https://claude.ai/chrome) dan login dengan akun Claude yang sama |
| Slack ter-login di **Chrome** | Kalau selama ini kamu cuma pakai aplikasi Slack desktop, login sekali di Chrome |

---

## Cara pasang

### 1. Download skill-nya

Download folder [`log-harian`](./log-harian) dari repo ini, atau langsung ambil file [`log-harian.zip`](./log-harian.zip).

### 2. Isi konfigurasi (disarankan)

Buka `log-harian/SKILL.md`, cari bagian **Konfigurasi**, lalu ganti isinya dengan data milikmu:

```
FOLDER_SCREENSHOT : C:\Users\NamaKamu\Desktop\Kerja\2026
LINK_DM_DINTA_BOT : https://<workspace>.slack.com/archives/D0XXXXXXXXX
PLATFORM          : Murid
SQUAD             : Pembelajaran Digital
```

- **FOLDER_SCREENSHOT**: folder induk tempat screenshot. Struktur yang diharapkan: `<folder>\<Bulan>\<M-D>\`, misalnya `...\2026\November\11-2\`.
- **LINK_DM_DINTA_BOT**: buka DM dengan DINTA Bot di Slack, lalu salin link-nya. Setiap orang punya link DM yang berbeda, jadi jangan pakai punya orang lain.
- **PLATFORM / SQUAD**: sesuaikan dengan penugasanmu.

Kalau kamu lewati langkah ini, Claude akan menanyakannya di pemakaian pertama.

Setelah diedit, zip ulang folder `log-harian` (yang di-zip folder-nya, dan di dalamnya harus ada `SKILL.md`).

### 3. Upload ke Claude

Di Claude, buka **Customize → Skills**, upload file zip-nya, lalu pastikan skill-nya aktif.

---

## Cara pakai

Di Claude desktop, ketik misalnya:

- `/log-harian 11-3`
- `kerjakan log harian tanggal 3 sampai 5 November`

Di pemakaian pertama, Claude akan meminta izin akses ke folder screenshot-mu, dan kamu tinggal klik setuju.

Claude akan menampilkan tabel draft seperti ini:

| # | Deskripsi | Task | Yakin? |
|---|---|---|---|
| 1 | Melakukan diskusi mingguan dengan desainer lain… | (Task bulan ini) | ✅ |
| 2 | Melakukan diskusi tentang instrumen latihan soal di CMS | (Task bulan ini) | ⚠️ cocok ke dua Task |

Cek tabelnya, jawab yang ⚠️, lalu bilang **"oke"**. Claude akan meng-submit satu per satu dan melaporkan hasilnya.

---

## Hal yang perlu diketahui

- **Daftar Task berubah tiap bulan.** Claude selalu membaca dropdown Task terbaru dari form DINTA Bot, jadi tidak memakai daftar bulan lalu.
- **Mode aman (default):** Claude tidak akan submit apa pun sebelum kamu bilang "oke". Kalau mapping-nya sudah sering benar, kamu bisa minta **"mode cepat"**: Claude langsung submit yang yakin dan hanya bertanya untuk yang meragukan.
- **Laporan tetap atas namamu.** Claude hanya menyusun log dari screenshot yang memang ada, tidak mengarang aktivitas. Tetap cek draft-nya.
- **Claude tidak pernah menulis ke Google Sheet.** Itu tugas DINTA Bot.
- **Claude tidak menyentuh password.** Login Slack selalu kamu lakukan sendiri.
- **Pengecekan tanggal:** kalau screenshot di folder `11-2` ternyata diambil di hari lain (dilihat dari waktu file atau jam di layar), Claude akan menandainya dan bertanya.

## Kalau ada masalah

| Masalah | Solusi |
|---|---|
| "Browser extension is not connected" | Pastikan Chrome terbuka, ekstensi Claude aktif, dan login dengan akun Claude yang sama |
| Slack minta login di tab Chrome | Login sendiri di tab itu, lalu bilang ke Claude untuk lanjut |
| Form DINTA Bot tidak muncul | Form hanya muncul lewat ⋮ → Connect to apps → **Jadikan laporan**. Claude akan mencoba lagi |
| Upload gagal | Claude perlu akses ke folder screenshot. Setujui permintaan akses foldernya |

---

## Struktur repo

```
log-harian-skill/
├── README.md            ← panduan ini (untuk manusia)
├── log-harian/
│   └── SKILL.md         ← instruksi untuk Claude
└── log-harian.zip       ← siap upload ke Claude
```

Dibuat dengan bantuan Claude.
