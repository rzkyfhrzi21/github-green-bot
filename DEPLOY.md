# GitHub Daily Activity

Implementasi ini berjalan di GitHub-hosted runner. Laptop tidak perlu menyala,
tidak perlu VPS, Discord bot, Personal Access Token, atau dependency aplikasi.

## Pemasangan

1. Buat repository GitHub standalone milik akun Anda, bukan fork.
2. Jadikan isi folder `backend/` sebagai root repository tersebut.
3. Pastikan workflow berada di default branch (`main`).
4. Buka **Settings > Actions > General > Workflow permissions**.
5. Pilih **Read and write permissions**, lalu simpan.
6. Buka tab **Actions > Daily Activity > Run workflow** untuk pengujian pertama.

Workflow otomatis berjalan 15 kali per hari menggunakan dua jadwal cron.
Jadwal sengaja digeser dari menit `00` untuk mengurangi risiko antrean padat.
Setiap run membuat satu file aktivitas baru sehingga menghasilkan commit nyata.
File aktivitas dirapikan berdasarkan tahun, bulan, lalu tanggal:

```text
activity/
└── 2026/
    └── 08/
        └── 2026-08-01/
            └── 09-07-00-123456.md
```

Jadwal GitHub Actions tidak dijamin tepat pada detik yang sama dan dapat
terlambat ketika antrean padat. Workflow terjadwal juga dapat dinonaktifkan
GitHub setelah repository tidak memiliki aktivitas dalam jangka waktu tertentu.

## Repository Private

Repository private tetap dapat menyumbang ke contribution graph akun Anda.
Di profil GitHub, buka **Contribution settings** lalu aktifkan **Private
contributions** jika Anda ingin jumlah kontribusi privat terlihat oleh publik.
Nama repository dan detail commit tetap tidak terlihat oleh publik.

Jadwal ini menjalankan 15 job per hari atau sekitar 450 job per 30 hari.
Repository private memakai kuota GitHub Actions sesuai paket akun Anda. Jika
kuota habis dan akun tidak memiliki metode pembayaran, workflow dapat berhenti
sampai kuota periode berikutnya tersedia. Repository public tidak dikenai biaya
untuk penggunaan runner standar GitHub-hosted.

## Agar Kontribusi Terhitung

- Repository harus standalone, bukan fork.
- Commit harus masuk ke default branch atau `gh-pages`.
- Identitas commit menggunakan alamat `noreply` milik actor GitHub yang
  menjalankan workflow.
- Akun tersebut harus memiliki akses kolaborator ke repository.
- Kontribusi dapat memerlukan waktu hingga 24 jam untuk muncul.

Jika push gagal karena branch protection, izinkan GitHub Actions melakukan push
ke branch tersebut atau gunakan repository khusus tanpa protection rule yang
mewajibkan pull request.

## Keamanan

Workflow hanya meminta izin `contents: write` dan menggunakan `GITHUB_TOKEN`
bawaan yang berlaku sementara selama job. Jangan menambahkan PAT ke file atau
remote Git.
