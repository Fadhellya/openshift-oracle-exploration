# Summary Explorasi Deployment MySQL Cluster dan Integrasi dengan OpenShift

## Ringkasan
Dokumen ini menjelaskan langkah-langkah dalam melakukan deployment MySQL Cluster di atas vSphere serta integrasi dengan OpenShift. Implementasi ini mencakup pembuatan layanan eksternal, konfigurasi MySQL Router, serta koneksi dengan aplikasi dan phpMyAdmin.

## Alur Pengerjaan

### 1. Setup MySQL Cluster di vSphere
- Deploy 3 VM pada vSphere untuk menjalankan cluster MySQL.
- Konfigurasi 1 VM sebagai master dan 2 VM lainnya sebagai slave.
- Konversi resource router MySQL ke dalam bentuk image container.
- Push image MySQL Router ke registry DockerHub.

### 2. Konfigurasi OpenShift
#### a. Membuat Service External untuk Database
- Buat **Service External** untuk memberikan alias endpoint menuju VM database utama (primary) dalam cluster MySQL.

#### b. Deploy MySQL Router di OpenShift
- Deploy pod MySQL Router menggunakan image yang telah diunggah ke DockerHub.
- Konfigurasi environment MySQL Router menggunakan ConfigMap dan Secret agar dapat terhubung ke cluster MySQL.
- Contoh: `MYSQL_HOST` menggunakan service dari Service External yang telah dibuat sebelumnya.

#### c. Konfigurasi Koneksi MySQL Router ke Aplikasi
- Buat **Service** untuk MySQL Router agar dapat berkomunikasi dengan aplikasi dan phpMyAdmin.
- Deploy aplikasi dengan environment yang sesuai menggunakan **ConfigMap** dan **Secret**.
- Contoh: `DB_HOST` pada aplikasi mengarah ke service MySQL Router dengan port yang sesuai dari log router (misalnya **6446 untuk Read-Write, 6447 untuk Read-Only**).

#### d. Konfigurasi Akses Aplikasi
- Buat **Service** untuk aplikasi.
- Buat **Route** untuk mengizinkan akses ke aplikasi dari klien eksternal.

#### e. Konfigurasi phpMyAdmin
- Deploy pod phpMyAdmin.
- Buat **Service Account** agar phpMyAdmin memiliki akses root ke MySQL.
- Lakukan **RoleBinding** untuk memberikan izin yang sesuai.
- Buat **Service** dan **Route** untuk phpMyAdmin agar dapat diakses.

## Konfigurasi Environment
Berikut adalah konfigurasi environment untuk masing-masing komponen:

### **1. MySQL Router**
```yaml
ENV:
  MYSQL_HOST: <Nama Service External>
  MYSQL_INNODB_CLUSTER_MEMBERS: 3
  MYSQL_PORT: 3306
  MYSQL_PASSWORD: <password>
  MYSQL_USER: <username>
```

### **2. Aplikasi**
```yaml
ENV:
  DB_HOST: <Nama Service MySQL Router>
  DB_PORT: <Port dari log router, misal 6446>
  DB_NAME: <Nama database>
  DB_USER: <Username>
  DB_PASS: <Password>
```

### **3. phpMyAdmin**
```yaml
ENV:
  PMA_HOST: <Nama Service MySQL Router>
  APACHE_PORT: 8080
  PMA_PORT: 6446
```

## Kesimpulan
Dokumentasi ini menjelaskan secara detail proses deployment MySQL Cluster di vSphere dan bagaimana mengintegrasikannya dengan OpenShift menggunakan MySQL Router. Dengan pendekatan ini, aplikasi yang berjalan di OpenShift dapat mengakses database MySQL dengan metode yang lebih fleksibel dan aman.
