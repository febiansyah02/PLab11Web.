📝 Laporan Praktikum 11: Pemrograman Web 2 - Integrasi Frontend VueJS 3

Laporan ini dibuat untuk memenuhi tugas mata kuliah Pemrograman Web 2 pada program studi Teknik Informatika, Universitas Pelita Bangsa. Praktikum kali ini berfokus pada implementasi arsitektur Decoupled Application dengan membangun Frontend API menggunakan VueJS 3 dan Axios sebagai client dari REST API backend CodeIgniter 4.

👤 Identitas Mahasiswa:

Nama: M. Febiansyah Mulyadi

Kampus: Universitas Pelita Bangsa

Mata Kuliah: Pemrograman Web 2

Dosen Pengampu: Agung Nugroho

📂 Struktur Direktori Proyek

Proyek Frontend didekatkan pada direktori lokal web server (C:\xampp\htdocs\lab8_vuejs) dengan pembagian berkas statis (Assets) yang teratur sebagai berikut:

```
lab8_vuejs/
├── assets/
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── app.js
└── index.html
```


💻 Implementasi Kode Sumber

1. File HTML Antarmuka (index.html)

Menyusun kerangka dokumen web, mengintegrasikan pustaka VueJS 3 dan Axios melalui CDN, serta mendeklarasikan elemen reaktif untuk rendering tabel data dan pop-up form modal.
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Frontend Vuejs 3 - CRUD Artikel</title>
    <!-- VueJS 3 & Axios CDN -->
    <script src="[https://unpkg.com/vue@3/dist/vue.global.js](https://unpkg.com/vue@3/dist/vue.global.js)"></script>
    <script src="[https://unpkg.com/axios/dist/axios.min.js](https://unpkg.com/axios/dist/axios.min.js)"></script>
    <link rel="stylesheet" href="assets/css/style.css">
</head>
<body>
    <div id="app">
        <h1>Daftar Artikel</h1>
```
        <button id="btn-tambah" @click="tambah">Tambah Data</button>

        <!-- Pop-Up Modal Form -->
        <div class="modal" v-if="showForm">
            <div class="modal-content">
                <span class="close" @click="showForm = false">&times;</span>
                <form id="form-data" @submit.prevent="saveData">
                    <h3 id="form-title">{{ formTitle }}</h3>
                    
                    <div>
                        <input type="text" name="judul" v-model="formData.judul" placeholder="Judul" required>
                    </div>
                    <div>
                        <textarea name="isi" id="isi" rows="10" v-model="formData.isi" placeholder="Isi Artikel" required></textarea>
                    </div>
                    <div>
                        <select name="status" id="status" v-model="formData.status">
                            <option v-for="option in statusOptions" :key="option.value" :value="option.value">
                                {{ option.text }}
                            </option>
                        </select>
                    </div>
                    
                    <input type="hidden" id="id" v-model="formData.id">
                    <button type="submit" id="btnSimpan">Simpan</button>
                    <button type="button" @click="showForm = false">Batal</button>
                </form>
            </div>
        </div>

        <!-- Tabel Merender Data Dinamis -->
        <table>
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Judul</th>
                    <th>Status</th>
                    <th>Aksi</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(row, index) in artikel" :key="row.id">
                    <td class="center-text">{{ row.id }}</td>
                    <td>{{ row.judul }}</td>
                    <td>{{ statusText(row.status) }}</td>
                    <td class="center-text">
                        <a href="#" @click.prevent="edit(row)">Edit</a> | 
                        <a href="#" @click.prevent="hapus(index, row.id)">Hapus</a>
                    </td>
                </tr>
                <tr v-if="artikel.length === 0">
                    <td colspan="4" class="center-text">Tidak ada data artikel.</td>
                </tr>
            </tbody>
        </table>
    </div>

    <script src="assets/js/app.js"></script>
</body>
</html>


2. File Script VueJS (assets/js/app.js)

Mengatur manajemen state reaktif dan mengintegrasikan request HTTP asynchronous (GET, POST, PUT, DELETE) ke backend REST API local server (localhost:8080).
```
const { createApp } = Vue

const apiUrl = 'http://localhost:8080'

