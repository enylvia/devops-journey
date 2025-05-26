# 🚀 DevOps Learning Recap: GitHub Actions & Kubernetes Introduction

**Date:** 2025-05-26  
**Mentor:** Partner Coding 🤖  
**Focus:** Membangun CI Pipeline dengan GitHub Actions, Pengenalan Konsep Kubernetes, dan Perbedaan Jenkins vs. Kubernetes.

---

## ✅ 1. GitHub Actions: Membangun CI Pipeline

Hari ini kita fokus pada implementasi **Continuous Integration (CI)** untuk aplikasi Go menggunakan **GitHub Actions**.

### 1.1 Konsep Dasar GitHub Actions

* **Workflow:** Serangkaian instruksi yang dieksekusi secara otomatis berdasarkan pemicu (event). Didefinisikan dalam file YAML.
* **Event (`on:`):** Pemicu *workflow* (misal: `push` ke branch `main`, `pull_request`).
* **Jobs (`jobs:`):** Kumpulan pekerjaan yang akan dijalankan. Bisa berjalan paralel atau berurutan (menggunakan `needs:`).
* **Steps (`steps:`):** Serangkaian aksi (perintah *shell* atau *action* bawaan) yang dilakukan dalam satu *job*.
* **Runner (`runs-on:`):** Lingkungan OS tempat *job* akan berjalan (misal: `ubuntu-latest`).

### 1.2 GitHub Actions Workflow (`main.yml`) untuk Go App

Kita membuat file `main.yml` di direktori `.github/workflows/` dengan dua *job* utama:

* **`build`:**
    * **Tujuan:** Meng-*compile* aplikasi Go dan menjalankan *unit test*.
    * **Langkah-langkah kunci:**
        * `actions/checkout@v4`: Mengambil kode dari repository.
        * `actions/setup-go@v5`: Menyiapkan lingkungan Go (sesuai versi).
        * `go mod download`: Mengunduh dependensi Go.
        * `go build -o main .`: Meng-*compile* Go app menjadi *executable*.
        * `go test ./...`: Menjalankan semua *unit test*.
* **`build-and-push-docker`:**
    * **Tujuan:** Membangun image Docker dari aplikasi Go dan meng-*push* ke Docker Hub.
    * **Ketergantungan (`needs: build`):** Hanya akan berjalan jika *job* `build` sukses.
    * **Variabel Lingkungan (`env:`):** Definisi variabel khusus untuk *job* ini.
        * `IMAGE_NAME: your-go-app-name` (Nama image Docker, ditulis langsung di YAML).
        * `REGISTRY: docker.io` (Registry Docker, `docker.io` sama dengan Docker Hub, ditulis langsung di YAML).
        * `DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}` (Diambil dari GitHub Secrets).
        * `DOCKER_TOKEN: ${{ secrets.DOCKER_TOKEN }}` (Diambil dari GitHub Secrets).
    * **Langkah-langkah kunci:**
        * `docker/login-action@v3`: Login ke Docker Hub menggunakan *secrets*.
        * `docker/build-push-action@v5`: Membangun dan meng-*push* image Docker dengan *tag* SHA commit dan `latest`.

### 1.3 Pengelolaan Environment Variables (`env`) dan Secrets

* **`.env` file (Lokal):** Untuk konfigurasi **lokal development**, **TIDAK DI-COMMIT KE GIT**.
* **`env:` di GitHub Actions YAML:** Untuk variabel lingkungan **non-sensitif** atau referensi ke *secrets* **di dalam GitHub Actions workflow**. Nilainya ditulis langsung di YAML atau diambil dari *secrets*.
* **GitHub Secrets:** Fitur keamanan GitHub untuk menyimpan **kredensial sensitif** (username, token, password) secara aman. Nilainya terenkripsi dan tidak terlihat di log.
    * Di-*setup* di: `GitHub Repository > Settings > Secrets and variables > Actions > New repository secret`.
    * Wajib set `DOCKER_USERNAME` (Docker ID lo) dan `DOCKER_TOKEN` (Personal Access Token dari Docker Hub).
    * **Hak Akses PAT Docker Hub:** Harus `Read, Write, Delete` agar bisa *push* image.
    * URL Docker Hub (`docker.io`) adalah nama domain resmi untuk Docker Hub.

