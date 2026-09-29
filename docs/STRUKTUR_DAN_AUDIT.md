# Struktur kode dan audit — 29 September 2026

## Prinsip

Kode hemat berarti satu tanggung jawab per bagian dan tidak mengulang aturan bisnis, bukan memadatkan semua kode dalam satu baris. Struktur Next.js dipertahankan agar URL dan impor yang sudah berjalan tidak rusak. Tidak ada dependensi baru, perubahan API, atau SQL pada paket ini.

## Peta folder

| Lokasi | Isi dan aturan |
| --- | --- |
| `src/app` | Halaman/rute Next.js; pengambilan data dan penyusunan layar. Jangan pindahkan nama khusus seperti `page.tsx` atau `route.ts`. |
| `src/app/api` | Validasi permintaan dan akses data server. Tidak boleh bergantung pada izin dari UI saja. |
| `src/components/ui` | Kontrol bersama: kartu, input, tombol, dialog, kalender. Perubahan di sini berdampak banyak halaman. |
| `src/components/layout` | Navigasi desktop/mobile, header, judul halaman, banner. |
| `src/components/dashboard` | Grafik, pintasan pilihan, dan dialog ringkasan harian. |
| `src/components/pengaturan` | Tema, PIN, kunci AI, dan persiapan API gerai. |
| `src/components/<modul>` | Formulir/panel khusus anggota, tugas, monitoring, keuangan. |
| `src/lib` | Perhitungan murni, hooks, validasi, koneksi data dan keamanan. Jangan masukkan JSX tampilan ke modul perhitungan. |
| `src/types` | Tipe data bersama. |
| `tests` | Pengujian data dan interaksi komponen; data buatan hanya untuk tes, bukan produksi. |
| `scripts` | Alat pengembang yang dapat dijalankan ulang. Audit sumber hanya membaca berkas. |
| `supabase/migrations` | Riwayat migrasi; bukan sampah walaupun tidak diimpor aplikasi. |
| `supabase/drafts` | Usulan SQL yang belum otomatis disetujui untuk dieksekusi. |
| `docs`, `prompts`, `brief` | Keputusan, petunjuk, dan konteks serah terima; tidak dihapus berdasarkan audit impor. |
| `.next`, `node_modules` | Hasil build/cache dan dependensi; bukan kode sumber untuk dirapikan manual. |

## Hasil audit sumber

Jalankan `npm.cmd run audit:source`. Skrip memakai parser/resolver TypeScript untuk menelusuri impor dan ekspor dari titik masuk Next.js, termasuk impor literal dinamis. Hasil: **129 berkas TypeScript sumber, 123 terjangkau, 6 kandidat** setelah perubahan ini.

Kandidat **bukan bukti bahwa berkas aman dihapus**. Skrip tidak menelusuri seluruh pemanggilan dinamis berbasis string, referensi dokumen, maupun penggunaan oleh alat eksternal.

| Kandidat | Keputusan |
| --- | --- |
| `src/lib/useCurrentUser.ts` | Dihapus: tidak ada pemakai di sumber/tes; mengandung identitas manajer tetap. Tampilan aktif memakai profil organisasi. Dapat dipulihkan dari Git. |
| `src/lib/auth/session.ts`, `src/lib/auth/utils.ts` | Dipertahankan: masih dipakai tes kompatibilitas/keamanan. Tidak dipakai jalur halaman aktif menurut audit. |
| `src/components/ui/Alert.tsx` | Dipertahankan: komponen bersama yang diuji; bisa dipakai mengganti peringatan berulang pada tahap berikut. |
| `src/components/ui/index.ts`, `DataTable.tsx`, `Skeleton.tsx` | Dipertahankan sebagai kandidat tinjauan. `DataTable` juga berisi perubahan dari sesi sebelumnya; tidak dihapus agar pekerjaan tersebut tidak hilang. |

Penghapusan `ModulePlaceholder.tsx` sudah ada sebelum sesi ini, bukan penghapusan baru. Tidak ada penghapusan data koperasi, tabel, atau riwayat migrasi.

## Perapian yang sudah dilakukan

- Grafik dipecah dari `ProductionDashboard.tsx` menjadi `DashboardTrends.tsx`; perhitungan lama tetap sama.
- Tema dipisah dari halaman Pengaturan menjadi `AppearanceSettings.tsx` dengan radio HTML asli.
- Preferensi pintasan divalidasi lewat `manager-shortcuts.ts`; hanya rute yang dikenal, tanpa duplikasi, maksimal enam.
- Ringkasan koordinasi dibentuk oleh fungsi murni `manager-briefing.ts`, sehingga angka dan batas laporan dapat diuji tanpa browser.
- Pengaturan menampilkan satu bagian aktif. Formulir tetap terpasang agar isian belum disimpan tidak hilang ketika berganti bagian.
- Vitest memakai transformasi JSX otomatis agar konsisten dengan Next/React; komponen tidak harus menambah impor React hanya untuk tes.

## Kelanjutan yang belum dikerjakan

1. Pecah formulir profil dan backup dari halaman Pengaturan setelah menambah tes simpan/reset/galat. Jangan mengubah format payload.
2. Audit halaman besar Tata Kelola, Bantuan, dan jurnal simulasi per fitur. Jangan menghapus fallback persiapan tanpa memeriksa kondisi database belum dikonfigurasi.
3. Tinjau kandidat UI: gunakan `DataTable`/`Alert` bila menyederhanakan halaman; bila benar-benar tidak dibutuhkan, hapus beserta ekspor/tes terkait dalam perubahan tersendiri.
4. Audit aksesibilitas keyboard serta visual tiap formulir di 360, 390, 820, dan 1366 px dengan teks panjang. Pemeriksaan sesi ini hanya spot-check dashboard, dialog pintasan, dan Pengaturan.
5. Pekerjaan besar terpisah: backup >1.000 baris/restore atomik, konektor gerai dengan persetujuan, pembukuan lengkap. Ikuti dokumen rancangan dan minta konfirmasi sebelum SQL.

## Verifikasi wajib

`npm.cmd test -- --run`, `npm.cmd run typecheck`, `npm.cmd run build`, lalu `npm.cmd run audit:source`.

Jangan menjalankan build bersamaan dengan dev karena keduanya memakai `.next`. Hentikan hanya server milik pekerjaan ini, jalankan build, lalu hidupkan kembali server lokal. Jangan menguji simpan/hapus/restore pada data koperasi tanpa data uji yang disetujui.