createApp({
    data() {
        return {
            artikel: [],
            formData: {
                id: null,
                judul: '',
                isi: '',
                status: 0
            },
            showForm: false,
            formTitle: 'Tambah Data',
            statusOptions: [
                { text: 'Draft', value: 0 },
                { text: 'Publish', value: 1 }
            ]
        }
    },
    mounted() {
        this.loadData()
    },
    methods: {
        // Ambil Data Utama
        loadData() {
            axios.get(apiUrl + '/post')
                .then(response => {
                    this.artikel = response.data.artikel
                })
                .catch(error => console.error("Gagal memuat data:", error))
        },
        // Membuka Form Kosong
        tambah() {
            this.showForm = true
            this.formTitle = 'Tambah Data'
            this.formData = { id: null, judul: '', isi: '', status: 0 }
        },
        // Membuka Form dengan Data Lama
        edit(data) {
            this.showForm = true
            this.formTitle = 'Ubah Data'
            this.formData = {
                id: data.id,
                judul: data.judul,
                isi: data.isi,
                status: parseInt(data.status)
            }
        },
        // Menyimpan Data (Create / Update)
        saveData() {
            const payload = {
                judul: this.formData.judul,
                isi: this.formData.isi
            };
```
            if (this.formData.id) {
                // HTTP PUT untuk Update
                axios.put(apiUrl + '/post/' + this.formData.id, payload)
                    .then(response => {
                        alert('Data artikel berhasil diperbarui!');
                        this.loadData()
                        this.showForm = false
                    })
                    .catch(error => console.error("Gagal mengubah data:", error))
            } else {
                // HTTP POST untuk Create Baru
                axios.post(apiUrl + '/post', payload)
                    .then(response => {
                        alert('Data artikel berhasil ditambahkan!');
                        this.loadData()
                        this.showForm = false
                    })
                    .catch(error => console.error("Gagal menambah data:", error))
            }

            this.formData = { id: null, judul: '', isi: '', status: 0 };
        },
        // Hapus Data
        hapus(index, id) {
            if (confirm('Yakin menghapus data artikel ini?')) {
                axios.delete(apiUrl + '/post/' + id)
                    .then(response => {
                        alert('Data artikel berhasil dihapus!');
                        this.artikel.splice(index, 1)
                    })
                    .catch(error => console.error("Gagal menghapus data:", error))
            }
        },
        statusText(status) {
            return status == 1 ? 'Publish' : 'Draft'
        }
    }

🛠️ Improvisasi & Troubleshooting (CORS Bypass)

Selama proses pengerjaan praktikum, terjadi masalah keamanan browser di mana request POST/PUT dari VueJS diblokir oleh kebijakan keamanan CORS (Cross-Origin Resource Sharing).

Masalah ini diselesaikan secara tuntas dan elegan dengan cara mem-bypass serta menyuntikkan restriksi header secara paksa pada Controller dan Filters di sisi Backend CodeIgniter 4:

Penerimaan Raw JSON pada app/Controllers/Post.php:
Metode penerimaan input diubah dari $this->request->getVar() menjadi $this->request->getJsonVar() agar dapat mendecode kiriman data JSON mentah dari Axios secara native.

Penyuntikan Header Manual pada app/Config/Filters.php:
Menuliskan instruksi header global dan merespon langsung Preflight Request (OPTIONS method) di dalam method konstruktor kelas filter utama:

```
public function __construct()
{
    parent::__construct();
    header("Access-Control-Allow-Origin: *");
    header("Access-Control-Allow-Headers: X-API-KEY, Origin, X-Requested-With, Content-Type, Accept, Access-Control-Request-Method, Authorization");
    header("Access-Control-Allow-Methods: GET, POST, OPTIONS, PUT, DELETE");
    if (isset($_SERVER['REQUEST_METHOD']) && $_SERVER['REQUEST_METHOD'] === "OPTIONS") {
        die();
    }
}
```

📸 Dokumentasi Hasil Pengujian CRUD

Berikut adalah dokumentasi visual pengujian performa fungsional aplikasi VueJS:

<img width="959" height="539" alt="Cuplikan layar 2026-06-10 064707" src="https://github.com/user-attachments/assets/647bd2b4-77e1-4b5d-9591-f859ad2ce39e" />

<img width="945" height="539" alt="Cuplikan layar 2026-06-10 064747" src="https://github.com/user-attachments/assets/0e6a9e4a-f7d1-457d-bcc8-fe05f707336a" />

<img width="960" height="540" alt="Cuplikan layar 2026-06-10 070801" src="https://github.com/user-attachments/assets/6d72f60c-00f9-4916-a325-a3f484e622e4" />


A. Tampilan Utama Aplikasi (Read)

Aplikasi memanggil REST API server pada port :8080 dan berhasil menyajikan seluruh artikel dari database ke dalam tabel secara reaktif.


B. Proses Tambah Artikel Baru (Create)

Menampilkan form modal pop-up yang menerima input judul dan isi, kemudian mengirimkannya dalam format JSON.


C. Proses Edit Artikel (Update)

Mengisi form modal secara otomatis dengan data lama yang dipilih pengguna, lalu mengirimkan request update (PUT) ke server database.


D. Konfirmasi Hapus Artikel (Delete)

Menampilkan modal dialog konfirmasi browser untuk memvalidasi penghapusan artikel sebelum data dibersihkan secara permanen.


💡 Kesimpulan

Praktikum ini berhasil membuktikan keunggulan arsitektur modern Decoupled Application, di mana frontend (VueJS) dan backend (CodeIgniter 4) dapat bekerja secara terpisah dan berkomunikasi hanya lewat jembatan JSON data. Sistem reaktivitas VueJS terbukti mampu meningkatkan kenyamanan user experience (single page application feel) karena manipulasi data dapat berjalan cepat tanpa adanya pemuatan ulang (page reload) halaman browser.
