# Hasil Survei SDG — Sintesa Group

Dokumentasi untuk file: `sdg_survey_results.html`

## Ringkasan

Halaman single-file (HTML + CSS custom + Tabler Icons) bergaya **editorial/majalah digital** (dark theme, font serif untuk judul) yang menampilkan hasil survei internal karyawan (format Google Form, jawaban Ya/Tidak) terkait persepsi karyawan atas komitmen SDG perusahaan.

Tidak memerlukan backend — data dummy tersimpan langsung di JavaScript, tanpa fetch API.

## Teknologi

| Komponen | Keterangan |
|---|---|
| CSS | Custom (CSS variables/`:root`), tanpa framework CSS eksternal |
| Tabler Icons | via CDN, untuk ikon tombol tutup panel detail |
| Font | Georgia/serif untuk judul (fallback lokal, bukan Google Fonts) |
| Data | Object literal `SDGS` di dalam `<script>` |

## Struktur Halaman

1. **Masthead** — judul "Persepsi Karyawan terhadap Komitmen SDG Sintesa Group" + meta info (jumlah responden: 312, periode survei: 1–28 Jun 2026, format jawaban Ya/Tidak).
2. **Lede** — paragraf pengantar yang menandaskan bahwa data bersifat dummy/mockup.
3. **Board (grid 3×3)** — kartu untuk tiap SDG yang disurvei (SDG 3, 5, 6, 7, 8, 12, 13, 16, 17), menampilkan:
   - Ikon/logo SDG resmi (fallback ke nomor kode jika gambar gagal dimuat)
   - Jumlah pertanyaan per SDG
   - Nama SDG
   - Rata-rata persentase jawaban "Ya" antar pertanyaan
   - Progress bar dua warna (lime = Ya, coral = Tidak)
4. **Legend & instruksi** — keterangan warna + arahan untuk klik kartu.
5. **Panel Detail** (muncul saat kartu diklik) — menampilkan seluruh pertanyaan untuk SDG terpilih, masing-masing dengan:
   - Bar horizontal proporsi Ya/Tidak (persentase)
   - Jumlah total responden per pertanyaan
6. **Footnote** — penegasan bahwa data & persentase adalah dummy untuk keperluan mockup.

## Struktur Data (JavaScript)

```
SDGS = [
  {
    id, code, img, name, n,
    questions: [
      { q, yes, no, total },
      ...
    ]
  },
  ...
]
```

- `id` — nomor SDG resmi (3, 5, 6, 7, 8, 12, 13, 16, 17)
- `img` — path gambar logo SDG (`img/Sustainable_Development_Goal_<n>.png`) — **catatan: folder `img/` tidak disertakan dalam file ini**, sehingga gambar akan gagal dimuat kecuali disediakan terpisah; sudah ada fallback ke tampilan kode angka (`onerror`)
- `questions[]` — setiap pertanyaan survei berisi teks pertanyaan (`q`), persentase jawaban `yes`/`no`, dan `total` responden untuk pertanyaan tersebut

## Fungsi JavaScript Utama

- `buildBoard()` (baris ~313) — merender 9 kartu grid SDG beserta rata-rata persentase "Ya" (dihitung oleh `avgYes()`):
  ```js
  function buildBoard(){
    document.getElementById('board').innerHTML = SDGS.map(s => {
      const avg = avgYes(s.questions);
      return `
      <div class="cell" id="cell-${s.id}" onclick="toggleDetail(${s.id})">
        <div class="cell-top">
          <img class="cell-img" src="${s.img}" alt="SDG ${s.code}" onerror="this.style.display='none'; this.nextElementSibling.style.display='block';"/>
          <div class="cell-num" style="display:none;">${s.code}</div>
          <div class="cell-resp"><div class="cell-resp-n">${s.questions.length} pertanyaan</div></div>
        </div>
        <div>
          <div class="cell-name">${s.name}</div>
          <div class="cell-sub">Rata-rata "Ya": ${avg}%</div>
          <div class="cell-bar">
            <div class="cell-bar-yes" style="width:${avg}%"></div>
            <div class="cell-bar-no" style="width:${100-avg}%"></div>
          </div>
        </div>
      </div>`;
    }).join('');
  }
  ```
- `avgYes(qs)` — menghitung rata-rata persentase jawaban "Ya" dari seluruh pertanyaan dalam satu SDG:
  ```js
  function avgYes(qs){
    const total = qs.reduce((a,q)=>a+q.yes,0);
    return Math.round(total / qs.length);
  }
  ```
