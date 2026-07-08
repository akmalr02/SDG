# Dashboard ESG — Sintesa Group

Dokumentasi untuk file: `dashboard-ESG.html`

## Ringkasan

Dashboard interaktif single-file (HTML + Tailwind CDN + Chart.js) untuk menampilkan data ESG (Environmental, Social, Governance) 15 anak perusahaan Sintesa Group, mengacu pada standar **GRI (Global Reporting Initiative)**, periode data 2023–2025.

Tidak memerlukan backend/server — cukup dibuka langsung di browser. Seluruh data bersifat **dummy/placeholder** yang di-hardcode di dalam `<script>`.

## Teknologi

| Komponen | Keterangan |
|---|---|
| Tailwind CSS | via CDN (`cdn.tailwindcss.com`), dengan custom color palette (`brand`, `accent`, `teal`, `amber`, `violet`, `rose`, dll) |
| Chart.js 4.4.1 | via CDN, untuk semua visualisasi grafik |
| Font | Inter (Google Fonts) |
| Data | JavaScript object literal di dalam file, tidak ada fetch/API eksternal |

## Struktur Halaman

1. **Header** — judul dashboard + dua filter global:
   - `sel-pt` — dropdown 15 perusahaan (WS, SDS, MEPPO, SBG, SGE, TES, MPRD, MD, TA, TRS, MPH, SPP, SPM, GSME, PMB) + opsi "Semua PT"
   - `sel-yr` — dropdown tahun 2023/2024/2025 + opsi "Semua tahun"
2. **Tab kategori** (8 tab, tiap tab = satu section data GRI):
   - 📋 General (GRI 2-7, 2-8) — total karyawan, gender, status kepegawaian
   - ⚡ Energy (GRI 302-1) — konsumsi energi renewable/non-renewable
   - ♻️ Waste (GRI 306-4, 306-5) — limbah dialihkan & dibuang
   - 💧 Water (GRI 303-3, 303-4) — penarikan & pembuangan air
   - 🤝 Diversity & Equal Opportunity (GRI 405-1, 405-2) — komposisi governance, usia, rasio gaji
   - 👥 Employment (GRI 401-1, 401-3) — rekrutmen baru, cuti orang tua
   - 🦺 OHS (Occupational Health & Safety) — fatality, insiden, jam kerja
   - 📚 Training & Education (GRI 404-1) — jam pelatihan per gender/kategori
3. Setiap tab berisi kartu KPI ringkas + beberapa grafik (`bar`, `line`, `pie/doughnut` via Chart.js) beserta legenda custom.

## Struktur Data (JavaScript)

Data disimpan sebagai object literal terpisah per kategori, masing-masing berbentuk `{ [KODE_PT]: { [TAHUN]: {...metrik...} } }`:

- `RAW` — data umum karyawan per PT/tahun (gender, status, worker type)
- (energi, waste tersimpan dengan pola serupa di bagian atas file, sebelum `RAW` water)
- `DIVERSITY` — `gov` (komposisi gender governance), `age` (kelompok umur), `salary` (rasio gaji per kategori)
- `EMPLOYMENT` — data rekrutmen baru & cuti orang tua (`new`, `parental`)
- (OHS) — `perm`/`contract` masing-masing berisi `fatality`, `highcons`, `recordable`, `hours`
- `TRAINING` — `gender` (jam rata-rata pria/wanita), `category` (senior/mid/junior/staff)
- `YEARS = [2023, 2024, 2025]`
- `PALETTE` — array warna hex untuk grafik multi-seri

## Alur Logika Utama

- `activePT` dan `activeYear` adalah state global filter (default `'all'`), didefinisikan di baris ~695:
  ```js
  let activePT = 'all', activeYear = 'all';
  const charts = {};

  function getYrs() { return activeYear === 'all' ? YEARS : [activeYear]; }
  function getPTs() { return activePT === 'all' ? Object.keys(RAW) : [activePT]; }
  ```
  `getPTs()` sengaja memakai `Object.keys(RAW)` sebagai sumber daftar PT — artinya **PT baru otomatis ikut ter-filter** begitu ditambahkan ke objek `RAW`, tanpa perlu ubah logika filter.

- Setiap kategori punya fungsi agregasi `agg<Kategori>()` dengan pola yang identik — melakukan `reduce`/`forEach` menjumlahkan data mentah sesuai `getPTs()` dan `getYrs()`. Contoh untuk kategori General:
  ```js
  function aggGeneral() {
    const pts = getPTs(); const ys = getYrs();
    const o = { gender: {}, status: {}, workers: {} };
    YEARS.forEach(y => { o.gender[y]={m:0,f:0}; o.status[y]={p:0,c:0}; o.workers[y]={con:0,int:0,vol:0,out:0}; });
    pts.forEach(p => YEARS.forEach(y => {
      o.gender[y].m+=RAW[p].gender[y].m; o.gender[y].f+=RAW[p].gender[y].f;
      o.status[y].p+=RAW[p].status[y].p; o.status[y].c+=RAW[p].status[y].c;
      o.workers[y].con+=RAW[p].workers[y].con; o.workers[y].int+=RAW[p].workers[y].int;
      o.workers[y].vol+=RAW[p].workers[y].vol; o.workers[y].out+=RAW[p].workers[y].out;
    }));
    return o;
  }
  ```
  Kategori lain (`aggEnergy`, `aggWaste`, `aggWater`, `aggDiversity`, dst.) mengikuti pola yang sama, hanya field yang dijumlahkan berbeda sesuai bentuk data masing-masing.