### 1.4 Troubleshooting Umum (Error `denied: requested access to the resource is denied`)

* Error ini menunjukkan masalah otentikasi atau izin saat *push* image ke Docker Hub.
* **Penyebab umum:**
    * `DOCKER_USERNAME` atau `DOCKER_TOKEN` salah/kadaluarsa di GitHub Secrets.
    * Hak akses PAT tidak cukup (hanya `Read`, bukan `Read, Write, Delete`).
    * Repository di Docker Hub belum dibuat secara manual (meskipun biasanya otomatis, kadang perlu dibuat jika ada isu izin).
* **Solusi:** Verifikasi ulang *secrets*, perbarui PAT dengan izin yang benar, dan pastikan *repository* di Docker Hub sudah ada.

## ✅ 2. Pengenalan Kubernetes (K8s)

Kubernetes adalah **platform *open-source* untuk otomatisasi *deployment*, *scaling*, dan *management* aplikasi yang dikontainerisasi**. Ini adalah **orkestrator kontainer**.

### 2.1 Kenapa Butuh Kubernetes?

* Mengelola banyak kontainer di banyak server (node).
* Skalabilitas, *self-healing*, dan *deployment* otomatis.
* *Service discovery* dan *load balancing*.
* Manajemen konfigurasi dan *secrets* yang aman.

### 2.2 Konsep Dasar & Arsitektur

* **Control Plane (Master Node):** "Otak" cluster, mengatur dan mengorkestrasi.
    * **Kube-API Server:** Pintu gerbang utama cluster.
    * **etcd:** "Database" cluster, menyimpan semua state dan konfigurasi.
    * **Kube-Scheduler:** Menentukan di mana kontainer akan dijalankan.
    * **Kube-Controller-Manager:** Memantau state cluster dan mencoba mencapai *desired state*.
* **Worker Node (Kubelet Node):** Tempat kontainer aplikasi dijalankan.
    * **Kubelet:** *Agent* yang berkomunikasi dengan Master dan menjalankan kontainer.
    * **Kube-Proxy:** Mengatur *network rules* dan *load balancing* untuk *Service*.
    * **Container Runtime:** Mesin yang menjalankan kontainer (misal: Docker).

### 2.3 Objek Dasar (Sekilas)

* **Pod:** Unit terkecil yang bisa di-*deploy*, berisi satu atau beberapa kontainer.
* **Deployment:** Mendefinisikan bagaimana aplikasi di-*deploy* dan di-*maintain* (memastikan jumlah Pod yang diinginkan selalu jalan).
* **Service:** Mengekspos aplikasi (Pod) ke dunia luar atau ke aplikasi lain di dalam cluster dengan IP dan DNS yang stabil.

### 2.4 Perbedaan Jenkins (CI/CD Tool) dan Kubernetes (Orkestrator Kontainer)

* **Jenkins (Event Handler/CI/CD Konduktor):**
    * Fokus pada **otomatisasi alur kerja** dari kode sampai **image Docker siap**.
    * Mendeteksi perubahan kode, *build*, *test*, *package* (misal: bikin image Docker), dan **memicu proses *deployment***.
    * Bekerja sebagai "konduktor" yang memberikan perintah ke *tool* lain untuk *deploy*.
* **Kubernetes (Orkestrator Kontainer):**
    * Fokus pada **menjalankan dan mengelola** aplikasi yang sudah dikontainerisasi di dalam cluster.
    * **Menerima perintah *deployment*** (misal dari Jenkins/GitHub Actions), menarik image, **melakukan *deployment* sebenarnya**, dan **mengelola *lifecycle* kontainer** di dalamnya (scaling, self-healing, dll.).

**Analogi:** Jenkins (atau GitHub Actions) adalah chef yang membuat dan mengemas makanan (image Docker), lalu memberikan perintah pengiriman. Kubernetes adalah pengelola restoran yang menerima makanan, menyajikannya (deploy), dan memastikan selalu tersedia dan dikelola dengan baik.

---