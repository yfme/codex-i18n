<h1 align="center">OpenAI Codex CLI</h1>
<p align="center">Agen pemrograman ringan yang berjalan di terminal Anda</p>

<p align="center"><code>npm i -g @openai/codex</code></p>

[English](./README.md) | [简体中文](./README.zh-CN.md) | [Español](./README.es.md) | [Português](./README.pt.md) | [Русский](./README.ru.md) | [Français](./README.fr.md) | [Deutsch](./README.de.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md) | [繁體中文](./README.zh-TW.md) | **Bahasa Indonesia** | [Italiano](./README.it.md)

![Codex demo GIF menggunakan: codex "explain this codebase to me"](./.github/demo.gif)

---

<details>
<summary><strong>Daftar Isi</strong></summary>

- [Penafian Teknologi Eksperimental](#penafian-teknologi-eksperimental)
- [Memulai Cepat](#memulai-cepat)
- [Mengapa Codex?](#mengapa-codex)
- [Model Keamanan & Izin](#model-keamanan-dan-izin)
  - [Detail sandbox platform](#detail-sandbox-platform)
- [Persyaratan Sistem](#persyaratan-sistem)
- [Referensi CLI](#referensi-cli)
- [Memori & Dokumen Proyek](#memori-dan-dokumen-proyek)
- [Mode Non-interaktif / CI](#mode-non-interaktif--ci)
- [Resep](#resep)
- [Instalasi](#instalasi)
- [Konfigurasi](#konfigurasi)
- [FAQ](#faq)
- [Peluang Pendanaan](#peluang-pendanaan)
- [Kontribusi](#kontribusi)
  - [Alur kerja pengembangan](#alur-kerja-pengembangan)
  - [Menulis perubahan kode berdampak tinggi](#menulis-perubahan-kode-berdampak-tinggi)
  - [Membuka pull request](#membuka-pull-request)
  - [Proses peninjauan](#proses-peninjauan)
  - [Nilai komunitas](#nilai-komunitas)
  - [Mendapatkan bantuan](#mendapatkan-bantuan)
  - [Perjanjian Lisensi Kontributor (CLA)](#perjanjian-lisensi-kontributor-cla)
    - [Perbaikan cepat](#perbaikan-cepat)
  - [Merilis `codex`](#merilis-codex)
- [Keamanan & AI Bertanggung Jawab](#keamanan-dan-ai-bertanggung-jawab)
- [Lisensi](#lisensi)

</details>

---

## Penafian Teknologi Eksperimental

Codex CLI adalah proyek eksperimental yang sedang dalam pengembangan aktif. Proyek ini belum stabil, mungkin mengandung bug, fitur yang belum lengkap, atau mengalami perubahan besar. Kami membangunnya secara terbuka bersama komunitas dan menyambut:

- Laporan bug
- Permintaan fitur
- Pull request
- Semangat positif

Bantu kami meningkatkan dengan mengajukan isu atau mengirimkan PR (lihat bagian di bawah tentang cara berkontribusi)!

## Memulai Cepat

Instal secara global:

```shell
npm install -g @openai/codex
```

Selanjutnya, atur kunci API OpenAI Anda sebagai variabel lingkungan:

```shell
export OPENAI_API_KEY="your-api-key-here"
```

> **Catatan:** Perintah ini hanya mengatur kunci untuk sesi terminal saat ini. Untuk membuatnya permanen, tambahkan baris `export` ke file konfigurasi shell Anda (misalnya, `~/.zshrc`).

Jalankan secara interaktif:

```shell
codex
```

Atau, jalankan dengan prompt sebagai input (dan opsional dalam mode `Full Auto`):

```shell
codex "jelaskan kode ini untuk saya"
```

```shell
codex --approval-mode full-auto "buat aplikasi daftar tugas paling canggih"
```

Selesai – Codex akan membuat kerangka file, menjalankannya di dalam sandbox, menginstal dependensi yang hilang, dan menunjukkan hasil secara langsung. Setujui perubahan, dan perubahan tersebut akan di-commit ke direktori kerja Anda.

---

## Mengapa Codex?

Codex CLI dibuat untuk pengembang yang sudah **hidup di terminal** dan menginginkan kemampuan penalaran setara ChatGPT **plus** kekuatan untuk benar-benar menjalankan kode, memanipulasi file, dan melakukan iterasi – semuanya di bawah kontrol versi. Singkatnya, ini adalah _pengembangan berbasis obrolan_ yang memahami dan mengeksekusi repositori Anda.

- **Tanpa pengaturan** — bawa kunci API OpenAI Anda dan langsung berfungsi!
- **Persetujuan otomatis penuh, namun aman** dengan menjalankan tanpa jaringan dan direktori dalam sandbox
- **Multimodal** — masukkan tangkapan layar atau diagram untuk mengimplementasikan fitur ✨

Dan ini **sepenuhnya open-source**, sehingga Anda bisa melihat dan berkontribusi pada perkembangannya!

---

## Model Keamanan & Izin

Codex memungkinkan Anda menentukan _seberapa banyak otonomi_ yang diterima agen dan kebijakan persetujuan otomatis melalui flag `--approval-mode` (atau prompt onboarding interaktif):

| Mode                      | Apa yang bisa dilakukan agen tanpa bertanya            | Masih memerlukan persetujuan                                         |
| ------------------------- | ----------------------------------------------- | --------------------------------------------------------------- |
| **Sarankan** <br>(default) | • Membaca semua file di repositori                     | • **Semua** penulisan/patch file <br>• **Semua** perintah shell/Bash |
| **Edit Otomatis**             | • Membaca **dan** menerapkan penulisan patch ke file      | • **Semua** perintah shell/Bash                                   |
| **Otomatis Penuh**             | • Membaca/menulis file <br>• Menjalankan perintah shell | —                                                               |

Dalam mode **Otomatis Penuh**, setiap perintah dijalankan dengan **jaringan dinonaktifkan** dan dibatasi pada direktori kerja saat ini (ditambah file sementara) untuk pertahanan mendalam. Codex juga akan menampilkan peringatan/konfirmasi jika Anda memulai dalam mode **edit otomatis** atau **otomatis penuh** sementara direktori _tidak_ dilacak oleh Git, sehingga Anda selalu memiliki jaring pengaman.

Segera hadir: Anda akan dapat mengizinkan perintah tertentu untuk dieksekusi secara otomatis dengan jaringan diaktifkan, setelah kami yakin dengan pengamanan tambahan.

### Detail sandbox platform

Mekanisme pengerasan yang digunakan Codex tergantung pada OS Anda:

- **macOS 12+** – perintah dibungkus dengan **Apple Seatbelt** (`sandbox-exec`).

  - Semuanya ditempatkan dalam penjara hanya-baca kecuali sekumpulan kecil akar yang dapat ditulis (`$PWD`, `$TMPDIR`, `~/.codex`, dll.).
  - Jaringan keluar _sepenuhnya diblokir_ secara default – bahkan jika proses anak mencoba melakukan `curl` ke suatu tempat, itu akan gagal.

- **Linux** – kami merekomendasikan menggunakan Docker untuk sandbox, di mana Codex meluncurkan dirinya sendiri di dalam **gambar kontainer minimal** dan memasang repositori Anda dengan _baca/tulis_ di jalur yang sama. Skrip firewall khusus `iptables`/`ipset` menolak semua lalu lintas keluar kecuali API OpenAI. Ini memberikan Anda eksekusi yang deterministik dan dapat direproduksi tanpa memerlukan root di host. Anda dapat membaca lebih lanjut di [`run_in_container.sh`](./codex-cli/scripts/run_in_container.sh).

Kedua pendekatan ini _transparan_ untuk penggunaan sehari-hari – Anda tetap menjalankan `codex` dari root repositori Anda dan menyetujui/menolak langkah-langkah seperti biasa.

---

## Persyaratan Sistem

| Persyaratan                 | Detail                                                         |
| --------------------------- | --------------------------------------------------------------- |
| Sistem operasi           | macOS 12+, Ubuntu 20.04+/Debian 10+, atau Windows 11 **via WSL2** |
| Node.js                     | **22 atau lebih baru** (LTS direkomendasikan)                               |
| Git (opsional, direkomendasikan) | 2.23+ untuk helper PR bawaan                                   |
| RAM                         | Minimum 4 GB (disarankan 8 GB)                                 |

> Jangan pernah menjalankan `sudo npm install -g`; perbaiki izin npm sebagai gantinya.

---

## Referensi CLI

| Perintah                              | Tujuan                             | Contoh                              |
| ------------------------------------ | ----------------------------------- | ------------------------------------ |
| `codex`                              | REPL interaktif                    | `codex`                              |
| `codex "…"`                          | Prompt awal untuk REPL interaktif | `codex "perbaiki kesalahan lint"`            |
| `codex -q "…"`                       | Mode non-interaktif "senyap"        | `codex -q --json "jelaskan utils.ts"` |
| `codex completion <bash\|zsh\|fish>` | Cetak skrip penyelesaian shell       | `codex completion bash`              |

Flag utama: `--model/-m`, `--approval-mode/-a`, dan `--quiet/-q`.

---

## Memori & Dokumen Proyek

Codex menggabungkan instruksi Markdown dalam urutan berikut:

1. `~/.codex/instructions.md` – panduan global pribadi
2. `codex.md` di root repositori – catatan proyek bersama
3. `codex.md` di direktori kerja saat ini – spesifik sub-paket

Nonaktifkan dengan `--no-project-doc` atau `CODEX_DISABLE_PROJECT_DOC=1`.

---

## Mode Non-interaktif / CI

Jalankan Codex tanpa antarmuka di pipeline. Contoh langkah GitHub Action:

```yaml
- name: Perbarui changelog via Codex
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex -a auto-edit --quiet "perbarui CHANGELOG untuk rilis berikutnya"
```

Atur `CODEX_QUIET_MODE=1` untuk membungkam kebisingan antarmuka interaktif.

---

## Resep

Berikut adalah beberapa contoh singkat yang dapat Anda salin dan tempel. Ganti teks dalam tanda kutip dengan tugas Anda sendiri. Lihat [panduan prompt](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md) untuk tips dan pola penggunaan lebih lanjut.

| ✨  | Apa yang Anda ketik                                                                   | Apa yang terjadi                                                               |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "Refaktor komponen Dashboard ke React Hooks"`                       | Codex menulis ulang komponen kelas, menjalankan `npm test`, dan menunjukkan perbedaan.   |
| 2   | `codex "Hasilkan migrasi SQL untuk menambahkan tabel pengguna"`                      | Menyimpulkan ORM Anda, membuat file migrasi, dan menjalankannya di DB sandbox. |
| 3   | `codex "Tulis tes unit untuk utils/date.ts"`                                    | Menghasilkan tes, menjalankannya, dan mengulang hingga lulus.              |
| 4   | `codex "Ubah nama massal *.jpeg → *.jpg dengan git mv"`                                | Mengubah nama file dengan aman dan memperbarui impor/penggunaan.                           |
| 5   | `codex "Jelaskan apa yang dilakukan regex ini: ^(?=.*[A-Z]).{8,}$"`                      | Mengeluarkan penjelasan langkah demi langkah yang mudah dipahami manusia.                                  |
| 6   | `codex "Tinjau repositori ini dengan cermat, dan usulkan 3 PR berdampak tinggi yang terdefinisi dengan baik"` | Menyarankan PR berdampak di codebase saat ini.                            |
| 7   | `codex "Cari kerentanan dan buat laporan tinjauan keamanan"`          | Menemukan dan menjelaskan bug keamanan.                                          |

---

## Instalasi

<details open>
<summary><strong>Dari npm (Direkomendasikan)</strong></summary>

```bash
npm install -g @openai/codex
# atau
yarn global add @openai/codex
```

</details>

<details>
<summary><strong>Bangun dari sumber</strong></summary>

```bash
# Klon repositori dan navigasikan ke paket CLI
git clone https://github.com/openai/codex.git
cd codex/codex-cli

# Instal dependensi dan bangun
npm install
npm run build

# Dapatkan penggunaan dan opsi
node ./dist/cli.js --help

# Jalankan CLI yang dibangun secara lokal langsung
node ./dist/cli.js

# Atau tautkan perintah secara global untuk kenyamanan
npm link
```

</details>

---

## Konfigurasi

Codex mencari file konfigurasi di **`~/.codex/`**.

```yaml
# ~/.codex/config.yaml
model: o4-mini # Model default
fullAutoErrorMode: ask-user # atau ignore-and-continue
```

Anda juga dapat mendefinisikan instruksi khusus:

```yaml
# ~/.codex/instructions.md
- Selalu tanggapi dengan emoji
- Hanya gunakan perintah git jika saya menyebutkannya secara eksplisit
```

---

## FAQ

<details>
<summary>OpenAI merilis model bernama Codex pada tahun 2021 - apakah ini terkait?</summary>

Pada tahun 2021, OpenAI merilis Codex, sistem AI yang dirancang untuk menghasilkan kode dari prompt bahasa alami. Model Codex asli tersebut telah dihentikan per Maret 2023 dan terpisah dari alat CLI ini.

</details>

<details>
<summary>Bagaimana cara mencegah Codex menyentuh repositori saya?</summary>

Codex selalu berjalan di **sandbox terlebih dahulu**. Jika perintah atau perubahan file yang diusulkan terlihat mencurigakan, Anda cukup menjawab **n** saat diminta dan tidak ada yang terjadi pada pohon kerja Anda.

</details>

<details>
<summary>Apakah ini berfungsi di Windows?</summary>

Tidak langsung. Ini memerlukan [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install) – Codex telah diuji pada macOS dan Linux dengan Node ≥ 22.

</details>

<details>
<summary>Model mana yang didukung?</summary>

Setiap model yang tersedia dengan [Responses API](https://platform.openai.com/docs/api-reference/responses). Default adalah `o4-mini`, tetapi Anda dapat meneruskan `--model gpt-4o` atau mengatur `model: gpt-4o` di file konfigurasi untuk menggantinya.

</details>

---

## Peluang Pendanaan

Kami sangat antusias untuk meluncurkan **inisiatif senilai $1 juta** untuk mendukung proyek open-source yang menggunakan Codex CLI dan model OpenAI lainnya.

- Hibah diberikan dalam kelipatan **$25,000** kredit API.
- Aplikasi ditinjau **secara berkelanjutan**.

**Tertarik? [Ajukan di sini](https://openai.com/form/codex-open-source-fund/).**

---

## Kontribusi

Proyek ini sedang dalam pengembangan aktif dan kode kemungkinan akan berubah secara signifikan. Kami akan memperbarui pesan ini setelah selesai!

Secara lebih luas, kami menyambut kontribusi – baik Anda membuka pull request pertama Anda atau Anda adalah pengelola berpengalaman. Namun, kami peduli pada keandalan dan pemeliharaan jangka panjang, sehingga standar untuk menggabungkan kode sengaja **tinggi**. Panduan di bawah ini menjelaskan apa yang dimaksud dengan "kualitas tinggi" dalam praktik dan harus membuat prosesnya transparan dan ramah.

### Alur kerja pengembangan

- Buat _cabang topik_ dari `main` – misalnya, `feat/interactive-prompt`.
- Jaga perubahan Anda tetap fokus. Beberapa perbaikan yang tidak terkait harus dibuka sebagai PR terpisah.
- Gunakan `npm run test:watch` selama pengembangan untuk umpan balik super cepat.
- Kami menggunakan **Vitest** untuk tes unit, **ESLint** + **Prettier** untuk gaya, dan **TypeScript** untuk pemeriksaan tipe.
- Sebelum push, jalankan seluruh suite tes/tipe/lint:

  ```bash
  npm test && npm run lint && npm run typecheck
  ```

- Jika Anda **belum** menandatangani Perjanjian Lisensi Kontributor (CLA), tambahkan komentar PR yang berisi teks persis:

  ```text
  I have read the CLA Document and I hereby sign the CLA
  ```

  Bot CLA-Assistant akan mengubah status PR menjadi hijau setelah semua penulis menandatangani.

```bash
# Mode watch (tes dijalankan ulang saat ada perubahan)
npm run test:watch

# Pemeriksaan tipe tanpa menghasilkan file
npm run typecheck

# Perbaiki masalah lint + prettier secara otomatis
npm run lint:fix
npm run format:fix
```

### Menulis perubahan kode berdampak tinggi

1. **Mulai dengan isu.** Buka isu baru atau komentari diskusi yang ada sehingga kami bisa menyepakati solusi sebelum kode ditulis.
2. **Tambahkan atau perbarui tes.** Setiap fitur baru atau perbaikan bug harus disertai dengan cakupan tes yang gagal sebelum perubahan dan lulus setelahnya. Cakupan 100% tidak diperlukan, tetapi bidik asersi yang bermakna.
3. **Dokumentasikan perilaku.** Jika perubahan Anda memengaruhi perilaku yang terlihat oleh pengguna, perbarui README, bantuan inline (`codex --help`), atau proyek contoh yang relevan.
4. **Jaga commit tetap atomik.** Setiap commit harus dapat dikompilasi dan tes harus lulus. Ini memudahkan peninjauan dan potensi rollback.

### Membuka pull request

- Isi template PR (atau sertakan informasi serupa) – **Apa? Mengapa? Bagaimana?**
- Jalankan **semua** pemeriksaan secara lokal (`npm test && npm run lint && npm run typecheck`). Kegagalan CI yang bisa ditangkap secara lokal memperlambat proses.
- Pastikan cabang Anda mutakhir dengan `main` dan konflik merge telah diselesaikan.
- Tandai PR sebagai **Siap untuk ditinjau** hanya ketika Anda yakin PR berada dalam kondisi yang dapat digabungkan.

### Proses peninjauan

1. Satu pengelola akan ditunjuk sebagai peninjau utama.
2. Kami mungkin meminta perubahan – jangan ambil ini secara pribadi. Kami menghargai kerja Anda, tetapi kami juga menghargai konsistensi dan pemeliharaan jangka panjang.
3. Ketika ada konsensus bahwa PR memenuhi standar, pengelola akan melakukan squash-and-merge.

### Nilai komunitas

- **Bersikap ramah dan inklusif.** Perlakukan orang lain dengan hormat; kami mengikuti [Contributor Covenant](https://www.contributor-covenant.org/).
- **Asumsikan niat baik.** Komunikasi tertulis itu sulit – condonglah pada sisi kemurahan hati.
- **Ajar dan belajar.** Jika Anda melihat sesuatu yang membingungkan, buka isu atau PR dengan perbaikan.

### Mendapatkan bantuan

Jika Anda mengalami masalah dalam menyiapkan proyek, ingin umpan balik tentang ide, atau hanya ingin menyapa – silakan buka Diskusi atau terjun ke isu yang relevan. Kami senang membantu.

Bersama-sama kita bisa menjadikan Codex CLI alat yang luar biasa. **Selamat hacking!** :rocket:

### Perjanjian Lisensi Kontributor (CLA)

Semua kontributor **harus** menerima CLA. Prosesnya ringan:

1. Buka pull request Anda.
2. Tempel komentar berikut (atau balas `recheck` jika Anda sudah menandatangani sebelumnya):

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. Bot CLA-Assistant mencatat tanda tangan Anda di repositori dan menandai pemeriksaan status sebagai lulus.

Tidak diperlukan perintah Git khusus, lampiran email, atau footer commit.

#### Perbaikan cepat

| Skenario          | Perintah                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------- |
| Perbaiki commit terakhir | `git commit --amend -s --no-edit && git push -f`                                          |
| Hanya UI GitHub    | Edit pesan commit di PR → tambahkan<br>`Signed-off-by: Your Name <email@example.com>` |

Pemeriksaan **DCO** memblokir penggabungan hingga setiap commit di PR membawa footer (dengan squash, ini hanya satu).

### Merilis `codex`

Untuk mempublikasikan versi baru CLI, jalankan skrip rilis yang didefinisikan di `codex-cli/package.json`:

1. Buka direktori `codex-cli`
2. Pastikan Anda berada di cabang seperti `git checkout -b bump-version`
3. Tingkatkan versi dan `CLI_VERSION` ke tanggal/waktu saat ini: `npm run release:version`
4. Commit peningkatan versi (dengan tanda tangan DCO):
   ```bash
   git add codex-cli/src/utils/session.ts codex-cli/package.json
   git commit -s -m "chore(release): codex-cli v$(node -p \"require('./codex-cli/package.json').version\")"
   ```
5. Salin README, bangun, dan publikasikan ke npm: `npm run release`
6. Push ke cabang: `git push origin HEAD`

---

## Keamanan & AI Bertanggung Jawab

Apakah Anda menemukan kerentanan atau memiliki kekhawatiran tentang keluaran model? Silakan kirim email ke **security@openai.com** dan kami akan segera merespons.

---

## Lisensi

Repositori ini dilisensikan di bawah [Lisensi Apache-2.0](LICENSE).