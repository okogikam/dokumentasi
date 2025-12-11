# First Time setup

Saat pertama kali membuka aplikasi akan mengecek apakah ada database `rekap_data` di database jika tidak ada maka akan diarahkan ke halaman install.
```php
$host = 'localhost';
$user = 'root';
$pass = '';
$dbname = 'rekap_data';

// 1. Koneksi tanpa pilih database dulu
$conn = new mysqli($host, $user, $pass);

// 2. Jika MySQL tidak bisa diakses
if ($conn->connect_errno) {
    die("Gagal konek ke MySQL: " . $conn->connect_error);
}

// 3. Cek apakah database `rekap_data` ada
$db_check = $conn->query("SHOW DATABASES LIKE '$dbname'");

// 4. Jika database belum ada → arahkan ke install
if ($db_check->num_rows === 0) {
    header("Location: install.php");
    exit;
}
```

Pada halaman install akan mengisikan data seperti :
1. Nama Prodi
2. Nama Operator
3. Kode Prodi
4. Kode universitas
5. Nama Fakultas
6. Jenjang
7. Status Prodi

kemudian prosesnya:
1. menyimpan isian dalam object `data_prodi.json` sementara JS.

    ```js
    {
        "prodi": $nama_prodi,
        "operator": $nama_operator,
        "kode_prodi": $kode_prodi,
        "kode_univ": $kode_universitas,
        "fakultas": $nama_fakultas,
        "jenjang": $jenjang_S1,
        "status": $status_prodi
    }
    ```

2. membuat database `rekap_data`. buat file `install_code.php` dan isikan:

    ```php
    $host = 'localhost';
    $user = 'root';
    $pass = '';
    $dbname = 'rekap_data';

    // 1. Koneksi tanpa pilih database dulu
    $conn = new mysqli($host, $user, $pass);

    // 2. Jika MySQL tidak bisa diakses
    if ($conn->connect_errno) {
        die("Gagal konek ke MySQL: " . $conn->connect_error);
    }

    // 3. Membuat database
    $query = "CREATE DATABASE rekap_data";
    $conn->query($query);
    ```
3. konek ke database `rekap_data`.
    ```php
    // 4. Koneksi ke database rekap_data
    $conn->select_db("rekap_data");
    ```
4. membuat tabel yang ada di `template_db` ke database `rekap_data` .seperti: `tabel_prodi`,`tabel_dosen`,`tabel_mhs`, dll.
    ```php
    // 5. Folder template_db
    $folder = __DIR__ . "/template_db";

    // 6. Ambil semua file .sql di folder
    $sql_files = glob($folder . "/*.sql");
    foreach ($sql_files as $file) {

        // 7. Baca isi file SQL
        $sql = file_get_contents($file);

        if (!$sql) {
            // Gagal membaca file 
            continue;
        }

        // 8. Eksekusi SQL (multi query)
        if ($conn->multi_query($sql)) {
            
            // Bersihkan result set untuk multi_query
            while ($conn->more_results()) {
                $conn->next_result();
            }
        } else {
            // Error handeler
            $conn->error;
        }
    }
    ```
5. menyimpan data di `data_prodi.json` kedalam tabel `tabel_prodi`.
    ```php
    //9. membuka file data_prodi
    $data_prodi = json_decode(file_get_content("./data_prodi.json"), true);

    //10. Masukkan semua data JSON ke tabel_prodi
    foreach ($data_prodi as $row) {
        // Ambil data dengan perlindungan aman
        $kode_prodi = $conn->real_escape_string($row['kode_prodi']);
        $nama_prodi = $conn->real_escape_string($row['prodi']);
        $fakultas    = $conn->real_escape_string($row['fasultas']);

        $query = "
            INSERT INTO tabel_prodi (kode_prodi, nama_prodi, fakultas)
            VALUES ('$kode_prodi', '$nama_prodi', '$fakultas')
        ";

        if (!$conn->query($query)) {
            echo "Gagal memasukkan data prodi: " . $conn->error . "<br>";
        }
    }
    ```
6. kembali ke halaman `index.php`.
    ```php
    //11. Kembali ke index.php
    header("Location: index.php");
    exit;
    ```

file `install_code.php`:
```php
<?php
    $host = 'localhost';
    $user = 'root';
    $pass = '';
    $dbname = 'rekap_data';

    // 1. Koneksi tanpa pilih database dulu
    $conn = new mysqli($host, $user, $pass);

    // 2. Jika MySQL tidak bisa diakses
    if ($conn->connect_errno) {
        die("Gagal konek ke MySQL: " . $conn->connect_error);
    }

    // 3. Membuat database
    $query = "CREATE DATABASE rekap_data";
    $conn->query($query);

    // 4. Koneksi ke database rekap_data
    $conn->select_db("rekap_data");

    // 5. Folder template_db
    $folder = __DIR__ . "/template_db";

    // 6. Ambil semua file .sql di folder
    $sql_files = glob($folder . "/*.sql");
    foreach ($sql_files as $file) {

        // 7. Baca isi file SQL
        $sql = file_get_contents($file);

        if (!$sql) {
            // Gagal membaca file 
            continue;
        }

        // 8. Eksekusi SQL (multi query)
        if ($conn->multi_query($sql)) {
            
            // Bersihkan result set untuk multi_query
            while ($conn->more_results()) {
                $conn->next_result();
            }
        } else {
            // Error handeler
            $conn->error;
        }
    }

    //9. membuka file data_prodi
    $data_prodi = json_decode(file_get_content("./data_prodi.json"), true);

    //10. Masukkan semua data JSON ke tabel_prodi
    foreach ($data_prodi as $row) {
        // Ambil data dengan perlindungan aman
        $kode_prodi = $conn->real_escape_string($row['kode_prodi']);
        $nama_prodi = $conn->real_escape_string($row['prodi']);
        $fakultas    = $conn->real_escape_string($row['fasultas']);

        $query = "
            INSERT INTO tabel_prodi (kode_prodi, nama_prodi, fakultas)
            VALUES ('$kode_prodi', '$nama_prodi', '$fakultas')
        ";

        if (!$conn->query($query)) {
            echo "Gagal memasukkan data prodi: " . $conn->error . "<br>";
        }
    }

    //11. Kembali ke index.php
    header("Location: index.php");
    exit;
?>
```