- `switchTab(tab)` — mengganti tab aktif dan memanggil `refresh()`:
  ```js
  function switchTab(tab) {
    document.querySelectorAll('.tab-section').forEach(s=>s.classList.remove('active'));
    document.querySelectorAll('.tab-btn').forEach(b=>b.classList.remove('active'));
    document.getElementById('sec-'+tab).classList.add('active');
    document.getElementById('tab-'+tab).classList.add('active');
    activeTab = tab;
    refresh();
  }
  ```
- `refresh()` — memanggil fungsi `build<Kategori>()` sesuai tab aktif:
  ```js
  function refresh() {
    updateSubtitle();
    switch(activeTab) {
      case 'general':    buildGeneral(); break;
      case 'energy':     buildEnergy(); break;
      case 'waste':      buildWaste(); break;
      case 'water':      buildWater(); break;
      case 'diversity':  buildDiversity(); break;
      case 'employment': buildEmployment(); break;
      case 'ohs':        buildOHS(); break;
      case 'training':   buildTraining(); break;
    }
  }
  ```
- Setiap fungsi `build...()` mengikuti pola: hitung agregat → isi kartu KPI (`textContent`) → destroy chart lama (`dc()`) → buat instance `Chart` baru → render legenda custom (`leg()`).
- `setPT(id)` / `setYear(y)` — dipanggil dari `onchange` dropdown:
  ```js
  function setYear(y) { activeYear=y; refresh(); }
  function setPT(id)  { activePT=id; refresh(); }
  ```

## Cara Menambah / Mengubah Data

### Menambah PT (perusahaan) baru

Karena daftar PT diambil otomatis dari `Object.keys(RAW)`, PT baru harus ditambahkan **di semua 8 objek data mentah** dengan kode PT (key) yang sama persis, lalu ditambahkan juga sebagai opsi dropdown:

1. Tambahkan entry baru di tiap objek data (kode PT harus identik di semua objek):
   - `RAW` (baris ~542) — general: `gender`, `status`, `workers`
     ```js
     NEWPT: {
       gender:{2023:{m:0,f:0},2024:{m:0,f:0},2025:{m:0,f:0}},
       status:{2023:{p:0,c:0},2024:{p:0,c:0},2025:{p:0,c:0}},
       workers:{2023:{con:0,int:0,vol:0,out:0},2024:{con:0,int:0,vol:0,out:0},2025:{con:0,int:0,vol:0,out:0}}
     },
     ```
   - `ENERGY` (baris ~561) — `{2023:{nonren:0,ren:0}, 2024:{...}, 2025:{...}}`
   - `WASTE` (baris ~580) — `{2023:{divert:{recycled:0,composted:0,recovery:0},disposal:{landfill:0,incineration:0,other:0}}, ...}`
   - `WATER` (baris ~599) — `{2023:{wd:{surface:0,ground:0,seawater:0,third:0},discharge:{...}}, ...}`
   - `DIVERSITY` (baris ~618) — `{2023:{gov:{m:0,f:0},age:{u30:0,a3050:0,o50:0},salary:{senior:1,mid:1,junior:1}}, ...}`
   - `EMPLOYMENT` (baris ~637) — `{2023:{new:{m:0,f:0,u30:0,a3050:0,o50:0},parental:{entitled:{m:0,f:0},took:{m:0,f:0},returned:{m:0,f:0}}}, ...}`
   - `OHS` (baris ~656) — `{2023:{perm:{fatality:0,highcons:0,recordable:0,hours:0},contract:{...}}, ...}`
   - `TRAINING` (baris ~675) — `{2023:{gender:{m:0,f:0},category:{senior:0,mid:0,junior:0,staff:0}}, ...}`
2. Tambahkan opsi dropdown baru di HTML (baris ~63–78), pastikan `value` sama dengan key di atas:
   ```html
   <option class="text-gray-800" value="NEWPT">NEWPT — Nama Perusahaan</option>
   ```
3. Tidak perlu mengubah kode JavaScript lain — filter dan agregasi otomatis mengikuti.

### Menambah tahun baru (mis. 2026)

1. Tambahkan `2026` di setiap entry PT pada 8 objek data mentah (mengikuti bentuk `2025`).
2. Tambahkan `2026` ke array `YEARS` (baris ~693):
   ```js
   const YEARS = [2023, 2024, 2025, 2026];
   ```
3. Tambahkan `<option value="2026">2026</option>` di dropdown tahun (baris ~86-88).

### Menambah kategori/tab GRI baru

1. Buat objek data mentah baru mengikuti pola `{ [PT]: { [TAHUN]: {...metrik...} } }`.
2. Tambahkan section `<div id="sec-<nama>" class="tab-section">...</div>` di `<main>` dan tombol tab baru di navigasi (`tab-btn`).
3. Tambahkan fungsi `agg<Nama>()` dan `build<Nama>()` mengikuti pola kategori lain.
4. Tambahkan `case '<nama>': build<Nama>(); break;` di dalam `refresh()`.

## Cara Menjalankan

1. Buka file `dashboard-ESG.html` langsung di browser (double-click atau drag ke tab browser).
2. Membutuhkan koneksi internet karena Tailwind & Chart.js dimuat dari CDN.
3. Gunakan dropdown "Perusahaan" dan "Tahun" di header untuk memfilter seluruh dashboard, dan klik tab kategori untuk berpindah topik GRI.

## Catatan & Batasan

- Semua angka adalah **data dummy/placeholder**, bukan data riil perusahaan — ditandai jelas di komentar kode (`/* ... — placeholder */`).
- Tidak ada penyimpanan data (tidak ada backend, database, atau local storage) — filter hanya mengubah tampilan sesi berjalan.
- Cocok digunakan sebagai mockup/prototipe visual sebelum dihubungkan ke data HRIS/ESG yang sesungguhnya.