- `toggleDetail(id)` — membuka/menutup panel detail pertanyaan untuk SDG yang diklik, mengisi header (gambar/nama) lalu me-render seluruh `questions[]` sebagai bar Ya/Tidak; melakukan smooth-scroll ke panel:
  ```js
  function toggleDetail(id){
    if(activeId === id){ closeDetail(); return; }
    activeId = id;
    document.querySelectorAll('.cell').forEach(c=>c.classList.remove('active'));
    document.getElementById('cell-'+id).classList.add('active');

    const s = SDGS.find(x=>x.id===id);
    // ...isi d-img, d-name, dst dari s...
    document.getElementById('d-body').innerHTML = s.questions.map((q,i)=>`
      <div class="q-block">
        <p class="q-text"><span class="q-num">${String(i+1).padStart(2,'0')}</span>${q.q}</p>
        <div class="q-result">
          <div class="q-bar-track">
            <div class="q-seg q-seg-yes" style="width:${q.yes}%"><span>${q.yes}%</span></div>
            <div class="q-seg q-seg-no" style="width:${q.no}%"><span>${q.no}%</span></div>
          </div>
        </div>
      </div>`).join('');
  }
  ```
- `closeDetail()` — menutup panel detail dan menghapus status aktif pada kartu.

## Cara Menambah / Mengubah Data

### Menambah SDG baru ke grid

Tambahkan object baru ke array `SDGS` (baris ~241):

```js
{
  id: 99, code: '99', img: 'img/Sustainable_Development_Goal_99.png',
  name: 'Nama SDG (Bahasa Indonesia)', n: 34,
  questions: [
    { q: 'Teks pertanyaan survei di sini?', yes: 70, no: 30, total: 312 },
    // tambahkan pertanyaan lain di sini
  ],
},
```

Catatan:
- `id` sebaiknya memakai nomor SDG resmi PBB (1–17) agar konsisten dengan file `sdg_roadmap_mockup01.html`.
- `img` harus menunjuk ke file logo di folder `img/` — jika file tidak ada, tampilan otomatis fallback ke `code` (lihat `onerror` di `buildBoard()`).
- `n` saat ini tidak dipakai untuk perhitungan (nilainya sama `34` di semua SDG) — kemungkinan sisa dari versi data sebelumnya, aman diabaikan atau dihapus.
- Grid 3×3 (CSS `.board`) otomatis menyesuaikan jumlah kartu — menambah/mengurangi elemen `SDGS` tidak memerlukan perubahan CSS, namun tampilan akan makin padat jika SDG bertambah banyak.

### Menambah/mengubah pertanyaan pada SDG yang sudah ada

Cukup tambah/ubah elemen pada `questions[]` milik SDG terkait:

```js
{ q: 'Pertanyaan baru?', yes: 55, no: 45, total: 300 }
```
- `yes` + `no` idealnya berjumlah 100 (dalam persen) karena keduanya dipakai langsung sebagai lebar bar (`width:${q.yes}%`).
- `total` adalah jumlah responden untuk pertanyaan spesifik itu (boleh berbeda antar pertanyaan, karena tidak semua responden menjawab semua pertanyaan — lihat data contoh yang sudah ada, `total` bervariasi 198–312).
- Tidak perlu mengubah `buildBoard()` atau `toggleDetail()` — keduanya otomatis merender ulang berdasarkan panjang array `questions[]`.

## Cara Menjalankan

1. Buka `sdg_survey_results.html` langsung di browser.
2. (Opsional) Sediakan folder `img/` berisi logo resmi SDG (format `Sustainable_Development_Goal_<nomor>.png`) di direktori yang sama agar ikon tampil; jika tidak ada, halaman tetap berfungsi dengan fallback kode angka.
3. Klik kartu SDG mana pun pada grid 3×3 untuk melihat rincian pertanyaan & distribusi jawaban.

## Catatan & Batasan

- Seluruh data survei (jumlah responden, persentase jawaban) adalah **dummy**, dinyatakan eksplisit di footnote halaman.
- Belum ada koneksi ke sumber data nyata (mis. Google Forms API/Google Sheets) — perlu integrasi lanjutan bila akan dipakai dengan data survei sungguhan.
- Gambar logo SDG resmi tidak disertakan; perlu disiapkan secara manual di folder `img/` sesuai penamaan yang digunakan dalam kode.
