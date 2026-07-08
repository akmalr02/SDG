# Dokumentasi Proyek ESG & SDG — Sintesa Group

Repositori ini berisi 3 mockup/prototipe halaman web **single-file** (HTML + CSS/JS, tanpa backend) terkait pelaporan ESG dan strategi SDG Sintesa Group. Semua data di dalamnya bersifat **dummy/placeholder** untuk keperluan demo tampilan.

## Daftar File

| File | Fungsi | Dokumentasi Detail |
|---|---|---|
| `dashboard-ESG.html` | Dashboard data ESG (GRI Standards) 15 anak perusahaan, 8 kategori (General, Energy, Waste, Water, Diversity, Employment, OHS, Training), dengan filter PT & tahun | [dashboard-ESG.md](./dashboard-ESG.md) |
| `sdg_roadmap_mockup01.html` | Wizard 4 langkah untuk menyusun rencana Road Map strategi SDG (pilih SDG → kategori → aksi bisnis → form konfirmasi) | [sdg_roadmap_mockup01.md](./sdg_roadmap_mockup01.md) |
| `sdg_survey_results.html` | Halaman hasil survei internal karyawan (Ya/Tidak) terkait persepsi SDG, gaya editorial/majalah digital | [sdg_survey_results.md](./sdg_survey_results.md) |

## Kesamaan Teknis Ketiga File

- **Single-file HTML** — semua CSS & JavaScript ada dalam satu file, tidak perlu proses build.
- **Tanpa backend/database** — semua data (dummy) di-hardcode sebagai object JavaScript di dalam `<script>`.
- **Bergantung pada CDN** — Tailwind CSS, Chart.js, dan/atau Tabler Icons dimuat dari internet, sehingga **membutuhkan koneksi internet** saat dibuka.
- **Bahasa Indonesia** — seluruh label UI dan konten menggunakan Bahasa Indonesia.
- Cara menjalankan: cukup buka file `.html` yang bersangkutan langsung di browser (double-click atau drag-and-drop).

## Perbedaan Gaya Visual

- `dashboard-ESG.html` — tema terang, korporat, berbasis Tailwind + Chart.js.
- `sdg_roadmap_mockup01.html` — tema terang, gaya form wizard step-by-step, Tailwind + Tabler Icons.
- `sdg_survey_results.html` — tema gelap, gaya editorial/majalah, custom CSS tanpa framework.

## Rekomendasi Pengembangan Selanjutnya

1. Menghubungkan ketiga mockup ke sumber data nyata (mis. database HRIS/ESG, Google Sheets, atau API internal) menggantikan data dummy.
2. Menambahkan penyimpanan hasil isian form (Road Map & Survei) ke backend agar tidak hilang saat halaman di-refresh.
3. Menyediakan folder `img/` berisi logo resmi SDG untuk `sdg_survey_results.html`.
4. Mempertimbangkan integrasi ketiganya ke dalam satu portal/menu navigasi terpadu jika akan digunakan bersamaan.

---
*Dokumentasi ini dibuat sebagai pendamping teknis untuk memudahkan pemahaman struktur, alur, dan data pada masing-masing file mockup.*
