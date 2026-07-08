# SDG Strategy Road Map — Sintesa Group

Dokumentasi untuk file: `sdg_roadmap_mockup01.html`

## Ringkasan

Mockup portal interaktif single-file (HTML + Tailwind CDN + Tabler Icons) berupa **wizard 4 langkah** untuk menyusun rencana strategi SDG (Sustainable Development Goals) perusahaan: memilih tujuan SDG, kategori strategi, aksi bisnis spesifik, hingga mengisi form konfirmasi tindak lanjut.

Tidak memerlukan backend — seluruh alur dan data tersimpan di memori browser (JavaScript), tidak ada penyimpanan permanen.

## Teknologi

| Komponen | Keterangan |
|---|---|
| Tailwind CSS | via CDN |
| Tabler Icons | via CDN (`@tabler/icons-webfont`) |
| Data | Object literal `SDGs` di dalam `<script>`, tidak ada fetch API |

## Alur Wizard (4 Langkah)

1. **Step 1 — Pilih tujuan SDG**
   Grid kartu pilihan tunggal (radio-style) untuk satu SDG (mis. SDG 3, 5, 6, dst.). Progress bar menunjukkan berapa dari total SDG yang tersedia.
2. **Step 2 — Pilih kategori & kolom**
   - Kategori strategi (pilih satu): `Policies`, `Operation`, `Investment`, `Partnership`.
   - Kolom tampilan (checkbox multi-pilih): kolom informasi yang ingin ditampilkan di tabel hasil (mis. Deliverable, Area Strategy, Action, Detail, Indicator) — ada tombol "Pilih semua".
3. **Step 3 — Pilih aksi spesifik**
   Daftar *Key Business Action* (pilih satu, radio-style) sesuai SDG + kategori yang dipilih. Setelah memilih, panel detail otomatis menampilkan:
   - **Detail Action & Solution**
   - **Action Indicator** (indikator keberhasilan/KPI)
4. **Step 4 — Hasil & Form Isian**
   - Ringkasan hasil pilihan dalam bentuk tabel/summary banner (kolom sesuai yang dipilih di Step 2).
   - Form isian konfirmasi tindak lanjut: radio "Apakah tindakan ini dilakukan? (Iya/Tidak)" + textarea keterangan.
   - Tombol: **Unduh isian**, **Simpan & konfirmasi**, **Cetak** (via `window.print()`), **Ubah** (kembali ke Step 3), dan **Mulai dari awal**.

Navigasi antar tab (`tab1`–`tab4`) di bagian atas hanya aktif jika prasyarat langkah sebelumnya terpenuhi (validasi di fungsi `goStep()`).

## Struktur Data (JavaScript)

```
SDGs = [
  {
    id, label, name, title, bg, color,
    categories: {
      Policies:    { deliverable, areaStrategy, actions: [{ action, detail, indicator }, ...] },
      Operation:   { ... },
      Investment:  { ... },
      Partnership: { ... }
    }
  },
  ...
]
```

- Setiap elemen SDG punya `id` (nomor SDG resmi PBB), `label`/`name`/`title`, warna badge (`bg`, `color`).
- Setiap `categories` berisi 4 strategi tetap (Policies/Operation/Investment/Partnership), masing-masing memiliki:
  - `deliverable` — dokumen/output yang dihasilkan
  - `areaStrategy` — area strategis terkait
  - `actions[]` — daftar aksi bisnis konkret beserta `detail` (penjelasan) dan `indicator` (indikator ukur keberhasilan)
- Konten mencakup topik seperti K3, cuti orang tua, BPJS kesehatan, gender equality (GEP), pengelolaan air, energi, dll — merepresentasikan berbagai SDG (3, 5, 6, dan lainnya).

## Fungsi JavaScript Utama

- `goStep(n)` (baris ~1060) — mengatur perpindahan antar step, memvalidasi prasyarat sebelum boleh lanjut, dan styling tab aktif:
  ```js
  function goStep(n) {
    if (n === 2 && !selectedSdg) return;
    if (n === 3 && (!selectedCat || selectedViews.size === 0)) return;
    if (n === 4 && !selectedAction) return;
    currentStep = n;

    const steps = ['step1','step2','step3','step4'];
    steps.forEach((id, i) => {
      document.getElementById(id).classList.toggle('show', i + 1 === n);
    });

    const tabs = ['tab1','tab2','tab3','tab4'];
    tabs.forEach((id, i) => {
      const active = i + 1 === n;
      const el = document.getElementById(id);
      el.style.color = active ? '#0f766e' : '#9ca3af';
      el.style.borderBottomColor = active ? '#0f766e' : 'transparent';
      el.style.fontWeight = active ? '600' : '500';
    });

    if (n === 2) { buildCatGrid(); buildViewGrid(); updateF2(); }
    if (n === 3) { buildActionList(); }
    if (n === 4) { buildStep4(); }
  }
  ```
  Pola validasi di atas adalah tempat pertama yang perlu diperiksa jika ingin mengubah aturan "boleh lanjut ke step berikutnya".

