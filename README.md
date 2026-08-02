# GitHub Green Bot

Otomatisasi aktivitas GitHub untuk akun **[@rzkyfhrzi21](https://github.com/rzkyfhrzi21)** menggunakan GitHub Actions.

Workflow berjalan sepenuhnya pada GitHub-hosted runner. Laptop, VPS, bot Discord,
Personal Access Token, dan aplikasi yang hidup 24 jam tidak diperlukan.

## Fitur

- Berjalan otomatis melalui GitHub Actions.
- Mendukung repository private.
- Menggunakan `GITHUB_TOKEN` sementara dari GitHub.
- Membuat commit dengan identitas `noreply` akun yang menjalankan workflow.
- Menyimpan log berdasarkan tahun, bulan, dan tanggal.
- Dapat dijalankan manual melalui `workflow_dispatch`.
- Mencegah beberapa job berjalan bersamaan melalui `concurrency`.
- Tidak memiliki dependency atau proses instalasi aplikasi.

## Cara Kerja

Setiap workflow run melakukan proses berikut:

1. Checkout default branch repository.
2. Membaca tanggal dan waktu menggunakan zona `Asia/Jakarta`.
3. Membuat satu file Markdown aktivitas baru.
4. Mengatur nama dan email Git author berdasarkan actor GitHub.
5. Membuat satu commit nyata.
6. Push commit ke default branch menggunakan `GITHUB_TOKEN`.

Alur singkat:

```text
GitHub schedule/manual trigger
        |
        v
GitHub-hosted Ubuntu runner
        |
        v
Create activity file -> Commit -> Push
        |
        v
GitHub contribution graph
```

## Struktur Repository

```text
.
|-- .github/
|   `-- workflows/
|       `-- daily-activity.yml
|-- activity/
|   `-- YYYY/
|       `-- MM/
|           `-- YYYY-MM-DD/
|               `-- HH-MM-SS-RUN_ID.md
|-- DEPLOY.md
`-- README.md
```

Contoh file yang dibuat:

```text
activity/2026/08/2026-08-02/13-50-12-123456789.md
```

## Jadwal Aktif

Workflow berjalan sekali setiap hari:

```yaml
schedule:
  - cron: "0 18 * * *"
```

GitHub Actions menggunakan UTC. Jadwal tersebut setara dengan satu run pada
`01:00 WIB`. Satu run menghasilkan 15 commit berturut-turut.

Eksekusi scheduled workflow tidak dijamin tepat pada detik yang sama dan dapat
terlambat beberapa menit ketika antrean GitHub Actions sedang padat.

### Jadwal Produksi yang Disarankan

Untuk membuat 15 commit dalam satu job setiap hari pukul `01:00 WIB`, gunakan:

```yaml
schedule:
  - cron: "0 18 * * *"
```

Karena `01:00 WIB` sama dengan `18:00 UTC` pada hari sebelumnya. Logika workflow
perlu menggunakan loop 15 commit jika memilih satu run per hari.

## Konfigurasi GitHub

1. Buka **Settings > Actions > General**.
2. Pada **Actions permissions**, izinkan workflow yang digunakan repository.
3. Pada **Workflow permissions**, pilih **Read and write permissions**.
4. Simpan pengaturan.
5. Pastikan workflow berada pada default branch repository.
6. Buka **Actions > Daily Activity** untuk menjalankan tes manual.

Jika branch memiliki protection rule yang mewajibkan pull request, izinkan
GitHub Actions melakukan push atau gunakan repository khusus tanpa aturan tersebut.

## Repository Private dan Grafik Kontribusi

Repository private tetap dapat menghasilkan kotak hijau pada profil. Agar jumlah
kontribusi private terlihat oleh pengunjung yang tidak login:

1. Buka halaman profil GitHub.
2. Klik **Contribution settings** di area grafik kontribusi.
3. Aktifkan **Private contributions**.

Pengunjung hanya dapat melihat jumlah kontribusi private. Nama repository, isi
file, pesan commit, dan detail perubahan tetap tidak ditampilkan kepada publik.

Agar commit dihitung sebagai kontribusi:

- Repository harus standalone, bukan fork.
- Commit harus berada pada default branch atau `gh-pages`.
- Email Git author harus terhubung dengan akun GitHub.
- Actor harus menjadi pemilik atau collaborator repository.
- Kontribusi dapat membutuhkan waktu hingga 24 jam untuk muncul.

Workflow menggunakan email resmi GitHub berikut secara otomatis:

```text
ACTOR_ID+ACTOR@users.noreply.github.com
```

## Menjalankan Secara Manual

1. Buka tab **Actions** pada repository.
2. Pilih workflow **Daily Activity**.
3. Klik **Run workflow**.
4. Pilih default branch.
5. Klik **Run workflow** untuk mengonfirmasi.

Setelah selesai, periksa commit dan folder `activity/` pada repository.

## Kuota GitHub Actions

GitHub Actions pada repository private menggunakan kuota sesuai paket akun.
Satu job sederhana biasanya selesai dalam waktu singkat, tetapi perhitungan billing
mengikuti kebijakan GitHub dan jenis runner yang digunakan.

Rekomendasi penggunaan:

| Pola | Run per hari | Perkiraan run per 30 hari |
|---|---:|---:|
| Satu job berisi 15 commit | 1 | 30 |
| Satu commit per run | 15 | 450 |
| Setiap 30 menit | 48 | 1.440 |

Satu job yang menghasilkan 15 commit adalah opsi paling hemat jika tujuan hanya
mencapai 15 kontribusi harian.

## Keamanan

- Tidak menyimpan PAT di source code.
- Tidak memasukkan token ke URL remote.
- Hanya meminta permission `contents: write`.
- Menggunakan `GITHUB_TOKEN` yang berlaku sementara selama workflow run.
- Tidak menggunakan service pihak ketiga.

Jangan pernah menulis token dengan format berikut ke repository:

```text
https://TOKEN@github.com/OWNER/REPOSITORY.git
```

Gunakan remote HTTPS biasa:

```text
https://github.com/rzkyfhrzi21/github-green-bot.git
```

## Troubleshooting

### Workflow tidak muncul

- Pastikan file berada di `.github/workflows/daily-activity.yml`.
- Pastikan file sudah berada pada default branch.
- Pastikan GitHub Actions diizinkan pada repository.

### Push ditolak

- Aktifkan **Read and write permissions**.
- Periksa branch protection rule.
- Pastikan `permissions: contents: write` tersedia pada workflow.

### Scheduled workflow terlambat

Scheduled workflow tidak menjamin eksekusi tepat pada detik yang ditentukan.
GitHub dapat menunda atau melewatkan job saat antrean sedang padat. Gunakan
`workflow_dispatch` untuk pengujian yang harus segera dijalankan.

### Kotak kontribusi belum hijau

- Tunggu hingga 24 jam.
- Pastikan private contributions sudah diaktifkan.
- Pastikan commit berada pada default branch.
- Periksa email author pada halaman commit.
- Pastikan repository bukan fork.

### Scheduled workflow berhenti

GitHub dapat menonaktifkan scheduled workflow pada repository yang lama tidak
memiliki aktivitas. Aktifkan kembali workflow melalui tab **Actions** jika terjadi.

## Verifikasi Lokal

```cmd
git status
git log --oneline -20
git remote -v
```

Untuk mengambil hasil workflow terbaru:

```cmd
git pull origin master
```

## Catatan

Grafik kontribusi sebaiknya mencerminkan aktivitas yang bermakna. Workflow ini
ditujukan sebagai eksperimen GitHub Actions dan pencatatan aktivitas otomatis.