- `buildSdgGrid()` — merender grid pilihan SDG di Step 1 dari array `SDGs`.
- `buildCatGrid()`, `buildViewGrid()` — merender pilihan kategori & checkbox kolom di Step 2, dibangun dari `selectedSdg.categories` dan daftar kolom tetap (Deliverable, Area Strategy, Action, Detail, Indicator).
- `buildActionList()` — merender daftar aksi bisnis di Step 3 dari `selectedSdg.categories[selectedCat].actions`; menampilkan `detail`/`indicator` saat satu aksi dipilih.
- `buildStep4()` — merakit tabel ringkasan hasil (baris ~910-960), memisahkan kolom "summary" (Deliverable/Area Strategy) dari kolom "tabel" (Action/Detail/Indicator):
  ```js
  if (summCols.length > 0) {
    summRows.innerHTML = `
      <div class="grid grid-cols-${summCols.length === 1 ? '1' : '2'} gap-0 border-b border-gray-100">
        ${summCols.map(v => {
          const val = v.id === 'deliverable' ? catData.deliverable : catData.areaStrategy;
          return `<div class="flex items-start gap-3 px-5 py-4">...${val}...</div>`;
        }).join('')}
      </div>`;
  }
  ```
- `buildFormSections()` — merender form isian konfirmasi (radio "Iya/Tidak" + textarea keterangan), menyimpan nilainya ke object `formValues`.
- `toggleAllViews()` — toggle pilih/batal semua checkbox kolom di Step 2.
- `clearForm()`, `submitForm()`, `downloadForm()` — aksi pada form isian (reset, simpan/konfirmasi via toast, siapkan baris teks untuk unduhan).
- `showToast(msg, icon)` — notifikasi toast di pojok kanan bawah.

## Cara Menambah / Mengubah Data

### Menambah SDG baru

Tambahkan object baru ke array `SDGs` (baris ~313), mengikuti struktur lengkap berikut (contoh diringkas dari SDG 3 yang sudah ada):

```js
{
  id: 99, label: 'SDG 99', name: 'Nama SDG (Bahasa Inggris)',
  title: 'Deskripsi singkat fokus SDG ini',
  bg: '#e8f5e9', color: '#2e7d32',   // warna badge: bg = latar, color = teks/ikon
  categories: {
    Policies: {
      deliverable: 'Dokumen/output yang dihasilkan kategori ini',
      areaStrategy: 'Nama area strategis terkait',
      actions: [
        { action: 'Nama aksi bisnis', detail: 'Penjelasan detail aksi & solusinya', indicator: 'Indikator/KPI keberhasilan aksi' },
        // tambahkan aksi lain di sini
      ],
    },
    Operation:   { deliverable: '...', areaStrategy: '...', actions: [ /* ... */ ] },
    Investment:  { deliverable: '...', areaStrategy: '...', actions: [ /* ... */ ] },
    Partnership: { deliverable: '...', areaStrategy: '...', actions: [ /* ... */ ] },
  },
},
```

**Penting:** keempat kategori (`Policies`, `Operation`, `Investment`, `Partnership`) harus selalu ada untuk tiap SDG — kode Step 2/3 mengasumsikan keempatnya tersedia. Jika salah satu kategori memang tidak relevan, isi `actions: []` (array kosong) daripada menghapus key-nya.

### Menambah/mengubah aksi bisnis dalam kategori yang sudah ada

Cukup tambahkan atau ubah elemen di dalam array `actions[]` pada kategori & SDG yang dituju:

```js
actions: [
  { action: 'Aksi baru', detail: 'Detail aksi baru', indicator: 'Indikator aksi baru' },
]
```
Tidak perlu mengubah kode JavaScript lain — `buildActionList()` dan `buildStep4()` otomatis merender ulang berdasarkan isi array ini.

### Menambah/mengubah kolom tampilan (view/kolom)

Kolom tampilan (Deliverable, Area Strategy, Action, Detail, Indicator) beserta ikon dan warnanya didefinisikan di bagian data view (dekat `buildViewGrid()`). Untuk menambah kolom baru:
1. Tambahkan definisi kolom baru (id, label, icon, color) di daftar kolom.
2. Tambahkan properti terkait di setiap object kategori SDG (mis. jika kolom baru butuh field `owner`, tambahkan `owner: '...'` di tiap kategori).
3. Tambahkan pemetaan render untuk kolom baru di `buildStep4()` (bagian `summCols`/`tableCols`).

## Cara Menjalankan

1. Buka `sdg_roadmap_mockup01.html` langsung di browser (butuh koneksi internet untuk CDN Tailwind & Tabler Icons).
2. Ikuti wizard dari Step 1 hingga Step 4; setiap step memvalidasi kelengkapan pilihan sebelum bisa lanjut.

## Catatan & Batasan

- Ini adalah **mockup tampilan**, bukan aplikasi produksi — belum ada integrasi backend/database untuk menyimpan hasil isian secara permanen.
- Tombol "Unduh isian" dan "Simpan & konfirmasi" saat ini hanya mensimulasikan aksi (toast notifikasi), belum benar-benar mengirim/menyimpan data ke server.
- Data SDG dan aksi bisnis di dalamnya sudah cukup detail dan tampak representatif untuk konten riil Sintesa Group, namun perlu divalidasi ulang dengan tim terkait sebelum dipakai sebagai rujukan resmi.
